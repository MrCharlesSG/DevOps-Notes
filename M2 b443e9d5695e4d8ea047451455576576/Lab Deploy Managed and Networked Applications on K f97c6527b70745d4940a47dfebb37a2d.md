# Lab: Deploy Managed and Networked Applications on Kubernetes

1. Log in to the OpenShift cluster.
    
    ```
    [student@workstation ~]$oc login -u developer -p developer \
      https://api.ocp4.example.com:6443
    Login successful
    ...output omitted...
    ```
    
2. Change to the `database-applications` project.
    
    ```
    [student@workstation ~]$oc project database-applications
    Now using project "database-applications" on server "https://api.ocp4.example.com:6443".
    ...output omitted...
    ```
    
3. Create the MySQL database deployment. Ignore the warning message.
    
    ```
    [student@workstation ~]$oc create deployment mysql-app \
      --image registry.ocp4.example.com:8443/redhattraining/mysql-app:v1
    deployment.apps/mysql-app created
    ```
    
4. Verify the deployment status. The pod name might differ in your output.
    
    ```
    [student@workstation ~]$oc get pods
    NAME              	          READY  STATUS  ...
    mysql-app-75dfd58f99-5xfqc   0/1    Error   ...
    [student@workstation ~]$oc status...output omitted...
    Errors:
      pod/mysql-app-75dfd58f99-5xfqc is crash-looping
    
    1 error, 1 info identified, use 'oc status --suggest' to see details.
    ```
    
5. Identify the root cause of the deployment failure.
    
    ```
    [student@workstation ~]$oc logs mysql-app-75dfd58f99-5xfqc...output omitted...
    You must either specify the following environment variables:
      MYSQL_USER
      MYSQL_PASSWORD
      MYSQL_DATABASE
    Or the following environment variable:
      MYSQL_ROOT_PASSWORD (regex: '^[a-zA-Z0-9_~!@#$%^&*()-=<>,.?;:|]+$')
    ...output omitted...
    ```
    
6. Update the environment variables for the `mysql-app` deployment.
    
    ```
    [student@workstation ~]$oc set env deployment/mysql-app \
      MYSQL_USER=redhat MYSQL_PASSWORD=redhat123 MYSQL_DATABASE=world_x
    deployment.apps/mysql-app updated
    ```
    
7. Verify that the `mysql-app` application pod is in the `RUNNING` state. The pod name might differ in your output.
    
    ```
    [student@workstation ~]$oc get pods
    NAME		                      READY   STATUS   ...
    mysql-app-57c44f646-5qt2k   1/1     Running  ...
    ```
    
8. Load the `world_x` database.
    
    ```
    [student@workstation ~]$oc exec -it mysql-app-57c44f646-5qt2k \
      -- /bin/bash -c "mysql -uredhat -predhat123 </tmp/world_x.sql"
    ...output omitted..
    [student@workstation ~]$
    ```
    
9. Confirm that you can access the MySQL database.
    
    ```
    [student@workstation ~]$oc rsh mysql-app-57c44f646-5qt2k
    ```
    
    ```
    sh-4.4$mysql -uredhat -predhat123 world_x...output omitted...
    mysql>
    ```
    
10. Exit the MySQL database, and then exit the container.
    
    ```
    mysql>exit
    Bye
    sh-4.4$exit
    ```
    
11. Expose the `mysql-app` deployment.
    
    ```
    [student@workstation ~]$oc expose deployment mysql-app --name mysql-service \
      --port 3306 --target-port 3306
    service/mysql-service created
    ```
    
12. Verify the service configuration. The endpoint IP address might differ in your output.
    
    ```
    [student@workstation ~]$oc get services
    NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    mysql-service   ClusterIP   172.30.146.213   <none>        3306/TCP   10s
    [student@workstation ~]$oc get endpoints
    NAME            ENDPOINTS         AGE
    mysql-service   10.8.0.102:3306   19s
    ```
    
13. Create the web application deployment. Ignore the warning message.
    
    ```
    [student@workstation ~]$oc create deployment php-app \
      --image registry.ocp4.example.com:8443/redhattraining/php-webapp:v1
    deployment.apps/php-app created
    ```
    
14. Verify the deployment status. Verify that the `php-app` application pod is in the `RUNNING` state.
    
    ```
    [student@workstation ~]$oc get pods
    NAME              READY  STATUS   ...
    php-app-725...    1/1    Running  ...
    mysql-app-57c...  1/1    Running  ...
    ```
    
    ```
    [student@workstation ~]$oc status...output omitted...
    deployment/php-app deploys registry.ocp4.example.com:8443/redhattraining/php-webapp:v1
      deployment #1 running for about a minute - 1 pod
    ...output omitted...
    ```
    
15. Expose the `php-app` deployment.
    
    ```
    [student@workstation ~]$oc expose deployment php-app --name php-svc \
      --port 8080 --target-port 8080
    service/php-svc exposed
    ```
    
16. Verify the service configuration. The endpoint IP address might differ in your output.
    
    ```
    [student@workstation ~]$oc get services
    NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    mysql-service   ClusterIP   172.30.146.213   <none>        3306/TCP   7m47s
    php-svc         ClusterIP   172.30.228.80    <none>        8080/TCP   4m34s
    [student@workstation ~]$oc get endpoints
    NAME            ENDPOINTS         AGE
    mysql-service   10.8.0.102:3306   7m50s
    php-svc         10.8.0.107:8080   4m37s
    ```
    
17. Expose the `php-svc` service.
    
    ```
    [student@workstation ~]$oc expose service/php-svc --name phpapp
    route.route.openshift.io/phpapp exposed
    ```
    
    ```
    [student@workstation ~]$oc get routes
    NAME    HOST/PORT                             ...
    phpapp  phpapp-database-applications.apps.ocp4.example.com  ...
    ```