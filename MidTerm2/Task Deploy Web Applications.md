# Task: Deploy Web Applications

# Tasks

**`lab start compreview-deploy`**

The API URL of your OpenShift cluster is https://api.ocp4.example.com:6443, and the `oc` command is already installed on your `workstation` machine.

The URL of the OpenShift web console is https://console-openshift-console.apps.ocp4.example.com. When you access the web console, select **Red Hat Identity Management** as the authentication mechanism.

Log in to the OpenShift cluster as the `developer` user with the `developer` password. The password for the `admin` user is `redhatocp`, although you do not need administrator privileges to complete the exercise.

In this exercise, you deploy a web application and its database for testing purposes. The resulting configuration is not ready for production, because you do not configure probes and resource limits, which are required for production. Another comprehensive review exercise covers these subjects.

Perform the following tasks to complete the exercise:

- Create a project named `review` to store your work.
- Configure your project so that its workloads refer to the database image by the `mysql8:1` short name.
    - The short name must point to the `registry.ocp4.example.com:8443/rhel9/mysql-80:1-228` container image. The database image name and its source registry are expected to change in the near future, and you want to isolate your workloads from that change.
        
        The classroom setup copied the image from the Red Hat Ecosystem Catalog. The original image is `registry.redhat.io/rhel9/mysql-80:1-228`.
        
    - Ensure that the workload resources in the `review` project can use the `mysql8:1` resource. You create these workload resources in a later step.
- Create the `dbparams` secret to store the MySQL database parameters. Both the database and the front-end deployment need these parameters. The `dbparams` secret must include the following variables:
    
    
    | Name | Value |
    | --- | --- |
    | user | operator1 |
    | password | redhat123 |
    | database | quotesdb |
- Create the `quotesdb` deployment and configure it as follows:
    - Use the `mysql8:1` image for the deployment.
    - The database must automatically roll out whenever the source container in the `:1` resource changes.
        
        To test your configuration, you can change the `mysql8:1` image to point to the `registry.ocp4.example.com:8443/rhel9/mysql-80:1-237` container image that the classroom provides, and then verify that the `quotesdb` deployment rolls out. Remember to reset the `mysql8:1` image to the `registry.ocp4.example.com:8443/rhel9/mysql-80:1-228` container image before grading your work.
        
    - Define the following environment variables in the deployment from the keys in the `dbparams` secret:
        
        
        | Environment variable | dbparams secret key |
        | --- | --- |
        | MYSQL_USER | user |
        | MYSQL_PASSWORD | password |
        | MYSQL_DATABASE | database |
    - Ensure that OpenShift preserves the database data between pod restarts. This data does not consume more than 2 GiB of disk space. The MySQL database stores its data under the `/﻿var/lib/mysql` directory. Use the `lvms-lg1` storage class for the volume.
- Create a `quotesdb` service to make the database available to the front-end web application. The database service is listening on port 3306.
- Create the `frontend` deployment and configure it as follows:
    - Use the `registry.ocp4.example.com:8443/redhattraining/famous-quotes:2-42` image. For this deployment, you refer to the image by its full name, because your organization develops the image and controls its release process.
    - Define the following environment variables in the deployment:
        
        
        | Environment variable name | Value |
        | --- | --- |
        | QUOTES_USER | The user key from the dbparams secret |
        | QUOTES_PASSWORD | The password key from the dbparams secret |
        | QUOTES_DATABASE | The database key from the dbparams secret |
        | QUOTES_HOSTNAME | quotesdb |
- You cannot yet test the application from outside the cluster. Expose the `frontend` deployment so that the application can be reached at http://frontend-review.apps.ocp4.example.com.
    
    The `frontend` deployment is listening to port 8000.
    
    When you access the http://frontend-review.apps.ocp4.example.com URL, the application returns a list of quotations from famous authors.
    

# Tasks Solved

## Login

Log in as the `developer` user.

```
[student@workstation ~]$oc login -u developer -p developer \https://api.ocp4.example.com:6443
Login successful.
...output omitted...
```

## Create Project

Create a project named `review` to store your work.

```
[student@workstation ~]$oc new-project review
Now using project "review" on server "https://api.ocp4.example.com:6443".
...output omitted...
```

## Configure Database

Configure your project so that its workloads refer to the database image by the `mysql8:1` short name.

- The short name must point to the `registry.ocp4.example.com:8443/rhel9/mysql-80:1-228` container image. The database image name and its source registry are expected to change in the near future, and you want to isolate your workloads from that change.
    
    The classroom setup copied the image from the Red Hat Ecosystem Catalog. The original image is `registry.redhat.io/rhel9/mysql-80:1-228`.
    
    1. Use the `oc create istag` command to create the image stream and the image stream tag.
        
        ```
        [student@workstation ~]$oc create istag mysql8:1 \--from-image registry.ocp4.example.com:8443/rhel9/mysql-80:1-228
        imagestreamtag.image.openshift.io/mysql8:1 created
        ```
        
    2. Use the `oc set image-lookup` command to enable image lookup resolution.
        
        ```
        [student@workstation ~]$oc set image-lookup mysql8
        imagestream.image.openshift.io/mysql8 image lookup updated
        ```
        
- Ensure that the workload resources in the `review` project can use the `mysql8:1` resource. You create these workload resources in a later step.
    1. Run the `oc set image-lookup` command without any arguments to verify your work.
        
        ```
        [student@workstation ~]$oc set image-lookup
        NAME    LOCAL
        mysql8true
        ```
        

## Create the `dbparams` secret

Create the `dbparams` secret to store the MySQL database parameters. Both the database and the front-end deployment need these parameters. The `dbparams` secret must include the following variables:

| Name | Value |
| --- | --- |
| user | operator1 |
| password | redhat123 |
| database | quotesdb |

```
[student@workstation ~]$oc create secret generic dbparams --from-literal user=operator1 --from-literal password=redhat123 --from-literal database=quotesdb
secret/dbparams created
```

## Create the `quotesdb` deployment

Create the `quotesdb` deployment and configure it as follows:

- Use the `mysql8:1` image for the deployment.
    
    ```
    [student@workstation ~]$oc create deployment quotesdb --image mysql8:1 --replicas 0
    deployment.apps/quotesdb created
    ```
    
- The database must automatically roll out whenever the source container in the `mysql8:1` resource changes.
    
    To test your configuration, you can change the `mysql8:1` image to point to the `registry.ocp4.example.com:8443/rhel9/mysql-80:1-237` container image that the classroom provides, and then verify that the `quotesdb` deployment rolls out. Remember to reset the `mysql8:1` image to the `registry.ocp4.example.com:8443/rhel9/mysql-80:1-228` container image before grading your work.
    
    1. Retrieve the name of the container from the `quotesdb` deployment.
        
        ```
        [student@workstation ~]$oc get deployment quotesdb -o wide
        NAME       READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS ...
        quotesdb   0/0     0            0           11s    mysql8    ...
        ```
        
    2. Use the `oc set triggers` command to add the trigger for the `mysql8:1` image stream tag to the `mysql8` container.
        
        ```
        [student@workstation ~]$oc set triggers deployment/quotesdb --from-image mysql8:1 --containers mysql8
        deployment.apps/quotesdb triggers updated
        ```
        
- Define the following environment variables in the deployment from the keys in the `dbparams` secret:
    
    
    | Environment variable | dbparams secret key |
    | --- | --- |
    | MYSQL_USER | user |
    | MYSQL_PASSWORD | password |
    | MYSQL_DATABASE | database |
    
    ```
    [student@workstation ~]$oc set env deployment/quotesdb --from secret/dbparams --prefix MYSQL_
    deployment.apps/quotesdb updated
    ```
    
- Ensure that OpenShift preserves the database data between pod restarts. This data does not consume more than 2 GiB of disk space. The MySQL database stores its data under the `/﻿var/lib/mysql` directory. Use the `lvms-vg1` storage class for the volume
    
    ```
    [student@workstation ~]$oc set volumes deployment/quotesdb --add --claim-class lvms-vg1 --claim-size 2Gi --mount-path /var/lib/mysql
    info: Generated volume name: volume-n7xpd
    deployment.apps/quotesdb volume updated
    ```
    

## Expose the `quotesdb`

Create a `quotesdb` service to make the database available to the front-end web application. The database service is listening on port 3306.

1. Scale up the deployment.
    
    ```
    [student@workstation ~]$oc scale deployment/quotesdb --replicas 1
    deployment.apps/quotesdb scaled
    ```
    
2. Wait for the pod to start. You might have to rerun the command several times for the pod to report a `Running` status. The name of the pod on your system probably differs.
    
    ```
    [student@workstation ~]$oc get pods
    NAME                       READY   STATUS    RESTARTS   AGE
    quotesdb-99f9b4ff8-ggs7z   1/1     Running       0       4s
    ```
    
3. Use the `oc expose deployment` command to create the service.
    
    ```
    [student@workstation ~]$oc expose deployment quotesdb --port 3306
    service/quotesdb exposed
    ```
    
4. Verify that OpenShift associates the IP address of the MySQL server with the endpoint. The endpoint IP address on your system probably differs.
    
    ```
    [student@workstation ~]$oc describe service quotesdb
    Name:              quotesdb
    Namespace:         review
    ...output omitted...
    TargetPort:        3306/TCP
    Endpoints:         10.8.0.123:3306
    Session Affinity:  None
    Events:            <none>
    ```
    

## Create Frontend

Create the `frontend` deployment and configure it as follows:

- Use the  `registry.ocp4.example.com:8443/redhattraining/famous-quotes:2-42` image. For this deployment, you refer to the image by its full name, because your organization develops the image and controls its release process.
    
    ```
    [student@workstation ~]$oc create deployment frontend--image registry.ocp4.example.com:8443/redhattraining/famous-quotes:2-42 --replicas 0
    deployment.apps/frontend created
    ```
    
- Define the following environment variables in the deployment:
    
    
    | Environment variable name | Value |
    | --- | --- |
    | QUOTES_USER | The user key from the dbparams secret |
    | QUOTES_PASSWORD | The password key from the dbparams secret |
    | QUOTES_DATABASE | The database key from the dbparams secret |
    | QUOTES_HOSTNAME | quotesdb |
    1. Add the variables from the `dbparams` secret. Add the `QUOTES_` prefix to each variable name.
        
        ```
        [student@workstation ~]$oc set env deployment/frontend --from secret/dbparams --prefix QUOTES_
        deployment.apps/frontend updated
        ```
        
    2. Declare the `QUOTES_HOSTNAME` variable.
        
        ```
        [student@workstation ~]$oc set env deployment/frontend QUOTES_HOSTNAME=quotesdb
        deployment.apps/frontend updated
        ```
        

## Expose Frontend

- You cannot yet test the application from outside the cluster. Expose the `frontend` deployment so that the application can be reached at http://frontend-review.apps.ocp4.example.com.
    
    The `frontend` deployment is listening to port 8000.
    
    When you access the http://frontend-review.apps.ocp4.example.com URL, the application returns a list of quotations from famous authors.
    
    1. Scale up the deployment.
        
        ```
        [student@workstation ~]$oc scale deployment/frontend --replicas 1
        deployment.apps/frontend scaled
        ```
        
    2. Wait for the pod to start. You might have to rerun the command several times for the pod to report a `Running` status. The name of the pod on your system probably differs.
        
        ```
        [student@workstation ~]$oc get pods
        NAME                       READY   STATUS    RESTARTS   AGE
        frontend-86cdd7c7bf-hpnwz  1/1     Running   0          44s
        quotesdb-99f9b4ff8-ggs7z   1/1     Running   0          2m11s
        ```
        
    3. Create the `frontend` service for the `frontend` deployment.
        
        ```
        [student@workstation ~]$oc expose deployment frontend --port 8000
        service/frontend exposed
        ```
        
    4. Create the route.
        
        ```
        [student@workstation ~]$oc expose service frontend
        route.route.openshift.io/frontend exposed
        ```
        
    5. Retrieve the application URL from the route.
        
        ```
        [student@workstation ~]$oc get route
        NAME       HOST/PORT                               PATH   SERVICES ...
        frontendfrontend-review.apps.ocp4.example.com          frontend ...
        ```
        
    6. Use the `curl` command to test the application.
        
        ```
        [student@workstation ~]$curl http://frontend-review.apps.ocp4.example.com
        <html>
        	<head>
                <title>Quotes</title>
            </head>
            <body>
        
                <h1>Quote List</h1>
        
                    <ul>
        
                            <li>1: When words fail, music speaks.
        - William Shakespeare
        </li>
        ...output omitted...
        ```