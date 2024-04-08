# Getting started with RH-SSO

Red Hat Single Sign-On is an integrated sign-on solution available as a Red Hat JBoss Middleware for OpenShift containerized image. The Red Hat Single Sign-On for OpenShift image provides an authentication server for users to centrally log in, log out, register, and manage user accounts for web applications, mobile applications, and RESTful web services.

Red Hat offers multiple OpenShift application templates utilizing the Red Hat Single Sign-On for OpenShift image version number 7.5.GA.

## What are OpenShift Templates?

A template describes a set of objects that can be parameterized and processed to produce a list of objects for creation by OpenShift Container Platform. A template can be processed to create anything you have permission to create within a project, for example services, build configurations, and deployment configurations. A template may also define a set of labels to apply to every object defined in the template.

You can create a list of objects from a template using the CLI or, if a template has been uploaded to your project or the global template library, using the web console. For a curated set of templates, see the OpenShift Image Streams and Templates library OpenShift Image Streams and Templates library.

We will be deploying Red Hat SSO from one of the templates. Also in one of the later sections we will create a custom image from the default imagestream provided by Red Hat SSO and apply it with our own Custom template.

Following are some more details about Red Hat SSO templates and their purpose. The reason for having multiple templates is to service different customer use cases. Red Hat SSO and Keycloak are used vastly and in different indesutries thereby it can fit into many use cases. Whether the customer uses OIDC or SAML i.e. using new and modern appliction architectures or connecting to legacy applications. Red Hat SSO has the capabilities to be the Single Sign On solution. More to come, stay tuned!

These define the resources needed to develop Red Hat Single Sign-On 7.5.GA server based deployment and can be split into the following two categories:

Templates using HTTPS and JGroups keystores and a truststore for the Red Hat Single Sign-On server, all prepared beforehand. These secure the TLS communication using passthrough TLS termination:

  - sso75-https: Red Hat Single Sign-On 7.5.GA backed by internal H2 database on the same pod.

  - sso75-postgresql: Red Hat Single Sign-On 7.5.GA backed by ephemeral PostgreSQL database on a separate pod.

  - sso75-postgresql-persistent: Red Hat Single Sign-On 7.5.GA backed by persistent PostgreSQL database on a separate pod.

Templates using OpenShift’s internal service serving x509 certificate secrets to automatically create the HTTPS keystore used for serving secure content. The JGroups cluster traffic is authenticated using the AUTH protocol and encrypted using the ASYM_ENCRYPT protocol. The Red Hat Single Sign-On server truststore is also created automatically, containing the /var/run/secrets/kubernetes.io/serviceaccount/service-ca.crt CA certificate file, which is used to sign the certificate for HTTPS keystore. Moreover, the truststore for the Red Hat Single Sign-On server is pre-populated with the all known, trusted CA certificate files found in the Java system path. These templates secure the TLS communication using re-encryption TLS termination:

  - sso75-x509-https: Red Hat Single Sign-On 7.5.GA with auto-generated HTTPS keystore and Red Hat Single Sign-On truststore, backed by internal H2 database. The ASYM_ENCRYPT JGroups protocol is used for encryption of cluster traffic.

  - sso75-x509-postgresql-persistent: Red Hat Single Sign-On 7.5.GA with auto-generated HTTPS keystore and Red Hat Single Sign-On truststore, backed by persistent PostgreSQL database. The ASYM_ENCRYPT JGroups protocol is used for encryption of cluster traffic.

Other templates that integrate with Red Hat Single Sign-On are also available:

  - eap64-sso-s2i: Red Hat Single Sign-On-enabled Red Hat JBoss Enterprise Application Platform 6.4.

  - eap71-sso-s2i: Red Hat Single Sign-On-enabled Red Hat JBoss Enterprise Application Platform 7.1.

  - datavirt63-secure-s2i: Red Hat Single Sign-On-enabled Red Hat JBoss Data Virtualization 6.3.

These templates contain environment variables specific to Red Hat Single Sign-On that enable automatic Red Hat Single Sign-On client registration when deployed. Further on in the Advance Config section we will go over some of those environment variables and details.

## Installing RH-SSO on OpenShift

Make sure that you have completed the setting up instructions before moving forward in this section.

Lets first take a look at the installed templates:

- Open up a terminal and run the following command, to navigate to our project.
- click on top left menu, select Terminal, select New Terminalsso

  ![](images/sso-9.png)

- in bash terminal, check and change your current project to your project (Please Change userX to your username!!!)

  ```ssh
  oc project userX-keycloak

  ```

  example output

  ![](images/sso-10.png)

- to list all the templates we are going to use in these labs - Run the following command:

  ```ssh
  oc get templates -n openshift -o name | grep -o 'sso75.\+'
  ```

  Which should result in the following output; these are the list of templates installed on this OpenShift cluster. We could also change the above expression to sso72 and would get the similar output for the SSO 7.2 release templates.

  ```ssh
  sso75-https
  sso75-ocp4-x509-https
  sso75-ocp4-x509-postgresql-persistent
  sso75-postgresql
  sso75-postgresql-persistent
  ```

- Lets try to install from a template But first we want to know the different parameters our template will use. Today we choose a persistent Postgresql backed SSO installation.

- Open the OpenShift console

  click this link to open OpenShift console : [OpenShift Console](https://console-openshift-console.apps.cluster-t9pz8.t9pz8.sandbox502.opentlc.com/)

  ![](images/sso-11.png)

  - input username and password (change username to your username!!! such as user1, password is openshift) 
  - click skip tour
    
    ![](images/sso-12.png)

  - select userX-keycloak project link (change userX to you username!!!)
    
    ![](images/sso-13.png)

- Click on the button Add in the right menu and then > 'Developer Catalog' > 'All services'
  
  ![](images/sso-14.png)

  ![](images/sso-15.png)

- search with "SSO"

  ![](images/sso-16.png)

- Select the following template "Red Hat Single Sign-On 7.5 on OpenJDK (Ephemeral)" by clicking on it.

- Click 'Initiate Template'. This will bring you to the Configuration section. 
  
  ![](images/sso-17.png)

- All the fields in this section denote the parameters that can be added to this template. Take a look at all the diferent parameters. For now we will only change the SSO Administrator username and password I have chosen 'admin/password' feel free to choose any, but make sure you remember them or note them, since we will be using them again. For the realm choose 'demojs' as shown in the image.

  - leave all default except
  - set RH-SSO Administrator Username: admin
  - set RH-SSO Administrator Password: password
  - set RH-SSO Realm: demojs
  - Click Create

  ![](images/sso-18.png)  

- Once compeleted, OpenShift will start the installation procedure and you can view the progress in the OpenShift console. Wait for the deployment to compelete. Once the deployment is finished. you should see the following on your OpenShift console.

  ![](images/sso-19.png)

- Now that SSO is installed lets login to our newly created SSO. Click on the link as denoted in the picture and it should take you to the Red Hat SSO main page.
  
  - open link with deployment icon (top right of sso deployment icon)

    ![](images/sso-20.png)

  - or open link from route info in deployment detail side panel
   
    ![](images/sso-21.png)
  
  - click adminstration console link
  
    ![](images/sso-22.png)

  - login with admin/password (from above configuration)

    ![](images/sso-23.png)

  - view RH-SSO Administration Console

    ![](images/sso-24.png)

## Let’s move on

You now have a running SSO server. Lets move on to do some basic configuration before we head on to using it with our apps.

- [Configuring RH-SSO](4-config.md)