# Chapter 4. Deploy Managed and Networked Applications on Kubernetes

# **Guided Exercise: Deploy Applications from Images and Templates**

1. As the `developer` user, create a project and verify that it is not empty after creation.
    1. Log in as the `developer` user with the `developer` password.
        
        ```bash
        [student@workstation ~]$oc login -u developer -p developer https://api.ocp4.example.com:6443
        ...output omitted...
        ```
        
    2. Create a project named `deploy-newapp`.
        
        ```
        [student@workstation ~]$oc new-project deploy-newapp
        Now using project "deploy-newapp" on server...output omitted...
        ```
        
        ```
        [student@workstation ~]$oc new-project deploy-newapp
        Now using project "deploy-newapp" on server...output omitted...
        ```
        
        The new project is automatically selected.
        
    3. Observe that resources for the new project are not returned with the `oc get all` command.
        
        ```
        [student@workstation ~]$oc get all
        No resources found in deploy-newapp namespace.
        ```
        
        <aside>
        <img src="https://www.notion.so/icons/light-bulb_blue.svg" alt="https://www.notion.so/icons/light-bulb_blue.svg" width="40px" /> Commands that use `all` for the resource type do not include every available resource type. Instead, `all` is a shorthand form for a predefined subset of types. When you use this command argument, ensure that `all` includes any types that you intend to address.
        
        </aside>
        
    4. Observe that the new project contains other types of resources.
        
        ```
        [student@workstation ~]$oc get serviceaccounts
        NAME       SECRETS   AGE
        builder    1         20s
        default    1         20s
        deployer   1         20s
        ```
        
        ```
        [student@workstation ~]$oc get secrets
        NAME                       TYPE                                  DATA   AGE
        builder-dockercfg-sczxg    kubernetes.io/dockercfg               1      36m
        builder-token-gsnqj        kubernetes.io/service-account-token   4      36m
        default-dockercfg-6f8nm    kubernetes.io/dockercfg               1      36m
        ...output omitted...
        ```
        
2. Create two MySQL instances by using the `oc new-app` command with different options.
    1. View the `mysql-persistent` template definition to see the resources that it creates. Specify the project that houses the template by using the `n openshift` option.
        
        ```
        [student@workstation ~]$oc describe template mysql-persistent -n openshift
        Name:		mysql-persistent
        Namespace:	openshift
        ...output omitted...
        Objects:
            Secret	                              ${DATABASE_SERVICE_NAME}
            Service	                             ${DATABASE_SERVICE_NAME}
            PersistentVolumeClaim	               ${DATABASE_SERVICE_NAME}
            DeploymentConfig.apps.openshift.io	  ${DATABASE_SERVICE_NAME}
        ```
        
        The `objects` attribute specifies several resource definitions that are applied on using the template. These resources include one of each of the following types: secret, service (`svc`), persistent volume claim (`pvc`), and deployment configuration (`dc`).
        
    2. Create an instance by using the `mysql-persistent` template. Specify a `name` option and attach a custom `team=red` label to the created resources.
        
        ```
        [student@workstation ~]$oc new-app -l team=red --template mysql-persistent \
          -p MYSQL_USER=developer \
          -p MYSQL_PASSWORD=developer...output omitted...
        --> Creating resources with label team=red ...
        secret "mysql" created
        service "mysql" created
        persistentvolumeclaim "mysql" created
        deploymentconfig.apps.openshift.io "mysql" created
        --> Success
        ...output omitted...
        ```
        
        The template creates resources of the types from the preceding step.
        
    3. View and wait for the pod to start, which takes a few minutes to complete. You might need to run the command several times before the status changes to `Running`.
        
        ```
        [student@workstation ~]$oc get pods
        NAME              READY   STATUS      RESTARTS  AGE
        mysql-1-deploy0/1     Completed   0         93s
        mysql-1-qmxbf     1/1     Running     1         60s
        ```
        
        Your pod names might differ from the previous output.
        
    4. Create an instance by using a container image. Specify a `name` option and attach a custom `team=blue` label to the created resources.
        
        ```
        [student@workstation ~]$oc new-app --name db-image -l team=blue \
          --image registry.ocp4.example.com:8443/rhel9/mysql-80:1 \
          -e MYSQL_USER=developer \
          -e MYSQL_PASSWORD=developer \
          -e MYSQL_ROOT_PASSWORD=redhat...output omitted...
        --> Creating resources with label team=blue ...
            imagestream.image.openshift.io "db-image" created
            deployment.apps "db-image" created
            service "db-image" created
        --> Success
        ...output omitted...
        ```
        
        The command creates predefined resources that are needed to deploy an image. These resource types are image stream (`is`), deployment, and service (`svc`). Image streams and services are discussed in more detail elsewhere in the course.
        
        <aside>
        <img src="https://www.notion.so/icons/light-bulb_blue.svg" alt="https://www.notion.so/icons/light-bulb_blue.svg" width="40px" /> It is safe to ignore pod security warnings for exercises in this course. OpenShift uses the Security Context Constraints controller to provide safe defaults for pod security.
        
        </aside>
        
    5. Wait for the pod to start. After a few moments, list all pods that contain `team` as a label.
        
        ```
        [student@workstation ~]$oc get pods -L team
        NAME                       READY   STATUS      RESTARTS   AGE    TEAM
        db-image-8d4b97594-6jb85   1/1     Running     0          55s    blue
        mysql-1-deploy             0/1     Completed   0          2m
        mysql-1-hn64v              1/1     Running     0          1m30s
        ```
        
        Your pod name might differ from the previous output. Without a `readinessProbe`, this pod shows as ready before the MySQL service is ready for requests. Readiness probes are discussed in more detail elsewhere in the course.
        
        Notice that only the `db-image` pod has a label that contains the word `team`. Pods that the `mysql-persistent` template creates do not have the `team=red` label, because the template does not define this label in its pod specification template.
        
3. Compare the resources that each image and template method creates.
    1. View the `template-created` pod and observe that it contains a readiness probe.
        
        ```
        [student@workstation ~]$oc get pods -l deploymentconfig=mysql \
          -o jsonpath='{.items[0].spec.containers[0].readinessProbe}' | jq
        {
          "exec": {
            "command": [
              "/bin/sh",
              "-i",
              "-c",
              "MYSQL_PWD=\"$MYSQL_PASSWORD\" mysqladmin -u $MYSQL_USER ping"
            ]
          },
          "failureThreshold": 3,
          "initialDelaySeconds": 5,
          "periodSeconds": 10,
          "successThreshold": 1,
          "timeoutSeconds": 1
        }
        ```
        
        <aside>
        <img src="https://www.notion.so/icons/light-bulb_blue.svg" alt="https://www.notion.so/icons/light-bulb_blue.svg" width="40px" /> The results of the preceding `oc` command are passed to the `jq` command, which formats the JSON output.
        
        </aside>
        
    2. Observe that the image-based pod does not contain a readiness probe.
        
        ```
        [student@workstation ~]$oc get pods -l deployment=db-image \
          -o jsonpath='{.items[0].spec.containers[0].readinessProbe}' | jqno output expected
        ```
        
    3. Observe that the template-based pod has a memory resource limit, which restricts allocated memory to the resulting pods. Resource limits are discussed in more detail elsewhere in the course.
        
        ```
        [student@workstation ~]$oc get pods -l deploymentconfig=mysql \
          -o jsonpath='{.items[0].spec.containers[0].resources.limits}' | jq
        {
          "memory": "512Mi"
        }
        ```
        
    4. Observe that the image-based pod has no resource limits.
        
        ```
        [student@workstation ~]$oc get pods -l deployment=db-image \
          -o jsonpath='{.items[0].spec.containers[0].resources}' | jq
        {}
        ```
        
    5. Retrieve secrets in the project. Notice that the template produced a secret, whereas the pod that was created with only an image did not.
        
        ```
        [student@workstation ~]$oc get secrets
        NAME                       TYPE                                  DATA   AGE
        ...output omitted...
        mysql                      Opaque                                4      3m
        ```
        
4. Explore filtering resources via labels.
    1. Observe that not supplying a label shows all services.
        
        ```
        [student@workstation ~]$oc get services
        NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
        db-image   ClusterIP   172.30.38.113   <none>        3306/TCP   1m30s
        mysql      ClusterIP   172.30.95.52    <none>        3306/TCP   2m30s
        ```
        
    2. Observe that supplying a label shows only the services with the label.
        
        ```
        [student@workstation ~]$oc get services -l team=red
        NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
        mysql   ClusterIP   172.30.95.52   <none>        3306/TCP   2m43s
        ```
        
    3. Observe that not all resources include the label, such as the pods that are created with the template.
        
        ```
        [student@workstation ~]$oc get pods -l team=red
        No resources found in deploy-newapp namespace.
        ```
        
5. Use labels to delete only the resources that are associated with the image-based deployment.
    1. Delete only the resources that use the `team=red` label by using it with the `oc delete` command. List the resource types from the template to ensure that all relevant resources are deleted.
        
        ```
        [student@workstation ~]$oc delete all -l team=red
        replicationcontroller "mysql-1" deleted
        service "mysql" deleted
        ...output omitted...
        deploymentconfig.apps.openshift.io "mysql" deleted
        ```
        
        ```
        [studen@workstation ~]$oc delete secret,pvc -l team=red
        secret "mysql" deleted
        persistentvolumeclaim "mysql" deleted
        ```
        
        <aside>
        <img src="https://www.notion.so/icons/light-bulb_blue.svg" alt="https://www.notion.so/icons/light-bulb_blue.svg" width="40px" /> By using the `oc delete all -l team=red` command, some resources are deleted, but the persistent volume claim and secret remain.
        
        </aside>
        
    2. Observe that the resources that the template created are deleted.
        
        ```
        [student@workstation ~]$oc get secret,svc,pvc,dc -l team=red...output omitted...
        No resources found in deploy-newapp namespace.
        ```
        
    3. Observe that the image-based resources remain unchanged.
        
        ```
        [student@workstation ~]$oc get is,deployment,svc
        NAME                                      IMAGE REPOSITORY...output omitted...
        imagestream.image.openshift.io/db-image   image-registry.openshift...output omitted...
        NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/db-image   1/1     1            1           46m
        
        NAME               TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
        service/db-image   ClusterIP   172.30.71.0   <none>        3306/TCP   46m
        ```
        

# **Guided Exercise: Manage Long-lived and Short-lived Applications by Using the Kubernetes Workload API**

1. As the `developer` user, create a MySQL deployment in a new project.
    1. Log in as the `developer` user with the `developer` password.
        
        ```bash
        [student@workstation ~]$oc login -u developer -p developer https://api.ocp4.example.com:6443
          ...output omitted...
        ```
        
    2. Create a project named `deploy-workloads`.
        
        ```bash
        [student@workstation ~]$oc new-project deploy-workloads
        Now using project "deploy-workloads" on server "https://api.ocp4.example.com:6443".
        ...output omitted...
        ```
        
    3. Create a deployment that runs an ephemeral MySQL server.
        
        ```bash
        [student@workstation ~]$oc create deployment my-db --image registry.ocp4.example.com:8443/rhel9/mysql-80:1
        Warning: would violate PodSecurity "restricted:v1.24"
        ...output omitted...
        deployment.apps/my-db created
        ```
        
        <aside>
        <img src="https://www.notion.so/icons/light-bulb_blue.svg" alt="https://www.notion.so/icons/light-bulb_blue.svg" width="40px" /> It is safe to ignore pod security warnings for exercises in this course. OpenShift uses the Security Context Constraints controller to provide safe defaults for pod security.
        
        </aside>
        
    4. Retrieve the status of the deployment.
        
        ```bash
        [student@workstation ~]$oc get deployments
        NAME    READY   UP-TO-DATE   AVAILABLE   AGE
        my-db0/1     1            0           67s
        ```
        
        The deployment never has a ready instance.
        
    5. Retrieve the status of the created pod. Your pod name might differ from the output.
        
        ```bash
        [student@workstation ~]$oc get pods
        NAME                     READY   STATUS             RESTARTS      AGE
        my-db-8567b478dd-d28f7   0/1CrashLoopBackOff   4 (60s ago)   2m35s
        ```
        
        The pod fails to start and repeatedly crashes.
        
    6. Review the logs for the pod to determine why it fails to start.
        
        ```bash
        [student@workstation ~]$oc logs deploy/my-db
        ...output omitted...You must either specify the following environment variables:
          MYSQL_USER (regex: '^$')
          MYSQL_PASSWORD (regex: '^[a-zA-Z0-9_~!@#$%^&*()-=<>,.?;:|]$')
          MYSQL_DATABASE (regex: '^$')
        Or the following environment variable:
          MYSQL_ROOT_PASSWORD (regex: '^[a-zA-Z0-9_~!@#$%^&*()-=<>,.?;:|]$')
        ...output omitted...
        ```
        
        Note that the container fails to start due to missing environment variables.
        
2. Fix the database deployment and verify that the server is running.
    1. Set the `MYSQL_USER`, `MYSQL_PASSWORD`, and `MYSQL_DATABASE` environment variables.
        
        ```bash
        [student@workstation ~]$oc set env deployment/my-db MYSQL_USER=developer MYSQL_PASSWORD=developer \
          MYSQL_DATABASE=sampledb
        deployment.apps/my-db updated 
        ```
        
    2. Retrieve the list of deployments and observe that the `my-db` deployment has a running pod.
        
        ```bash
        [student@workstation ~]$oc get deployments
        NAME    READY   UP-TO-DATE   AVAILABLE   AGE
        my-db1/1     1            1           4m50s
        ```
        
    3. Retrieve the internal IP address of the MySQL pod within the list of all pods.
        
        ```bash
        [student@workstation ~]$oc get pods -o wide
        NAME                     READY   STATUS    RESTARTS   AGE   IP        ...
        my-db-748c97d478-g8xc9   1/1     Running   0          64s   10.8.0.91 ...
        ```
        
        The `-o wide` option enables additional output, such as IP addresses. Your IP address value might differ from the previous output.
        
    4. Verify that the database server is running, by running a query. Replace the IP address with the one that you retrieved in the preceding step.
        
        ```bash
        [student@workstation ~]$oc run -it db-test --restart=Never \
          --image registry.ocp4.example.com:8443/rhel9/mysql-80:1 \
          -- mysql sampledb -h10.8.0.91 -u developer --password=developer \
          -e "select 1;"...output omitted...
        ---
        | 1 |
        ---
        | 1 |
        ---
        ```
        
3. Delete the database server pod and observe that the deployment causes the pod to be re-created.
    1. Delete the existing MySQL pod by using the label that is associated with the deployment.
        
        ```bash
        [student@workstation ~]$oc delete pod -l app=my-db
        pod "my-db-84c8995d5-2sssl" deleted
        ```
        
    2. Retrieve the information for the MySQL pod and observe that it is newly created. Your pod name might differ in your output.
        
        ```bash
        [student@workstation ~]$oc get pod -l app=my-db
        NAME                    READY   STATUS    RESTARTS   AGE
        my-db-fbccb9447-p99jd   1/1     Running   06s
        ```
        
4. Create and apply a job resource that prints the time and date repeatedly.
    1. Create a job resource called `date-loop` that runs a script. Ignore the warning.
        
        ```bash
        [student@workstation ~]$oc create job date-loop \
          --image registry.ocp4.example.com:8443/ubi9/ubi \
          -- /bin/bash -c "for i in {1..30}; do date; done"
        job.batch/date-loop created
        ```
        
    2. Retrieve the job resource to review the pod specification.
        
        ```bash
        [student@workstation ~]$oc get job date-loop -o yaml...output omitted...
            spec:
              containers:
              - command: 1
                - /bin/bash
                - -c
                - for i in {1..30}; do date; done
                image: registry.ocp4.example.com:8443/ubi9/ubi 2
                imagePullPolicy: Always
                name: date-loop
                resources: {}
                terminationMessagePath: /dev/termination-log
                terminationMessagePolicy: File
              dnsPolicy: ClusterFirst
              restartPolicy: Never 3
              schedulerName: default-scheduler
              securityContext: {}
              terminationGracePeriodSeconds: 30
        ...output omitted...
        ```
        
        1. The `command` object, which specifies the defined script to execute within the pod.
        2. Sets the container image for the pod.
        3. Defines the restart policy for the pod. Kubernetes does not restart the job pod after the pod exits.
        
    3. List the jobs to see that the `date-loop` job completed successfully.
        
        ```
        [student@workstation ~]$oc get jobs
        NAME          COMPLETIONS   DURATION   AGE
        date-loop1/1           7s         8s
        ```
        
        You might need to wait for the script to finish and run the command again.
        
    4. Retrieve the logs for the associated pod. The log values might differ in your output.
        
        ```
        [student@workstation ~]$oc logs job/date-loop
        Fri Nov 18 14:50:56 UTC 2022
        Fri Nov 18 14:50:59 UTC 2022
        ...output omitted...
        ```
        
5. Delete the pod for the `date-loop` job and observe that the pod is not created again.
    1. Delete the associated pod.
        
        ```
        [student@workstation ~]$oc delete pod -l job-name=date-loop
        pod "date-loop-wvn2q" deleted
        ```
        
    2. View the list of pods and observe that the pod is not re-created for the job.
        
        ```
        [student@workstation ~]$oc get pod -l job-name=date-loop
        No resources found in deploy-workloads namespace.
        ```
        
    3. Verify that the job status is still listed as successfully completed.
        
        ```
        [student@workstation ~]$oc get job -l job-name=date-loop
        NAME        COMPLETIONS   DURATION   AGE
        date-loop1/1           7s         7m36s
        ```
        

# **Guided Exercise: Kubernetes Pod and Service Networks**

1. Log in to the OpenShift cluster as the `developer` user with the `developer` password. Use the `deploy-services` project.
    1. Log in to the OpenShift cluster.
        
        ```bash
        [student@workstation ~]$oc login -u developer -p developer https://api.ocp4.example.com:6443
        Login successful.
        ...output omitted...
        ```
        
    2. Set the `deploy-services` project as the active project.
        
        ```bash
        [student@workstation ~]$oc project deploy-services
        ...output omitted...
        ```
        
2. Use the `registry.ocp4.example.com:8443/rhel8/mysql-80` container image to create a MySQL deployment named `db-pod`. Add the missing environment variables for the pod to run.
    1. Create the `db-pod` deployment.
        
        ```bash
        [student@workstation ~]$oc create deployment db-pod --port 3306 --image registry.ocp4.example.com:8443/rhel8/mysql-80
        deployment.apps/db-pod created
        ```
        
    2. Add the environment variables.
        
        ```bash
        [student@workstation ~]$oc set env deployment/db-pod MYSQL_USER=user1  MYSQL_PASSWORD=mypa55w0rd MYSQL_DATABASE=items
        deployment.apps/db-pod updated
        ```
        
    3. Confirm that the pod is running.
        
        ```
        [student@workstation ~]$oc get pods
        NAME                      READY   STATUS    RESTARTS   AGE
        db-pod-6ccc485cfc-vrc4r   1/1     Running   0          2m30s
        ```
        
        Your pod name might differ from the previous output.
        
3. Expose the `db-pod` deployment to create a ClusterIP service.
    1. View the deployment for the pod.
        
        ```bash
        [student@workstation ~]$oc get deployment
        NAME     READY   UP-TO-DATE   AVAILABLE   AGE
        db-pod   1/1     1            1           3m36s
        ```
        
    2. Expose the `db-pod` deployment to create a service.
        
        ```bash
        [student@workstation ~]$oc expose deployment/db-pod
        service/db-pod exposed
        ```
        
4. Validate the service. Confirm that the service selector matches the label on the pod. Then, confirm that the `db-pod` service endpoint matches the IP of the pod.
    1. Identify the selector for the `db-pod` service. Use the `oc get service command` with the `o wide` option to retrieve the selector that the service uses.
        
        ```bash
        [student@workstation ~]$oc get service db-pod -o wide
        NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE    SELECTOR
        db-pod   ClusterIP   172.30.108.92   <none>        3306/TCP   108s
        app=db-pod
        ```
        
        The selector shows an `app=db-pod` key:value pair.
        
    2. Capture the name of the pod in a variable.
        
        ```bash
        [student@workstation ~]$ PODNAME=$(oc get pods -o jsonpath='{.items[0].metadata.name}')
        ```
        
    3. Query the label on the pod.
        
        ```bash
        [student@workstation ~]$oc get pod $PODNAME --show-labels
        NAME                     READY   STATUS    RESTARTS   AGE     LABELS
        db-pod-6ccc485cfc-vrc4r  1/1     Running   0          6m50s
        app=db-pod ...
        ```
        
        Notice that the label list includes the `app=db-pod` key-value pair, which is the selector for the `db-pod` service.
        
    4. Retrieve the endpoints for the `db-pod` service.
        
        ```bash
        [student@workstation ~]$oc get endpoints
        NAME     ENDPOINTS        AGE
        db-pod10.8.0.85:3306   4m38s
        ```
        
        Your endpoints values might differ from the previous output.
        
    5. Verify that the service endpoint matches the `db-pod` IP address. Use the `oc get pods` command with the `o wide` option to view the pod IP address.
        
        ```bash
        [student@workstation ~]$ oc get pods -o wide
        NAME                      READY   STATUS    RESTARTS   AGE   IP        ...
        db-pod-6ccc485cfc-vrc4r   1/1     Running   0          54m10.8.0.85 ...
        ```
        
        The service endpoint resolves to the IP address that is assigned to the pod.
        
5. Delete and then re-create the `db-pod` deployment. Confirm that the `db-pod` service endpoint automatically resolves to the IP address of the new pod.
    1. Delete the `db-pod` deployment.
        
        ```bash
        [student@workstation ~]$oc delete deployment.apps/db-pod
        deployment.apps "db-pod" deleted
        ```
        
    2. Verify that the service still exists without the deployment.
        
        ```bash
        [student@workstation ~]$oc get service
        NAME     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
        db-pod   ClusterIP   172.30.108.92  <none>        3306/TCP   9m53s
        ```
        
    3. The list of endpoints for the service is now empty.
        
        ```bash
        [student@workstation ~]$oc get endpoints
        NAME     ENDPOINTS   AGE
        db-pod   <none>      12m
        ```
        
    4. Re-create the `db-pod` deployment.
        
        ```bash
        [student@workstation ~]$oc create deployment db-pod --port 3306 --image registry.ocp4.example.com:8443/rhel8/mysql-80
        deployment.apps/db-pod created
        ```
        
    5. Add the environment variables.
        
        ```bash
        [student@workstation ~]$oc set env deployment/db-pod MYSQL_USER=user1 MYSQL_PASSWORD=mypa55w0rd MYSQL_DATABASE=items
        deployment.apps/db-pod updated
        ```
        
    6. Confirm that the newly created pod has the `app=db-pod` selector.
        
        ```bash
        [student@workstation ~]$oc get pods --selector app=db-pod -o wide
        NAME                    READY   STATUS    RESTARTS   AGE   IP        ...
        db-pod-6ccc485cfc-l2x   1/1     Running   0          32s   10.8.0.85 ...
        ```
        
        Notice the change in the pod name. The pod IP address might also change. Your pod name and IP address might differ from the previous output.
        
    7. Confirm that the endpoints for the `db-pod` service include the newly created pod.
        
        ```bash
        [student@workstation ~]$oc get endpoints
        NAME     ENDPOINTS        AGE
        db-pod10.8.0.85:3306   16m
        ```
        
6. Create a pod to identify the available DNS name assignments for the service.
    1. Create a pod named `shell` to use for troubleshooting. Use the `oc run` command and the `registry.ocp4.example.com:8443/openshift4/network-tools-rhel8` container image.
        
        ```bash
        [student@workstation ~]$oc run shell -it --image registry.ocp4.example.com:8443/openshift4/network-tools-rhel8
        If you don't see a command prompt, try pressing enter.
        bash-4.4$
        ```
        
    2. From the prompt inside the `shell` pod, view the `/etc/resolv.conf` file to identify the cluster-domain name.
        
        ```bash
        bash-4.4$cat /etc/resolv.conf
        search deploy-services.svc.cluster.local svc.cluster.local ...
        nameserver 172.30.0.10
        options ndots:5
        ```
        
        The container uses the values from the `search` directive as suffix values on DNS searches. The container appends these values to a DNS query, in the written order, to resolve the search. The cluster-domain name is the last few components of these values that start after `svc`.
        
    3. Use the `nc` and `echo` commands to test the available DNS names for the service.
        
        ```bash
        nc -z<service>_<server>_<port>
        ```
        
        The long version of the DNS name is required when accessing the service from a different project. When the pod is in the same project, you can use a shorter version of the DNS name.
        
        ```bash
        bash-4.4$ nc -z db-pod.deploy-services 3306 &&
         echo "Connection success to db-pod.deploy-services:3306" || echo "Connection failed"
        Connection success to db-pod.deploy-services:3306
        ```
        
    4. Exit the interactive session.
        
        ```bash
        bash-4.4$exit
        Session ended, resume using 'oc attach shell -c shell -i -t' command when the pod is running
        ```
        
    5. Delete the pod for the shell.
        
        ```bash
        [student@workstation ~]$ oc delete pod shell
        pod "shell" deleted
        ```
        
7. Use a new project to test pod communications across namespaces.
    1. Create a second namespace with the `oc new-project` command.
        
        ```bash
        [student@workstation ~]$ oc new-project deploy-services-2
        Now using project "deploy-services-2" on server "https://api.ocp4.example.com:6443".
        ...output omitted...
        ```
        
    2. Execute the `nc` and `echo` commands from a pod to test the DNS name access to another namespace.
        
        ```bash
        [student@workstation ~]$ oc run shell -it --rm --image registry.ocp4.example.com:8443/openshift4/network-tools-rhel8 --restart Never -- nc -z db-pod.deploy-services.svc.cluster.local 3306 && echo "Connection success to db-pod.deploy-services.svc.cluster.local:3306" || echo "Connection failed"
        pod "shell" deleted
        Connection success to db-pod.deploy-services.svc.cluster.local:3306
        ```
        
    3. Return to the `deploy-services` project.
        
        ```bash
        [student@workstation ~]$oc project deploy-services
        Now using project "deploy-services" on server "https://api.ocp4.example.com:6443".
        ```
        
8. Use a Kubernetes job to add initialization data to the database.
    1. Create a job named `mysql-init` that uses the `registry.ocp4.example.com:8443/redhattraining/do180-dbinit:v1` container image. This image uses the `mysql-80` container image as a base image, and it includes a script that adds a few initial records to the database.
        
        ```bash
        [student@workstation ~]$ oc create job mysql-init --image registry.ocp4.example.com:8443/redhattraining/do180-dbinit:v1 -- /bin/bash -c "mysql -uuser1 -pmypa55w0rd --protocol tcp -h db-pod -P3306 items </tmp/db-init.sql"
        job.batch/mysql-init created
        ```
        
        The `-h` option of the `mysql` command directs the command to communicate with the DNS short name of the `db-pod` service. The `db-pod` short name can be used here, because the pod for the job is created in the same namespace as the service.
        
        The double dash `--` before `/bin/bash` separates the `oc` command arguments from the command in the pod. The `-c` option of `/bin/bash` directs the command interpreter in the container to execute the command string. The `/tmp/db-init.sql` file is redirected as input for the command. The `db-init.sql` file is included in the image, and contains the following script.
        
        ```sql
        DROP TABLE IF EXISTS `Item`;
        CREATE TABLE `Item` (`id` BIGINT not null auto_increment primary key, `description` VARCHAR(100), `done` BIT);
        INSERT INTO `Item` (`id`,`description`,`done`) VALUES (1,'Pick up newspaper', 0);
        INSERT INTO `Item` (`id`,`description`,`done`) VALUES (2,'Buy groceries', 1);
        ```
        
    2. Confirm the status of the `mysql-init` job. Wait for the job to complete.
        
        ```bash
        [student@workstation ~]$oc get job
        NAME         COMPLETIONS   DURATION   AGE
        mysql-init1/1                      22m
        ```
        
    3. Retrieve the status of the `mysql-init` job pod, to confirm that the pod has a `Completed` status.
        
        ```bash
        [student@workstation ~]$ oc get pods
        NAME                      READY   STATUS      RESTARTS      AGE
        db-pod-6ccc485cfc-2lklx   1/1     Running     0             4h24m
        mysql-init-ln9cg          0/1Completed   0             23m
        ```
        
    4. Delete the `mysql-init` job, because it is no longer needed.
        
        ```bash
        [student@workstation ~]$ oc delete job mysql-init
        job.batch "mysql-init" deleted
        ```
        
    5. Verify that the corresponding `mysql-init` pod is also deleted.
        
        ```bash
        [student@workstation ~]$ oc get pods
        NAME                      READY   STATUS    RESTARTS      AGE
        db-pod-6ccc485cfc-2lklx   1/1     Running   0             4h2
        ```
        
9. Create a `query-db` pod by using the `oc run` command and the `registry.ocp4.example.com:8443/redhattraining/do180-dbinit:v1` container image. Use the pod to execute a query against the database service.
    1. Create the `query-db` pod. Configure the pod to use the MySQL client to execute a query against the `db-pod` service. You can use the `db-pod` service short name, which provides a stable reference.
        
        ```bash
        [student@workstation ~]$ oc run query-db -it --rm --image registry.ocp4.example.com:8443/redhattraining/do180-dbinit:  --restart Never -- mysql -uuser1 -pmypa55w0rd --protocol tcp -h db-pod -P3306 items -e 'select * from Item;'
        mysql: [Warning] Using a password on the command line interface can be insecure.
        +----+-------------------+------------+
        | id | description       | done       |
        +----+-------------------+------------+
        |  1 | Pick up newspaper | 0x00       |
        |  2 | Buy groceries     | 0x01       |
        +----+-------------------+------------+
        pod "query-db" deleted
        ```
        
10. It might be necessary to use pod-to-pod communications for troubleshooting. Use the `oc run` command to create a pod that executes a network test against the IP address of the database pod.
    1. Confirm the IP address of the MySQL database pod. Your pod IP address might differ from the output.
        
        ```bash
        [student@workstation ~]$ oc get pods -o wide
        NAME                     READY  STATUS     RESTARTS     AGE  IP         ...
        db-pod-6ccc485cfc-2lklx  1/1    Running    0            4h5  10.8.0.69  ...
        ```
        
    2. Capture the IP address in an environment variable.
        
        ```bash
        [student@workstation ~]$ POD_IP=$(oc get pod -l app=db-pod -o jsonpath='{.items[0].status.podIP}')
        ```
        
    3. Create a test pod named `shell` with the `oc run` command. Execute the `nc` command to test against the `$POD_IP` environment variable and the `3306` port for the database.
        
        ```bash
        [student@workstation ~]$ oc run shell --env POD_IP=$POD_IP -it --rm   --image registry.ocp4.example.com:8443/openshift4/network-tools-rhel8   --restart Never  -- nc -z $POD_IP 3306 && echo "Connection success to $POD_IP:3306" \
         || echo "Connection failed"
        pod "shell" deleted
        Connection success to 10.8.0.69:3306
        ```
        

# **Guided Exercise: Scale and Expose Applications to External Access**

1. Create two web application deployments, named `satir-app` and `sakila-app`. Use the  `registry.ocp4.example.com:8443/httpd-app:v1` container image for both deployments.
    1. Log in to the OpenShift cluster as the `developer` user with the `developer` password.
        
        ```bash
        [student@workstation ~]  oc login -u developer -p developer https://api.ocp4.example.com:6443
        Login successful
        ...output omitted...
        ```
        
    2. Change to the `web-applications` project.
        
        ```bash
        [student@workstation ~]$ oc project web-applications
        Now using project "web-applications" on server "https://api.ocp4.example.com:6443".
        ...output omitted...
        ```
        
    3. Create the `satir-app` web application deployment by using the `registry.ocp4.example.com:8443/redhattraining/do180-httpd-app:v1` container image. Ignore the warning message.
        
        ```bash
        [student@workstation ~]$ oc create deployment satir-app --image registry.ocp4.example.com:8443/redhattraining/do180-httpd-app:v1
        deployment.apps/satir-app created
        ```
        
    4. After a few moments, verify that the deployment is successful.
        
        ```bash
        [student@workstation ~]$ oc get pods
        NAME                         READY   STATUS    RESTARTS   ...
        satir-app-787b7d7858-5dfsh   1/1     Running   0          ...
        ```
        
        ```bash
        [student@workstation ~]$ oc status
        ...output omitted...
        deployment/satir-app deploys registry.ocp4.example.com:8443/redhattraining/do180-httpd-app:v1
          deployment #1 running for 20 seconds - 1 pod
        ...output omitted...
        ```
        
    5. Create the `sakila-app` web application deployment by using the `registry.ocp4.example.com:8443/redhattraining/do180-httpd-app:v1` image. Ignore the warning message.
        
        ```bash
        [student@workstation ~]$ oc create deployment sakila-app --image registry.ocp4.example.com:8443/redhattraining/do180-httpd-app:v1
        deployment.apps/sakila-app created
        ```
        
    6. Wait a few moments and then verify that the deployment is successful.
        
        ```bash
        [student@workstation ~]$ oc get pods
        NAME                     READY   STATUS    RESTARTS   ...
        sakila-app-6694...5kpd   1/1     Running   0          ...
        satir-app-787b7...dfsh   1/1     Running   0          ...
        ```
        
        ```bash
        [student@workstation ~]$ oc status
        ...output omitted...
        deployment/satir-app deploys registry.ocp4.example.com:8443/redhattraining/do180-httpd-app:v1
          deployment #1 running for 5 minutes - 1 pod
        
        deployment/sakila-app deploys registry.ocp4.example.com:8443/redhattraining/do180-httpd-app:v1
          deployment #1 running for 2 minutes - 1 pod
        ...output omitted...
        ```
        
2. Create services for the web application deployments. Then, use the services to create a route for the `satir-app` application and an ingress object for the `sakila-app` application.
    1. Expose the `satir-app` deployment. Name the service `satir-svc`, and specify port `8080` as the port and target port.
        
        ```bash
        [student@workstation ~]$ oc expose deployment satir-app --name satir-svc --port 8080 --target-port 8080
        service/satir-svc exposed
        ```
        
    2. Expose the `sakila-app` deployment to create the `sakila-svc` service.
        
        ```bash
        [student@workstation ~]$ oc expose deployment sakila-app --name sakila-svc --port 8080 --target-port 8080
        service/sakila-svc exposed
        ```
        
    3. Verify the status of the services.
        
        ```bash
        [student@workstation ~]$ oc get services
        NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    ...
        sakila-svc   ClusterIP   172.30.230.41   <none>        8080/TCP   ...
        satir-svc    ClusterIP   172.30.143.15   <none>        8080/TCP   ...
        ```
        
        ```bash
        [student@workstation ~]$ oc get endpoints
        NAME         ENDPOINTS        ...
        sakila-svc   10.8.0.66:8080   ...
        satir-svc    10.8.0.65:8080   ...
        ```
        
        ```bash
        [student@workstation ~]$ oc get pods -o wide
        NAME                    READY  STATUS   RESTARTS  AGE   IP        ...
        sakila-app-6694...5kpd  1/1    Running  0         92s   10.8.0.66 ...
        satir-app-787b7...dfsh  1/1    Running  0         2m49s 10.8.0.65 ...
        ```
        
    4. Create a route named `satir` for the `satir-app` web application by exposing the `satir-svc` service.
        
        ```bash
        [student@workstation ~]$oc expose service satir-svc --name satir
        route.route.openshift.io/satir exposed
        ```
        
        ```bash
        [student@workstation ~]$oc get routes
        NAME    HOST/PORT                                    ... SERVICES    PORT   ...
        satir   satir-web-applications.apps.ocp4.example.com ... satir-svc   8080   ...
        ```
        
    5. Create an ingress object named `ingr-sakila` for the `sakila-svc` service. Configure the `-rule` option with the following values:
        
        
        | Field | Value |
        | --- | --- |
        | Host | ingr-sakila.apps.ocp4.example.com |
        | Service name | sakila-svc |
        | Port number | 8080 |
        
        ```bash
        [student@workstation ~]$oc create ingress ingr-sakila --rule "ingr-sakila.apps.ocp4.example.com/*=sakila-svc:8080" 
        ingress.networking.k8s.io/ingr-sakila created
        ```
        
        ```bash
        [student@workstation ~] $oc get ingress
        NAME        ... HOSTS                              ADDRESS       PORTS ...
        ingr-sakila ... ingr-sakila.apps.ocp4.example.com  router...com  80    ...
        ```
        
    6. Confirm that a route exists for the `ingr-sakila` ingress object.
        
        ```bash
        [student@workstation ~]$ oc get routes
        NAME           HOST/PORT                                    ... SERVICES   PORT
        ingr-sakila... ingr-sakila.apps.ocp4.example.com            ... sakila-svc <all>
        satir          satir-web-applications.apps.ocp4.example.com ... satir-svc  8080
        ```
        
        A specific port is not assigned to routes that ingress objects created. By contrast, a route that an exposed service created is assigned the same ports as the service.
        
    7. Use the `curl` command to access the `ingr-sakila` ingress object and the `satir` route. The output states the name of the pod that is servicing the request.
        
        ```
        [student@workstation ~]$curl ingr-sakila.apps.ocp4.example.com
        Welcome to Red Hat Training, from sakila-app-66947cdd78-x5kpd
        ```
        
        ```
        [student@workstation ~]$curl satir-web-applications.apps.ocp4.example.com
        Welcome to Red Hat Training, from satir-app-787b7d7858-bdfsh
        ```
        
3. Scale the web application deployments to load-balance their services. Scale the `sakila-app` to two replicas, and the `satir-app` to three replicas.
    1. Scale the `sakila-app` deployment with two replicas.
        
        ```bash
        [student@workstation ~]$ oc scale deployment sakila-app --replicas 2
        deployment.apps/sakila-app scaled
        ```
        
    2. Wait a few moments and then verify the status of the replica pods.
        
        ```bash
        [student@workstation ~]$ oc get pods
        NAME                     READY   STATUS    RESTARTS   ...
        sakila-app-6694...5kpd   1/1     Running   0          ...
        sakila-app-6694...rfzg   1/1     Running   0          ...
        satir-app-787b...dfsh    1/1     Running   0          ...
        ```
        
    3. Scale the `satir-app` deployment with three replicas.
        
        ```bash
        [student@workstation ~]$ oc scale deployment satir-app --replicas 3
        deployment.apps/satir-app scaled
        ```
        
    4. Wait a few moments and then verify the status of the replica pods.
        
        ```bash
        [student@workstation ~]$ oc get pods -o wide
        NAME                    READY  STATUS   RESTARTS  ... IP        ...
        sakila-app-6694...5kpd  1/1    Running  0         ... 10.8.0.66 ...
        sakila-app-6694...rfzg  1/1    Running  0         ... 10.8.0.67 ...
        satir-app-787b...dfsh   1/1    Running  0         ... 10.8.0.65 ...
        satir-app-787b...z8xm   1/1    Running  0         ... 10.8.0.69 ...
        satir-app-787b...7bhj   1/1    Running  0         ... 10.8.0.70 ...
        ```
        
    5. Retrieve the service endpoints to confirm that the services are load-balanced between the additional replica pods.
        
        ```bash
        [student@workstation ~]$ oc get endpoints
        NAME         ENDPOINTS                                     ...
        sakila-svc   10.8.0.66:8080,10.8.0.67:8080                 ...
        satir-svc    10.8.0.65:8080,10.8.0.69:8080,10.8.0.70:8080  ...
        ```
        
4. Enable the sticky sessions for the `sakila-app` web application. Then, use the `curl` command to confirm that the sticky sessions are working for the `ingr-sakila` object.
    1. Configure a cookie for the `ingr-sakila` ingress object.
        
        ```bash
        [student@workstation ~]$ oc annotate ingress ingr-sakila ingress.kubernetes.io/affinity=cookie
        ingress.networking.k8s.io/ingr-sakila annotated
        ```
        
    2. Use the `curl` command to access the `ingr-sakila` ingress object. The output states the name of the pod that is servicing the request. Notice that the connection is load-balanced between the replicas.
        
        ```bash
        [student@workstation ~]$ for i in {1..3}; do curl ingr-sakila.apps.ocp4.example.com ; done
        Welcome to Red Hat Training, from sakila-app-66947cdd78-x5kpd
        Welcome to Red Hat Training, from sakila-app-66947cdd78-xrfzg
        Welcome to Red Hat Training, from sakila-app-66947cdd78-x5kpd
        ```
        
    3. Use the `curl` command to save the `ingr-sakila` ingress object cookie to the `/tmp/cookie_jar` file. Confirm that the cookie exists in the `/tmp/cookie_jar` file.
        
        ```bash
        [student@workstation ~]$ curl ingr-sakila.apps.ocp4.example.com -c /tmp/cookie_jar
        Welcome to Red Hat Training, from sakila-app-66947cdd78-xrfzg
        ```
        
        ```bash
        [student@workstation ~] $cat /tmp/cookie_jar
        ...output omitted...
        #HttpOnly_ingr-sakila.apps.ocp4.example.com	FALSE	/	FALSE	0	b9b484110526b4b1b3159860d3aebe04	921e139c5145950d00424bf3b0a46d22
        ```
        
    4. The cookie provides session stickiness for connections to the `ingr-sakila` route. Use the `curl` command and the cookie in the `/tmp/cookie_jar` file to connect to the `ingr-sakila` route again. Confirm that you are connected to the same pod that handled the request in the previous step.
        
        ```bash
        [student@workstation ~]$ for i in {1..3}; do curl ingr-sakila.apps.ocp4.example.com -b /tmp/cookie_jar; done
        Welcome to Red Hat Training, from sakila-app-66947cdd78-xrfzg
        Welcome to Red Hat Training, from sakila-app-66947cdd78-xrfzg
        Welcome to Red Hat Training, from sakila-app-66947cdd78-xrfzg
        ```
        
    5. Use the `curl` command to connect to the `ingr-sakila` route without the cookie. Observe that session stickiness occurs only with the cookie.
        
        ```bash
        [student@workstation ~]$for i in {1..3}; do curl ingr-sakila.apps.ocp4.example.com ; done
        Welcome to Red Hat Training, from sakila-app-66947cdd78-x5kpd
        Welcome to Red Hat Training, from sakila-app-66947cdd78-xrfzg
        Welcome to Red Hat Training, from sakila-app-66947cdd78-x5kpd
        ```
        
5. Enable the sticky sessions for the `satir-app` web application. Then, use the `curl` command to confirm that sticky sessions are active for the `satir` route.
    1. Configure a cookie with a `hello` value for the `satir` route.
        
        ```bash
        [student@workstation ~]$oc annotate route satir router.openshift.io/cookie_name="hello"
        route.route.openshift.io/satir annotated
        ```
        
    2. Use the `curl` command to access the `satir` route. The output states the name of the pod that is servicing the request. Notice that the connection is load-balanced between the three replica pods.
        
        ```bash
        [student@workstation ~]$for i in {1..3}; do  curl satir-web-applications.apps.ocp4.example.com; done
        Welcome to Red Hat Training, from satir-app-787b7d7858-bdfsh
        Welcome to Red Hat Training, from satir-app-787b7d7858-gz8xm
        Welcome to Red Hat Training, from satir-app-787b7d7858-q7bhj
        ```
        
    3. Use the `curl` command to save the `hello` cookie to the `/tmp/cookie_jar` file. Afterward, confirm that the `hello` cookie exists in the `/tmp/cookie_jar` file.
        
        ```bash
        [student@workstation ~]$curl satir-web-applications.apps.ocp4.example.com -c /tmp/cookie_jar
        Welcome to Red Hat Training, from satir-app-787b7d7858-q7bhj
        ```
        
        ```bash
        [student@workstation ~]$cat /tmp/cookie_jar
        ...output omitted...
        #HttpOnly_satir-web-applications.apps.ocp4.example.com	FALSE	/	FALSE	0	hello	b7dd73d32003e513a072e25a32b6c881
        ```
        
    4. The `hello` cookie provides session stickiness for connections to the `satir` route. Use the `curl` command and the `hello` cookie in the `/tmp/cookie_jar` file to connect to the `satir` route again. Confirm that you are connected to the same pod that handled the request in the previous step.
        
        ```bash
        [student@workstation ~]$for i in {1..3}; do curl satir-web-applications.apps.ocp4.example.com -b /tmp/cookie_jar; done
        Welcome to Red Hat Training, from satir-app-787b7d7858-q7bhj
        Welcome to Red Hat Training, from satir-app-787b7d7858-q7bhj
        Welcome to Red Hat Training, from satir-app-787b7d7858-q7bhj
        ```
        
    5. Use the `curl` command to connect to the `satir` route without the `hello` cookie. Observe that session stickiness occurs only with the cookie.
        
        ```bash
        [student@workstation ~]$for i in {1..3}; do  curl satir-web-applications.apps.ocp4.example.com; done
        Welcome to Red Hat Training, from satir-app-787b7d7858-gz8xm
        Welcome to Red Hat Training, from satir-app-787b7d7858-q7bhj
        Welcome to Red Hat Training, from satir-app-787b7d7858-bdfsh
        ```
        

# **Lab: Deploy Managed and Networked Applications on Kubernetes**

1. Log in to the OpenShift cluster and change to the `database-applications` project
    1. Log in to the OpenShift cluster.
        
        ```bash
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
        
2. Create a MySQL database deployment named `mysql-app` by using the `registry.ocp4.example.com:8443/redhattraining/mysql-app:v1` image, and identify the root cause of the failure.
    1. Create the MySQL database deployment. Ignore the warning message.
        
        ```
        [student@workstation ~]$oc create deployment mysql-app \
          --image registry.ocp4.example.com:8443/redhattraining/mysql-app:v1
        deployment.apps/mysql-app created
        ```
        
    2. Verify the deployment status. The pod name might differ in your output.
        
        ```
        [student@workstation ~]$oc get pods
        NAME              	          READY  STATUS  ...
        mysql-app-75dfd58f99-5xfqc   0/1    Error   ...
        [student@workstation ~]$oc status...output omitted...
        Errors:
          pod/mysql-app-75dfd58f99-5xfqc is crash-looping
        
        1 error, 1 info identified, use 'oc status --suggest' to see details.
        ```
        
    3. Identify the root cause of the deployment failure.
        
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
        
3. Configure the environment variables for the `mysql-app` deployment by using the following information:
    
    
    | Field | Value |
    | --- | --- |
    | MYSQL_USER | redhat |
    | MYSQL_PASSWORD | redhat123 |
    | MYSQL_DATABASE | world_x |
    
    Then, execute the following command in the `mysql-app` deployment pod to load the `world_x` database:
    
    ```bash
    
    /bin/bash -c "mysql -uredhat -predhat123 </tmp/world_x.sql"
    ```
    
    1. Update the environment variables for the `mysql-app` deployment.
        
        ```
        [student@workstation ~]$oc set env deployment/mysql-app \
          MYSQL_USER=redhat MYSQL_PASSWORD=redhat123 MYSQL_DATABASE=world_x
        deployment.apps/mysql-app updated
        ```
        
    2. Verify that the `mysql-app` application pod is in the `RUNNING` state. The pod name might differ in your output.
        
        ```
        [student@workstation ~]$oc get pods
        NAME		                      READY   STATUS   ...
        mysql-app-57c44f646-5qt2k   1/1     Running  ...
        ```
        
    3. Load the `world_x` database.
        
        ```
        [student@workstation ~]$oc exec -it mysql-app-57c44f646-5qt2k \
          -- /bin/bash -c "mysql -uredhat -predhat123 </tmp/world_x.sql"
        ...output omitted..
        [student@workstation ~]$
        ```
        
    4. Confirm that you can access the MySQL database.
        
        ```
        [student@workstation ~]$oc rsh mysql-app-57c44f646-5qt2k
        ```
        
        ```
        sh-4.4$mysql -uredhat -predhat123 world_x...output omitted...
        mysql>
        ```
        
    5. Exit the MySQL database, and then exit the container.
        
        ```
        mysql>exit
        Bye
        sh-4.4$exit
        ```
        
4. Create a service for the `mysql-app` deployment by using the following information:
    
    
    | Field | Value |
    | --- | --- |
    | Name | mysql-service |
    | Port | 3306 |
    | Target port | 3306 |
    1. Expose the `mysql-app` deployment.
        
        ```
        [student@workstation ~]$oc expose deployment mysql-app --name mysql-service \
          --port 3306 --target-port 3306
        service/mysql-service created
        ```
        
    2. Verify the service configuration. The endpoint IP address might differ in your output.
        
        ```
        [student@workstation ~]$oc get services
        NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
        mysql-service   ClusterIP   172.30.146.213   <none>        3306/TCP   10s
        [student@workstation ~]$oc get endpoints
        NAME            ENDPOINTS         AGE
        mysql-service   10.8.0.102:3306   19s
        ```
        
5. Create a web application deployment named `php-app` by using the `registry.ocp4.example.com:8443/redhattraining/php-webapp:v1` image.
    1. Create the web application deployment. Ignore the warning message.
        
        ```
        [student@workstation ~]$oc create deployment php-app \
          --image registry.ocp4.example.com:8443/redhattraining/php-webapp:v1
        deployment.apps/php-app created
        ```
        
    2. Verify the deployment status. Verify that the `php-app` application pod is in the `RUNNING` state.
        
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
        
6. Create a service for the `php-app` deployment by using the following information:
    
    
    | Field | Value |
    | --- | --- |
    | Name | php-svc |
    | Port | 8080 |
    | Target port | 8080 |
    
    Then, create a route named `phpapp` to expose the web application to external access.
    
    1. Expose the `php-app` deployment.
        
        ```
        [student@workstation ~]$oc expose deployment php-app --name php-svc \
          --port 8080 --target-port 8080
        service/php-svc exposed
        ```
        
    2. Verify the service configuration. The endpoint IP address might differ in your output.
        
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
        
    3. Expose the `php-svc` service.
        
        ```
        [student@workstation ~]$oc expose service/php-svc --name phpapp
        route.route.openshift.io/phpapp exposed
        ```
        
        ```
        [student@workstation ~]$oc get routes
        NAME    HOST/PORT                             ...
        phpapp  phpapp-database-applications.apps.ocp4.example.com  ...
        ```
        
7. Test the connectivity between the web application and the MySQL database. In a web browser, navigate to the `phpapp-database-applications.apps.ocp4.example.com` route, and verify that the application retrieves data from the MySQL database.
    1. Navigate to the `phpapp-database-applications.apps.ocp4.example.com` route in the web browser.

![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/deploy/review/assets/phpwebapp.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/deploy/review/assets/phpwebapp.png)