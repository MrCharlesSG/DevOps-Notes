# Troubleshoot and Scale Applications

# Tasks

The API URL of your OpenShift cluster is https://api.ocp4.example.com:6443, and the `oc` command is already installed on your `workstation` machine.

The URL of the OpenShift web console is https://console-openshift-console.apps.ocp4.example.com. When you access the web console, select **Red Hat Identity Management** as the authentication mechanism.

Log in to the OpenShift cluster as the `developer` user with the `developer` password. The password for the `admin` user is `redhatocp`.

**`lab start compreview-scale`**

- A pod in the cluster is consuming excessive CPU and is interfering with other tasks. Identify the pod and remove its workload.
- The `compreview-scale` project already includes a web application at http://frontend-compreview-scale.apps.ocp4.example.com. When you access this URL, the application returns a list of quotations from famous authors. The application is broken for now, and is missing some configuration to be ready for production.
    
    The application uses two Kubernetes `Deployment` objects. The `frontend` deployment provides the application web pages, and relies on the `quotesdb` deployment that runs a MySQL database. The `lab` command already created the services and routes that connect the application components and that make the application available from outside the cluster.
    
    Fix the application and make it ready for production:
    
    - The `quotesdb` deployment in the `compreview-scale` project starts a MySQL server, but the database is failing. Review the logs of the pod to identify and then fix the issue.
        
        Use the following parameters for the database:
        
        | Name | Value |
        | --- | --- |
        | Username | operator1 |
        | Password | redhat123 |
        | Database name | quotes |
    - You security team validated a new version of the MySQL container image that fixes a security issue. The new container image is `registry.ocp4.example.com:8443/rhel9/mysql-80:1-237`.
        
        Update the `quotesdb` deployment to use this image. Ensure that the database redeploys.
        
        The classroom setup copied the image from the Red Hat Ecosystem Catalog. The original image is `registry.redhat.io/rhel9/mysql-80:1-237`.
        
    - Add a probe to the `quotesdb` deployment so that OpenShift can detect when the database is ready to accept requests. Use the `mysqladmin ping` command for the probe.
    - Add a second probe that regularly verifies the status of the database. Use the `mysqladmin ping` command as well.
    - Configure CPU and memory usage for the `quotesdb` deployment. The deployment needs 200 millicores of CPU and 256 MiB of memory to run, and you must restrict its CPU usage to 500 millicores and its memory usage to 1 GiB.
    - Add a probe to the `frontend` deployment so that OpenShift can detect when the web application is ready to accept requests. The application is ready when an HTTP request on port 8000 to the `/status` path is successful.
    - Add a second probe that regularly verifies the status of the web front end. The front end works as expected when an HTTP request on port 8000 to the `/env` path is successful.
    - Configure CPU and memory usage for the `frontend` deployment. The deployment needs 200 millicores of CPU and 256 MiB of memory to run, and you must restrict its CPU usage to 500 millicores and its memory usage to 512 MiB.
    - Scale the `frontend` application to three pods to accommodate for the estimated production load.
    - To verify your work, access the http://frontend-compreview-scale.apps.ocp4.example.com URL. The application returns a list of quotations from famous authors.

# Tasks Solved

**`lab start compreview-scale`**

- A pod in the cluster is consuming excessive CPU and is interfering with other tasks. Identify the pod and remove its workload.
    1. Use a web browser to access the https://console-openshift-console.apps.ocp4.example.com URL.
    2. Select **Red Hat Identity Management**, and then log in as the `admin` user with the `redhatocp` password. Click **Skip tour** if the **Welcome to the Developer Perspective** message is displayed.
    3. Switch to the **Administrator** perspective and then navigate to **Observe** → **Dashboards**.
        
        ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/compreview/scale/assets/dashboards.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/compreview/scale/assets/dashboards.png)
        
    4. Select the **Kubernetes / Compute Resources / Cluster** dashboard, and then click **Inspect** in the **CPU Usage** graph.
        
        ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/compreview/scale/assets/cpu.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/compreview/scale/assets/cpu.png)
        
    5. Set the zoom to five minutes and then hover over the graph. Notice that the interface lists the `compreview-scale-load` namespace in the first position, which indicated that this namespace is the first CPU consumer.
        
        ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/compreview/scale/assets/consumer.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/compreview/scale/assets/consumer.png)
        
    6. Navigate to **Observe** → **Dashboards** and then select the **Kubernetes / Compute Resources / Namespace (Workloads)** dashboard. Select the `compreview-scale-load` namespace and then set the time range to the last five minutes. The `computeprime` deployment is the workload that consumes excessive CPU.
        
        ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/compreview/scale/assets/workload.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/compreview/scale/assets/workload.png)
        
    7. Navigate to **Workloads** → **Deployments** and then select the `compreview-scale-load` project. Select the menu for the `computeprime` deployment and then click **Delete Deployment**. Click **Delete** to confirm the operation.
        
        ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/compreview/scale/assets/delete.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/compreview/scale/assets/delete.png)
        
- The `compreview-scale` project already includes a web application at http://frontend-compreview-scale.apps.ocp4.example.com. When you access this URL, the application returns a list of quotations from famous authors. The application is broken for now, and is missing some configuration to be ready for production.
    
    The application uses two Kubernetes `Deployment` objects. The `frontend` deployment provides the application web pages, and relies on the `quotesdb` deployment that runs a MySQL database. The `lab` command already created the services and routes that connect the application components and that make the application available from outside the cluster.
    
    Fix the application and make it ready for production:
    
    - The `quotesdb` deployment in the `compreview-scale` project starts a MySQL server, but the database is failing. Review the logs of the pod to identify and then fix the issue.
        
        Use the following parameters for the database:
        
        | Name | Value |
        | --- | --- |
        | Username | operator1 |
        | Password | redhat123 |
        | Database name | quotes |
        1. Log in to the OpenShift cluster from the command line.
            
            ```
            [student@workstation ~]$oc login -u developer -p developer https://api.ocp4.example.com:6443
            Login successful.
            ...output omitted...
            ```
            
        2. Set the `compreview-scale` project as the active project.
            
            ```
            [student@workstation ~]$oc project compreview-scale...output omitted...
            ```
            
        3. List the pods to identify the failing pod from the `quotesdb` deployment. The names of the pods on your system probably differ.
            
            ```
            [student@workstation ~]$oc get pods
            NAME                        READY   STATUS             RESTARTS         AGE
            frontend-5fb85b4c75-5s7xr   0/1     CrashLoopBackOff   14 (2m52s ago)   50m
            quotesdb-9b9776479-4z4g9    0/1     CrashLoopBackOff   14 (3m4s ago)    50m
            ```
            
        4. Retrieve the logs for the failing pod. Some environment variables are missing.
            
            ```
            [student@workstation ~]$oc logs quotesdb-9b9776479-4z4g9
            => sourcing 20-validate-variables.sh ...
            You must either specify the following environment variables:
            MYSQL_USER (regex: '^[a-zA-Z0-9_]+$')
            MYSQL_PASSWORD (regex: '[a-zA-Z0-9_~!@#$%&*()-=<>,.?;:|]+$')
            MYSQL_DATABASE (regex: '^[a-zA-Z0-9_]+$')
            ...output omitted...
            ```
            
        5. Add the missing environment variables to the `quotesdb` deployment.
            
            ```
            [student@workstation ~]$oc set env deployment/quotesdb \MYSQL_USER=operator1  MYSQL_PASSWORD=redhat123 MYSQL_DATABASE=quotes
            deployment.apps/quotesdb updated
            ```
            
    - You security team validated a new version of the MySQL container image that fixes a security issue. The new container image is `registry.ocp4.example.com:8443/rhel9/mysql-80:1-237`.
        
        Update the `quotesdb` deployment to use this image. Ensure that the database redeploys.
        
        The classroom setup copied the image from the Red Hat Ecosystem Catalog. The original image is `registry.redhat.io/rhel9/mysql-80:1-237`.
        
        1. Retrieve the name of the container that is running inside the pod. You need the container name to update its image.
            
            ```
            [student@workstation ~]$oc get deployment/quotesdb -o wide
            NAME       READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS ...
            quotesdb   1/1     1            1           59m  mysql-80   ...
            ```
            
        2. Set the image to `registry.ocp4.example.com:8443/rhel9/mysql-80:1-237`.
            
            ```
            [student@workstation ~]$oc set image deployment/quotesdb \mysql-80=registry.ocp4.example.com:8443/rhel9/mysql-80:1-237
            deployment.apps/quotesdb image updated
            ```
            
        3. Verify your work.
            
            ```
            [student@workstation ~]$oc get deployment/quotesdb -o wide
            NAME      ... CONTAINERS   IMAGES
            quotesdb  ... mysql-80registry.ocp4.example.com:8443/rhel9/mysql-80:1-237
            ```
            
        4. Wait for the deployment to roll out. You might have to rerun the command several times for the pod to report a `Running` status. The name of the pod on your system probably differs.
            
            ```
            [student@workstation ~]$oc get pods
            NAME                        READY   STATUS             RESTARTS         AGE
            frontend-5fb85b4c75-5s7xr   0/1     CrashLoopBackOff   15 (3m39s ago)   56m
            quotesdb-54d64749c4-chhq6   1/1Running            0                106s
            ```
            
    - Add a probe to the `quotesdb` deployment so that OpenShift can detect when the database is ready to accept requests. Use the `mysqladmin ping` command for the probe.
        1. Use the `oc set probe` command with the `-readiness` option to add the readiness probe.
            
            *A **readiness probe** determines whether the application is ready to serve requests.* 
            
            ```
            [student@workstation ~]$oc set probe deployment/quotesdb \--readiness -- mysqladmin ping
            deployment.apps/quotesdb probes updated
            ```
            
    - Add a second probe that regularly verifies the status of the database. Use the `mysqladmin ping` command as well.
        1. Use the `oc set probe` command with the `-liveness` option to add the liveness probe.
            
            *Like a readiness probe, a **liveness probe** is called throughout the lifetime of the application. Liveness probes determine whether the application container is in a healthy state. If an application fails its liveness probe enough times, then the cluster restarts the pod according to its restart policy.*
            
            ```
            [student@workstation ~]$oc set probe deployment/quotesdb \--liveness -- mysqladmin ping
            deployment.apps/quotesdb probes updated
            ```
            
    - Configure CPU and memory usage for the `quotesdb` deployment. The deployment needs 200 millicores of CPU and 256 MiB of memory to run, and you must restrict its CPU usage to 500 millicores and its memory usage to 1 GiB.
        
        ```
        [student@workstation ~]$oc set resources deployment/quotesdb \--requests cpu=200m,memory=256Mi --limits cpu=500m,memory=1Gi
        deployment.apps/quotesdb resource requirements updated
        ```
        
    - Add a probe to the `frontend` deployment so that OpenShift can detect when the web application is ready to accept requests. The application is ready when an HTTP request on port 8000 to the `/status` path is successful.
        - Use the `oc set probe` command with the `--readiness` option to add the readiness probe that tests the `/status` path on HTTP port 8000.
            
            ```
            [student@workstation ~]$oc set probe deployment/frontend --readiness \--get-url http://:8000/status
            deployment.apps/frontend probes updated
            ```
            
    - Add a second probe that regularly verifies the status of the web front end. The front end works as expected when an HTTP request on port 8000 to the `/env` path is successful.
        - Use the `oc set probe` command with the `--liveness` option to add the liveness probe that tests the `/env` path on HTTP port 8000.
            
            ```
            [student@workstation ~]$oc set probe deployment/frontend --liveness \--get-url http://:8000/env
            deployment.apps/frontend probes updated
            ```
            
    - Configure CPU and memory usage for the `frontend` deployment. The deployment needs 200 millicores of CPU and 256 MiB of memory to run, and you must restrict its CPU usage to 500 millicores and its memory usage to 512 MiB.
        
        ```
        [student@workstation ~]$oc set resources deployment/frontend \--requests cpu=200m,memory=256Mi --limits cpu=500m,memory=512Mi
        deployment.apps/frontend resource requirements updated
        ```
        
    - Scale the `frontend` application to three pods to accommodate for the estimated production load.
        1. Scale the deployment.
            
            ```
            [student@workstation ~]$oc scale deployment/frontend --replicas 3
            deployment.apps/frontend scaled
            ```
            
        2. Wait for the deployment to scale up. You might have to rerun the command several times for the pods to report a `Running` status. The names of the pods on your system probably differ.
            
            ```
            [student@workstation ~]$oc get pods
            NAME                        READY   STATUS    RESTARTS   AGE
            frontend-86cdd7c7bf-8vrrs   1/1Running   0          3m10s
            frontend-86cdd7c7bf-ds79w   1/1Running   0          44s
            frontend-86cdd7c7bf-hpnwz   1/1Running   0          44s
            quotesdb-66ff98b88c-fhwhs   1/1     Running   0          12m
            ```
            
    - To verify your work, access the http://frontend-compreview-scale.apps.ocp4.example.com URL. The application returns a list of quotations from famous authors.
        1. Retrieve the URL of the application.
            
            ```
            [student@workstation ~]$oc get route
            NAME       HOST/PORT                                         PATH   SERVICES ...
            frontendfrontend-compreview-scale.apps.ocp4.example.com          frontend ...
            ```
            
        2. Use the `curl` command to test the application.
            
            ```
            [student@workstation ~]$curl http://frontend-compreview-scale.apps.ocp4.example.com
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