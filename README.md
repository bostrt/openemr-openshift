Go to https://github.com/RedHatOfficial/openemr-kube for a more concerted effort on supporting OpenEMR in OpenShift/K8s. 

#### Scratch Notes

Below are the commands you can run. 6 things to note first though:

- I honestly have no idea how to prove this works beyond seeing the Pod created and going into a ready state. I've never interacted with OpenEMR before unfortunately.
- I have no idea what will happen if you scale up OpenEMR
- You'll have to add the anyuid SCC (or something allowing root UID) to the openemr Service Account. I'm not sure if there's a way to prevent this at this point since OpenEMR wants to run as root.
- The OpenEMR Pod logs will scream errors about connecting to MySQL on first start-up but once it is able to connect, that stops and it moves along. The OpenEMR Pod should become ready in like 5 minutes or so on a decent cluster (add more time to health probes if is gets killed).
- I'm sure there's all sorts of improvements on parameterizing things in this template and making it more resilient. I'm happy to submit this to the OpenEMR community if you find improvements.
- Tested this on both OCP 3.11 and 4.2 (didn't have access to a 4.3 but I don't see why it wouldn't work there too)

```
$ oc new-project my-openemr
$ oc adm policy add-scc-to-user anyuid -z openemr
$ oc create -f https://raw.githubusercontent.com/bostrt/openemr-openshift/master/openemr-template.yaml
$ oc new-app openemr-persistent --param MYSQL_ROOT_PASSWORD=root --param MYSQL_PASSWORD=openemr --param OE_PASS=pass
--> Deploying template "my-openemr/openemr-persistent" to project my-openemr

     openemr-persistent
     ---------
     Template for OpenEMR, medical practice management software.

     * With parameters:
        * MariaDB root Password=root
        * MariaDB Connection Username=openemr # generated
        * MariaDB Connection Password=openemr
        * MariaDB Database Name=openemr
        * OpenEMR admin username.=admin # generated
        * OpenEMR admin user password.=pass

--> Creating resources ...
    serviceaccount "openemr" created
    service "mariadb" created
    service "openemr" created
    deploymentconfig.apps.openshift.io "mariadb" created
    deploymentconfig.apps.openshift.io "openemr" created
    imagestream.image.openshift.io "mariadb" created
    imagestream.image.openshift.io "openemr" created
    route.route.openshift.io "openemr" created
    persistentvolumeclaim "mariadb" created
--> Success
    Access your application via route 'xxxxxxxxxx'
    Run 'oc status' to view your app.
```
