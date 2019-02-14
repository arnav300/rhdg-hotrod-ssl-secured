Using SSL Secured Remote Access to Cache in Openshift via HotRod
==============================================
Example of how to setup a local Hot Rod client running against a RHDG cluster in an Openshift environment.


What is it?
-----------

Hot Rod is a binary TCP client-server protocol used in Red Hat Data Grid. The Hot Rod protocol facilitates faster client and server interactions in comparison to other text based protocols(Memcached, REST) and allows clients to make decisions about load balancing, failover and data location operations.

The `rhdg-hotrod-ssl-secured` quickstart demonstrates how to connect securely to remote Red Hat Data Grid (RHDG) to store, retrieve, and remove data from cache using the Hot Rod protocol. It is a simple Football Manager console application allows:
  coach can add or remove players from teams, or print a list of the all players
  player to see information about other players
  in the team using the Hot Rod based connector.

System requirements
-------------------

All you need to build this project is Java 8.0 (Java SDK 1.8) or higher, Maven 3.0 or higher.
The Java application is designed to be run locally
Red Hat Data Grid 7.x is running on top of Openshift 3.x
keytool is also needed to manage certificates, keystore and truststore

Preparating Openshift Environment
---------------

> Some parts of this sections can be skipped if already done

1. Login and create OCP project

~~~shell
$ oc login
$ oc new-project rhdg
~~~

2. Import datagrid7x-https template

~~~shell
$ oc create -n openshift -f https://raw.githubusercontent.com/jboss-container-images/jboss-datagrid-7-openshift-image/datagrid72/templates/datagrid72-https.json
~~~

3. Import jboss-datagrid7x-openshift image

~~~shell
$ oc import-image -n openshift jboss-datagrid72-openshift:1.3 --from=registry.access.redhat.com/jboss-datagrid-7/datagrid72-openshift:1.3 --confirm
~~~

4. Import HTTPS and JGroups keystores samples via Secret

~~~shell
$ oc create -n rhdg -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/secrets/datagrid-app-secret.json
~~~


Deploying Red Hat Data Grid 7.x on Openshift
---------------

1. Deploy Red Hat Data Grid

~~~shell
$ oc new-app datagrid72-https -n rhdg \
    -p CACHE_NAMES=teams \
    -p HOTROD_AUTHENTICATION=true \
    -p USERNAME=datagrid \
    -p PASSWORD=datagrid \
    -p HTTPS_NAME=jboss \
    -p HTTPS_PASSWORD=mykeystorepass \
    -p JGROUPS_ENCRYPT_NAME=secret-key \
    -p JGROUPS_ENCRYPT_PASSWORD=password
~~~

2. Create a SSL Route for Hod Rod Service

~~~shell
$ oc create -n rhdg route passthrough secure-datagrid-app-hotrod --service datagrid-app-hotrod
~~~

Hot Rod client configuration
----------------------------
  
1. Clone the current project

~~~shell
$ git clone https://github.com/mcouliba/rhdg-hotrod-ssl-secured.git
$ cd rhdg-hotrod-ssl-secured
~~~

2. Edit `src/main/ressources/jdg.properties` as following

~~~
jdg.host=secure-datagrid-app-hotrod-rhdg.<Openshift Application Suffix>
jdg.hotrod.port=443
~~~

> IMPORTANT: For the following step, you need to extract the trusted certificate provided by Openshift
> You can use a WebBrowser by clicking on the HTTPS route then export the certificate into `openshift.crt`

3. Import the trusted certificate into a new trustore for the Java application

~~~shell
$ keytool -import -noprompt -v -trustcacerts -keyalg RSA -alias openshift \
    -storepass mykeystorepass -file openshift.crt -keystore src/main/resources/truststore.jks
~~~

Build and Run the Quickstart
----------------------------

~~~shell
$ mvn clean package && mvn exec:java
~~~

Using the application
---------------------
Basic usage scenarios can look like this (keyboard shortcuts will be shown to you upon 

        at  -  add a team
        ap  -  add a player to a team
        rt  -  remove a team
        rp  -  remove a player from a team
        p   -  print all teams and players
        q   -  quit
        
Type `q` one more time to exit the application.
