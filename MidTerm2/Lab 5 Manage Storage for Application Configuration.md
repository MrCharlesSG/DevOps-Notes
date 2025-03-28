# Lab 5: Manage Storage for Application Configuration and Data

1. Log in to the OpenShift cluster and change to the `storage-review` project.
    1. Log in to the OpenShift cluster.
        
        ```
        [student@workstation ~]$oc login -u developer -p developer \
        https://api.ocp4.example.com:6443...output omitted...
        ```
        
    2. Change to the `storage-review` project.
        
        ```
        [student@workstation ~]$oc project storage-review
        Now using project "storage-review" on server "https://api.ocp4.example.com:6443".
        ...output omitted...
        ```
        
2. Create a secret named `world-cred` that contains the following data:
    
    
    | Field | Value |
    | --- | --- |
    | user | redhat |
    | password | redhat123 |
    | database | world_x |
    1. Create a secret that contains the database credentials.
        
        ```
        [student@workstation]$oc create secret generic world-cred \
          --from-literal user=redhat \
          --from-literal password=redhat123 \
          --from-literal database=world_x
        secret/world-cred created
        ```
        
    2. Confirm the creation of the secret.
        
        ```
        [student@workstation ~]$oc get secrets world-cred
        NAME         TYPE     DATA   AGE
        world-cred   Opaque   3      2m34s
        ```
        
3. Create a configuration map named `dbfiles` by using the `~/DO180/labs/storage-review/insertdata.sql` file.
    1. Create a configuration map named `dbfiles` by using the `insertdata.sql` file in the ﻿`~/DO180/labs/storage-review` directory.
        
        ```
        [student@workstation ~]$oc create configmap dbfiles \
          --from-file ~/DO180/labs/storage-review/insertdata.sql
        configmap/dbfiles created
        ```
        
    2. Verify the creation of the configuration map.
        
        ```
        [student@workstation]$oc get configmaps
        NAME	 DATA  AGE
        dbfiles  1     11s
        ...output omitted...
        ```
        
4. Create a database server deployment named `dbserver` by using the `registry.ocp4.example.com:8443/redhattraining/mysql-app:v1` container image. Then, set the missing environment variables by using the `world-cred` secret.
    1. Create the database server deployment.
        
        ```
        [student@workstation ~]$oc create deployment dbserver \
          --image registry.ocp4.example.com:8443/redhattraining/mysql-app:v1
        deployment.apps/dbserver created
        ```
        
    2. Set the missing environment variables.
        
        ```
        [student@workstation ~]$oc set env deployment/dbserver --from secret/world-cred \
          --prefix MYSQL_
        deployment.apps/dbserver updated
        ```
        
    3. Verify that the `dbserver` pod is in the `RUNNING` state. The pod name might differ in your output.
        
        ```
        [student@workstation ~]$oc get pods
        NAME                        READY   STATUS  ...
        dbserver-6d5bf5d86c-ptrb2   1/1     Running ...
        ```
        
5. Add a volume to the `dbserver` deployment by using the following information:
    
    
    | Field | Value |
    | --- | --- |
    | name | dbserver-lvm |
    | type | persistentVolumeClaim |
    | claim mode | rwo |
    | claim size | 1Gi |
    | mount path | /var/lib/mysql |
    | claim class | lvms-vg1 |
    | claim name | dbserver-lvm-pvc |
    1. Add a volume to the `dbserver` deployment.
        
        ```
        [student@workstation ~]$oc set volume deployment/dbserver \
          --add --name dbserver-lvm --type persistentVolumeClaim \
          --claim-mode rwo --claim-size 1Gi --mount-path /var/lib/mysql \
          --claim-class lvms-vg1 --claim-name dbserver-lvm-pvc
        deployment.apps/dbserver volume updated
        ```
        
    2. Verify the deployment status.
        
        ```
        [student@workstation ~]$oc get pods
        NAME              	    READY   STATUS   ...
        dbserver-5bc6bd5d7b-7z7lv   1/1     Running  ...
        ```
        
    3. Verify the volume status.
        
        ```
        [student@workstation ~]$oc get pvc
        NAMESTATUS   VOLUME             CAPACITY...output omitted...
        dbserver-lvm-pvcBound    pvc-2cb85025-...   1Gi      ...output omitted...
        ```
        
6. Create a service for the `dbserver` deployment by using the following information:
    
    
    | Field | Value |
    | --- | --- |
    | Name | mysql-service |
    | Port | 3306 |
    | Target port | 3306 |
    1. Expose the `dbserver` deployment.
        
        ```
        [student@workstation ~]$oc expose deployment dbserver --name mysql-service \
          --port 3306 --target-port 3306
        service/mysql-service exposed
        ```
        
    2. Verify the service configuration. The endpoint IP address might differ in your output.
        
        ```
        [student@workstation ~]$oc get services
        NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   ...
        mysql-service   ClusterIP   172.30.240.100  <none>        3306/TCP  ...
        ```
        
        ```
        [student@workstation ~]$oc get endpoints
        NAME            ENDPOINTS      ...
        mysql-service   10.8.1.36:3306 ...
        ```
        
7. Create a web application deployment named `file-sharing` by using the `registry.ocp4.example.com:8443/redhattraining/php-webapp-mysql:v1` container image. Scale the deployment to two replicas. Then, expose the deployment by using the following information:
    
    
    | Field | Value |
    | --- | --- |
    | Name | file-sharing |
    | Port | 8080 |
    | Target port | 8080 |
    
    Create a route named `file-sharing` to expose the `file-sharing` web application to external access. Access the `file-sharing` route in a web browser to test the connection between the web application and the database server.
    
    1. Create a web application deployment.
        
        ```
        [student@workstation ~]$oc create deployment file-sharing \
          --image registry.ocp4.example.com:8443/redhattraining/php-webapp-mysql:v1
        deployment.apps/file-sharing created
        ```
        
    2. Verify the deployment status. Verify that the `file-sharing` application pod is in the `RUNNING` state. The pod names might differ on your system.
        
        ```
        [student@workstation ~]$oc get pods
        NAME                            READY   STATUS   ...
        dbserver-5bc6bd5d7b-7z7lv       1/1     Running  ...
        file-sharing-789c5948c8-gdrlz   1/1     Running  ...
        ```
        
    3. Scale the deployment to two replicas.
        
        ```
        [student@workstation ~]$oc scale deployment file-sharing --replicas 2
        deployment.apps/file-sharing scaled
        ```
        
    4. Verify the replica status and retrieve the pod name. The pod names might differ on your system.
        
        ```
        [student@workstation ~]$oc get pods
        NAME                            READY   STATUS    ...
        dbserver-5bc6bd5d7b-7z7lv       1/1     Running   ...
        file-sharing-789c5948c8-62j9s   1/1     Running   ...
        file-sharing-789c5948c8-gdrlz   1/1     Running   ...
        ```
        
    5. Expose the `file-sharing` deployment.
        
        ```
        [student@workstation ~]$oc expose deployment file-sharing --name file-sharing \
          --port 8080 --target-port 8080
        service/file-sharing exposed
        ```
        
    6. Verify the service configuration. The endpoint IP address might differ in your output.
        
        ```
        [student@workstation ~]$oc get services
        NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   ...
        file-sharing    ClusterIP   172.30.139.210  <none>        8080/TCP  ...
        mysql-service   ClusterIP   172.30.240.100  <none>        3306/TCP  ...
        ```
        
        ```
        [student@workstation ~]$oc get endpoints
        NAME           ENDPOINTS
        file-sharing   10.8.1.37:8080,10.8.1.38:8080   ...
        mysql-service  10.8.1.36:3306                  ...
        ```
        
    7. Expose the `file-sharing` service.
        
        ```
        [student@workstation ~]$oc expose service/file-sharing
        route.route.openshift.io/file-sharing exposed
        ```
        
        ```
        [student@workstation ~]$oc get routes
        NAME           HOST/PORT                                           ...   SERVICES     ...
        file-sharing   file-sharing-storage-review.apps.ocp4.example.com   ...   file-sharing ...
        ```
        
    8. Test the connectivity between the web application and the database server. In a web browser, navigate to `http://file-sharing-storage-review.apps.ocp4.example.com`, and verify that a `Connected successfully` message is displayed.
8. Mount the `dbfiles` configuration map to the `file-sharing` deployment as a volume named `config-map-pvc`. Set the mount path to the `/home/database-files` directory. Then, verify the content of the `insertdata.sql` file.
    1. Mount the `dbfiles` configuration map to the `file-sharing` deployment.
        
        ```
        [student@workstation ~]$oc set volume deployment/file-sharing \
          --add --name config-map-pvc --type configmap \
          --configmap-name dbfiles \
          --mount-path /home/database-files
        deployment.apps/file-sharing volume updated
        ```
        
    2. Verify the deployment status.
        
        ```
        [student@workstation ~]$oc get pods
        NAME              	        READY   STATUS   ...
        dbserver-5bc6bd5d7b-7z7lv       1/1     Running  ...
        file-sharing-7f77855b7f-949lg   1/1     Running  ...
        file-sharing-7f77855b7f-9zvwq   1/1     Running  ...
        ```
        
    3. Verify the content of the `/home/database-files/insertdata.sql` file.
        
        ```
        [student@workstation ~]$oc exec -it pod/file-sharing-7f77855b7f-949lg -- \
          head /home/database-files/insertdata.sql
        -- MySQL dump 10.13  Distrib 8.0.19, for osx10.14 (x86_64)
        --
        -- Host: 127.0.0.1    Database: world_x
        -- ------------------------------------------------------
        -- Server version	8.0.19-debug
        ...output omitted...
        ```
        

1. Add a shared volume to the `file-sharing` deployment. Use the following information to create the volume:
    
    
    | Field | Value |
    | --- | --- |
    | Name | shared-volume |
    | Type | persistentVolumeClaim |
    | Claim mode | rwo |
    | Claim size | 1Gi |
    | Mount path | /home/sharedfiles |
    | Claim class | nfs-storage |
    | Claim name | shared-pvc |
    
    Next, connect to a `file-sharing` deployment pod and then use the `cp` command to copy the `/home/database-files/insertdata.sql` file to the `/home/sharedfiles` directory. Then, remove the `config-map-pvc` volume from the `file-sharing` deployment.
    
    1. Add the `shared-volume` volume to the `file-sharing` deployment.
        
        ```
        [student@workstation ~]$oc set volume deployment/file-sharing \
          --add --name shared-volume --type persistentVolumeClaim \
          --claim-mode rwo --claim-size 1Gi --mount-path /home/sharedfiles \
          --claim-class nfs-storage --claim-name shared-pvc
        deployment.apps/file-sharing volume updated
        ```
        
    2. Verify the deployment status. Your pod names might differ on your system.
        
        ```
        [student@workstation ~]$oc get pods
        NAME              	              READY   STATUS  ...
        dbserver-5bc6bd5d7b-7z7lv       1/1     Running ...
        file-sharing-65884f75bb-92fxf   1/1     Running ...
        file-sharing-65884f75bb-gsghk   1/1     Running ...
        ```
        
    3. Verify the volume status.
        
        ```
        [student@workstation ~]$oc get pvc
        NAME              STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS
        dbserver-lvm-pvc  Bound    pvc-2cb...  1Gi        RWO            lvms-vg1    ...
        shared-pvc        Bound    pvc-cf2...  1Gi        RWO            nfs-storage ...
        ```
        
    4. Copy the `/home/database-files/insertdata.sql` file to the `/home/sharedfiles` path.
        
        ```
        [student@workstation ~]$oc exec -it pod/file-sharing-65884f75bb-92fxf -- \
          cp /home/database-files/insertdata.sql /home/sharedfiles/
        ```
        
        ```
        [student@workstation ~]$oc exec -it pod/file-sharing-65884f75bb-92fxf -- \
          ls /home/sharedfiles/
        insertdata.sql
        ```
        
    5. Remove the `config-map-pvc` volume from the `file-sharing` deployment.
        
        ```
        [student@workstation ~]$oc set volume deployment/file-sharing \
          --remove --name=config-map-pvc
        deployment.apps/file-sharing volume updated
        ```
        
2. Add the `shared-volume` PVC to the `dbserver` deployment. Then, connect to a `dbserver` deployment pod and verify the content of the `/home/sharedfiles/insertdata.sql` file.
    1. Add the `shared-volume` volume to the `dbserver` deployment.
        
        ```
        [student@workstation ~]$oc set volume deployment/dbserver \
          --add --name shared-volume \
          --claim-name shared-pvc \
          --mount-path /home/sharedfiles
        deployment.apps/dbserver volume updated
        ```
        
    2. Verify the deployment status. The pod names might differ on your system.
        
        ```
        [student@workstation ~]$oc get pods
        NAME              	              READY   STATUS   ...
        dbserver-6676fbf5fc-n9hpk       1/1     Running  ...
        file-sharing-5fdb44cf57-2hhwj   1/1     Running  ...
        file-sharing-5fdb44cf57-z4n7g   1/1     Running  ...
        ```
        
    3. Verify the content of the `/home/sharedfiles/insertdata.sql` file.
        
        ```
        [student@workstation ~]$oc exec -it pod/dbserver-6676fbf5fc-n9hpk -- \
          head /home/sharedfiles/insertdata.sql
        -- MySQL dump 10.13  Distrib 8.0.19, for osx10.14 (x86_64)
        --
        -- Host: 127.0.0.1    Database: world_x
        -- ------------------------------------------------------
        -- Server version	8.0.19-debug
        ...output omitted...
        ```
        
3. Connect to the database server and execute the `/home/sharedfiles/insertdata.sql` file to add data to the `world_x` database. You can execute the file by using the following command:
    
    ```
    mysql -u$MYSQL_USER -p$MYSQL_PASSWORD world_x </home/sharedfiles/insertdata.sql
    ```
    
    Then, confirm connectivity between the web application and database server by accessing the `file-sharing` route in a web browser.
    
    1. Connect to the database server and execute the `/home/sharedfiles/insertdata.sql` file. Then, exit the database server.
        
        ```
        [student@workstation ~]$oc rsh dbserver-6676fbf5fc-n9hpk
        ```
        
        ```
        sh-4.4$mysql -u$MYSQL_USER -p$MYSQL_PASSWORD world_x </home/sharedfiles/insertdata.sql
        mysql: [Warning] Using a password on the command line interface can be insecure.
        sh-4.4$exit
        exit
        ```
        
    2. Test the connectivity between the web application and the database server. In a web browser, navigate to `http://file-sharing-storage-review.apps.ocp4.example.com`, and verify that the application retrieves data from the `world_x` database.
        
        ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/storage/review/assets/data-retrieved.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/storage/review/assets/data-retrieved.png)