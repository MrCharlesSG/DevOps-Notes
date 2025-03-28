# Lab 6: Configure Applications for Reliability

1. The `longload` application in the `reliability-review` project fails to start. Diagnose and then fix the issue. The application needs 512 MiB of memory to work.
    
    After you fix the issue, you can confirm that the application works by running the `~/DO180/labs/reliability-review/curl_loop.sh` script that the `lab` command prepared. The script sends requests to the application in a loop. For each request, the script displays the pod name and the application status. Press **Ctrl**+**C** to quit the script.
    
    1. Log in to the OpenShift cluster.
        
        ```
        [student@workstation ~]$oc login -u developer -p developer \https://api.ocp4.example.com:6443
        Login successful.
        ...output omitted...
        ```
        
    2. Set the `reliability-review` project as the active project.
        
        ```
        [student@workstation ~]$oc project reliability-review...output omitted...
        ```
        
    3. List the pods in the project. The pod is in the `Pending` status. The name of the pod on your system probably differs.
        
        ```
        [student@workstation ~]$oc get pods
        NAME                        READY   STATUS    RESTARTS   AGE
        longload-64bf8dd776-b6rkz   0/1Pending   0          8m1s
        ```
        
    4. Retrieve the events for the pod. No compute node has enough memory to accommodate the pod.
        
        ```
        [student@workstation ~]$oc describe pod longload-64bf8dd776-b6rkz
        Name:             longload-64bf8dd776-b6rkz
        Namespace:        reliability-review
        ...output omitted...
        Events:
          Type     Reason            Age   From               Message
          ----     ------            ----  ----               -------
          Warning  FailedScheduling  8m    default-scheduler  0/1 nodes are available: 1Insufficient memory. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.
        ```
        
    5. Review the resource requests for memory. The `longload` deployment requests 8 GiB of memory.
        
        ```
        [student@workstation ~]$oc get deployment longload -o \
          jsonpath='{.spec.template.spec.containers[0].resources.requests.memory}{"\n"}'
        8Gi
        ```
        
    6. Set the memory requests to 512 MiB. Ignore the warning message.
        
        ```
        [student@workstation ~]$oc set resources deployment/longload \
          --requests memory=512Mi
        deployment.apps/longload resource requirements updated
        ```
        
    7. Wait for the pod to start. You might have to rerun the command several times for the pod to report a `Running` status. The name of the pod on your system probably differs.
        
        ```
        [student@workstation ~]$oc get pods
        NAME                        READY   STATUS    RESTARTS   AGE
        longload-5897c9558f-cx4gt   1/1Running   0          86s
        ```
        
    8. Run the `~/DO180/labs/reliability-review/curl_loop.sh` script to confirm that the application works.
        
        ```
        [student@workstation ~]$~/DO180/labs/reliability-review/curl_loop.sh
        1 curl: (7) Failed to connect to master01.ocp4.example.com port 30372: Connection refused
        2 longload-5897c9558f-cx4gt: app is still starting
        3 longload-5897c9558f-cx4gt: app is still starting
        4 longload-5897c9558f-cx4gt: app is still starting
        5 longload-5897c9558f-cx4gt:Ok
        6 longload-5897c9558f-cx4gt:Ok
        7 longload-5897c9558f-cx4gt:Ok
        8 longload-5897c9558f-cx4gt:Ok...output omitted...
        ```
        
        Press **Ctrl**+**C** to quit the script.
        
2. When the application scales up, your customers complain that some requests fail. To replicate the issue, manually scale up the `longload` application to three replicas, and run the `~/DO180/labs/reliability-review/curl_loop.sh` script at the same time.
    
    The application takes seven seconds to initialize. The application exposes the `/health` API endpoint on HTTP port 3000. Configure the `longload` deployment to use this endpoint, to ensure that the application is ready before serving client requests.
    
    1. Open a new terminal window and run the `~/DO180/labs/reliability-review/curl_loop.sh` script.
        
        ```
        [student@workstation ~]$~/DO180/labs/reliability-review/curl_loop.sh
        1 longload-5897c9558f-cx4gt: Ok
        2 longload-5897c9558f-cx4gt: Ok
        3 longload-5897c9558f-cx4gt: Ok
        4 longload-5897c9558f-cx4gt: Ok
        ...output omitted...
        ```
        
        Leave the script running and do not interrupt it.
        
    2. Scale up the application to three replicas.
        
        ```
        [student@workstation ~]$oc scale deployment/longload --replicas 3
        deployment.apps/longload scaled
        ```
        
    3. Watch the output of the `curl_loop.sh` script in the second terminal. Some requests fail because OpenShift sends requests to the new pods before the application is ready.
        
        ```
        ...output omitted...
        22 longload-5897c9558f-cx4gt: Ok
        23 longload-5897c9558f-cx4gt: Ok
        24 longload-5897c9558f-cx4gt: Ok
        25 curl: (7) Failed to connect to master01.ocp4.example.com port 30372: Connection refused
        26 curl: (7) Failed to connect to master01.ocp4.example.com port 30372: Connection refused
        27 longload-5897c9558f-cx4gt: Ok
        28 curl: (7) Failed to connect to master01.ocp4.example.com port 30372: Connection refused
        29 longload-5897c9558f-cx4gt: Ok
        30 curl: (7) Failed to connect to master01.ocp4.example.com port 30372: Connection refused
        31 longload-5897c9558f-tpssf: app is still starting
        32 longload-5897c9558f-kkvm5: app is still starting
        33 longload-5897c9558f-cx4gt: Ok
        34 longload-5897c9558f-tpssf: app is still starting
        35 longload-5897c9558f-tpssf: app is still starting
        36 longload-5897c9558f-tpssf: app is still starting
        37 longload-5897c9558f-cx4gt: Ok
        38 longload-5897c9558f-tpssf: app is still starting
        39 longload-5897c9558f-cx4gt: Ok
        40 longload-5897c9558f-cx4gt: Ok
        ...output omitted...
        ```
        
        Leave the script running and do not interrupt it.
        
    4. Add a readiness probe to the `longload` deployment. Ignore the warning message.
        
        ```
        [student@workstation ~]$oc set probe deployment/longload --readiness \
          --initial-delay-seconds 7 \
          --get-url http://:3000/health
        deployment.apps/longload probes updated
        ```
        
    5. Scale down the application back to one pod.
        
        ```
        [student@workstation ~]$oc scale deployment/longload --replicas 1
        deployment.apps/longload scaled
        ```
        
        If scaling down breaks the `curl_loop.sh` script, then press **Ctrl**+**c** to stop the script in the second terminal. Then, restart the script.
        
    6. To test your work, scale up the application to three replicas again.
        
        ```
        [student@workstation ~]$oc scale deployment/longload --replicas 3
        deployment.apps/longload scaled
        ```
        
    7. Watch the output of the `curl_loop.sh` script in the second terminal. No request fails.
        
        ```
        ...output omitted...
        92 longload-7ddcc9b7fd-72dtm: Ok
        93 longload-7ddcc9b7fd-72dtm: Ok
        94 longload-7ddcc9b7fd-72dtm: Ok
        95 longload-7ddcc9b7fd-qln95: Ok
        96 longload-7ddcc9b7fd-wrxrb: Ok
        97 longload-7ddcc9b7fd-qln95: Ok
        98 longload-7ddcc9b7fd-wrxrb: Ok
        99 longload-7ddcc9b7fd-72dtm: Ok
        ...output omitted...
        ```
        
        Press **Ctrl**+**C** to quit the script.
        
3. Configure the application so that it automatically scales up when the average memory usage is above 60% of the memory requests value, and scales down when the usage is below this percentage. The minimum number of replicas must be one, and the maximum must be three. The resource that you create for scaling the application must be named `longload`.
    
    The `lab` command provides the `~/DO180/labs/reliability-review/hpa.yml` resource file as an example. Use the `oc explain` command to learn the valid parameters for the `hpa.spec.metrics.resource.target` attribute. Because the file is incomplete, you must update it first if you choose to use it.
    
    To test your work, use the `oc exec deploy/longload — curl localhost:3000/leak` command to sends an HTTP request to the application `/﻿leak` API endpoint. Each request consumes an additional 480 MiB of memory. To free this memory, you can use the `~/DO180/labs/reliability-review/free.sh` script
    
    1. Before you create the horizontal pod autoscaler resource, scale down the application to one pod.
        
        ```
        [student@workstation ~]$oc scale deployment/longload --replicas 1
        deployment.apps/longload scaled
        ```
        
    2. Edit the `~/DO180/labs/reliability-review/hpa.yml` resource file. You can retrieve the parameters for the `resource` attribute by using the `oc explain hpa.spec.metrics.resource` and `oc explain hpa.spec.metrics.resource.target` commands.
        
        ```
        apiVersion: autoscaling/v2
        kind: HorizontalPodAutoscaler
        metadata:
          name: longload
          labels:
            app: longload
        spec:
          maxReplicas:3
          minReplicas:1
          scaleTargetRef:
            apiVersion: apps/v1
            kind: Deployment
            name: longload
          metrics:
          - type: Resource
            resource:
        name: memory
              target:
                type: Utilization
                averageUtilization: 60
        ```
        
    3. Use the `oc apply` command to deploy the horizontal pod autoscaler.
        
        ```
        [student@workstation ~]$oc apply -f ~/DO180/labs/reliability-review/hpa.yml
        horizontalpodautoscaler.autoscaling/longload created
        ```
        
    4. In the second terminal, run the `watch` command to monitor the `oc get hpa longload` command. Wait for the `longload` horizontal pod autoscaler to report usage in the `TARGETS` column. The percentage on your system probably differs.
        
        ```
        [student@workstation ~]$watch oc get hpa longload
        Every 2.0s: oc get hpa longload            workstation: Fri Mar 10 05:15:34 2023
        
        NAME       REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
        longload   Deployment/longload13%/60%   1         3         1          75s
        ```
        
        Leave the command running and do not interrupt it.
        
    5. To test your work, run the `oc exec deploy/longload — curl localhost:3000/leak` command in the first terminal for the application to allocate 480 MiB of memory.
        
        ```
        [student@workstation ~]$oc exec deploy/longload -- curl -s localhost:3000/leak
        longload-7ddcc9b7fd-72dtm: consuming memory!
        ```
        
    6. In the second terminal, after two minutes, the `oc get hpa longload` command shows the memory increase. The horizontal pod autoscaler scales up the application to more than one replica. The percentage on your system probably differs.
        
        ```
        Every 2.0s: oc get hpa longload            workstation: Fri Mar 10 05:19:44 2023
        
        NAME       REFERENCE             TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
        longload   Deployment/longload145%/60%   1         32          5m18s
        ```
        
    7. To test your work, run the `~/DO180/labs/reliability-review/free.sh` script in the first terminal for the application to release the memory. Ensure that the pod that frees the memory is the same pod that was consuming memory. Execute the `free.sh` script several times if necessary.
        
        ```
        [student@workstation ~]$~/DO180/labs/reliability-review/free.sh
        longload-7ddcc9b7fd-72dtm: releasing memory!
        ```
        
    8. In the second terminal, after ten minutes, the `oc get hpa longload` command shows the memory decrease. The horizontal pod autoscaler scales down the application to one replica. The percentage on your system probably differs.
        
        ```
        Every 2.0s: oc get hpa longload            workstation: Fri Mar 10 05:19:44 2023
        
        NAME       REFERENCE             TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
        longload   Deployment/longload12%/60%   1         31          15m28s
        ```
        
        Press **Ctrl**+**C** to quit the `watch` command. Close that second terminal when done.