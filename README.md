This repository contains a sample application that will be used to showcase deploying EAP on OpenShift, based the Red Hat JBoss EAP quickstarts.

Please refer to the root README.html for additional information.

As prerequisite, install a mySQL instance to simulate the original DB and support the EAP datasource


```
oc new-app -e MYSQL_DATABASE=eap -e MYSQL_PASSWORD=demo -e MYSQL_USER=eap mysql-persistent
```

Create a new project

```
oc new-project order-management-system
```

Apply the following yaml

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: eap-config
data: 
  DATASOURCES: "TEST"
  TEST_DATABASE: "eap"
  TEST_NAME: "mysql"
  TEST_DRIVER: "mysql"
  TEST_JNDI: "java:/jdbc/mysql"
  TEST_USERNAME: "eap"
  TEST_PASSWORD: "demo"
  TEST_URL: "jdbc:mysql://mysql:3306/eap"
  TEST_NONXA: "true"

```


### EAP Operator installation

As cluster admin, from the admin perspective

From the left menu, select Operators, Operator Hub

Using the filter form search EAP

Select the operator, select install and wait for the completion.

### EAP Deployment on OpenShift using the web console
As regular user, log and select the developer perspective

Select the order-management-system project 

On the topology view, go to add and search for 'helm eap', select then EAP74 and press Install Helm Chart

From the form view
- keep the image section as it is (default values)
- for the build section insert 
    - the uri with this repository root url and the cont
    - the contextDir with order-management-system
    - env
        - MAVEN_ARGS_APPEND -Dcom.redhat.xpaas.repo.jbossorg
        - CUSTOM_INSTALL_DIRECTORIES extensions
- for the deploy section
    - enabled false
    
The resulting helm chart will create 2 different build-configs:
- eap74-build-artifacts: The first to be run, that takes care of generating the application artifacts
- eap74: the last one which actually creates and push to the internal registry the image that will be run

From the developer perspective, select add, operator backed, wildfly server and then create

In the configuration, leave everything as it is but:
- image -> eap74:latest
- envfrom -> use the reference to the previously created eap-config (for the DB connectivity)

Wait for the container to be created and start and then connect to the application using the auto-generated route

### EAP Deployment on OpenShift using the CLI
Create the OpenShift project

oc new-project eap-webinar

Deploy mysql

oc new-app -e MYSQL_DATABASE=eap -e MYSQL_PASSWORD=demo -e MYSQL_USER=eap mysql-persistent

Install the helm cli following these instructions https://helm.sh/docs/intro/install/

Install the helm chart

helm install eap74 -f install-helm.yml http://github.com/openshift-helm-charts/charts/releases/download/redhat-eap74-1.1.0/redhat-eap74-1.1.0.tgz

Wait for the two builds to complete, eap_app_build_artifacts and eap_app. This will take a few minutes.

Create the config map with mysql environment variables;

oc apply -f eap-cm.yml

Deploy the JBoss EAP operator

oc apply -f eap-operator.yml

Deploy the instance of the eap application using the operator

oc apply -f eap-deploy.yml

Navigate to the route to test out the application.