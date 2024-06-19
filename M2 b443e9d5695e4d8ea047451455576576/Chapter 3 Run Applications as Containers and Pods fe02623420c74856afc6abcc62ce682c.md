# Chapter 3.  Run Applications as Containers and Pods

# Guided Exercise: **Create Linux Containers and Kubernetes Pods**

## Login and Determine UID and GID for Project

Log in to the OpenShift cluster and create the `pods-containers` project. Determine the UID and GID ranges for pods in the `pods-containers` project.

1. Log in to the OpenShift cluster as the `developer` user with the `oc` command.
    
    ```
    [student@workstation ~]$oc login -u developer -p developer \
      https://api.ocp4.example.com:6443
    Login successful
    ...output omitted...
    ```
    
2. Create the `pods-containers` project.
    
    ```
    [student@workstation ~]$oc new-project pods-containers
    Now using project "pods-containers" on server "https://api.ocp4.example.com:6443".
    ...output omitted...
    ```
    
3. Identify the UID and GID ranges for pods in the `pods-containers` project.
    
    ```
    [student@workstation ~]$oc describe project pods-containers
    Name:			pods-containers
    Created:		28 seconds ago
    Labels:			kubernetes.io/metadata.name=pods-containers
    			pod-security.kubernetes.io/audit=restricted
    			pod-security.kubernetes.io/audit-version=v1.24
    			pod-security.kubernetes.io/warn=restricted
    			pod-security.kubernetes.io/warn-version=v1.24
    Annotations:		openshift.io/description=
    			openshift.io/display-name=
    			openshift.io/requester=developer
    			openshift.io/sa.scc.mcs=s0:c28,c22
    openshift.io/sa.scc.supplemental-groups=1000800000/10000openshift.io/sa.scc.uid-range=1000800000/10000
    Display Name:		<none>
    Description:		<none>
    Status:			Active
    Node Selector:		<none>
    Quota:			<none>
    Resource limits:	<none>
    ```
    
    Your UID and GID range values might differ from the previous output.
    

## Create Pod `ubi9-user`

As the `developer` user, create a pod called `ubi9-user` from a UBI9 base container image. The image is available in the `registry.ocp4.example.com:8443/ubi9/ubi` container registry. Set the restart policy to `Never` and start an interactive session. Configure the pod to execute the `whoami` and `id` commands to determine the UIDs, supplemental groups, and GIDs of the container user in the pod. Delete the pod afterward.

After the `ubi-user` pod is deleted, log in as the `admin` user and then re-create the `ubi9-user` pod. Retrieve the UIDs and GIDs of the container user. Compare the values to the values of the `ubi9-user` pod that the `developer` user created.

Afterward, delete the `ubi9-user` pod.

1. Use the `oc run` command to create the `ubi9-user` pod. Configure the pod to execute the `whoami` and `id` commands through an interactive bash shell session.
    
    ```
    [student@workstation ~]$oz \
      --image registry.ocp4.example.com:8443/ubi9/ubi \
      -- /bin/bash -c "whoami && id"
    1000800000
    uid=1000800000(1000800000) gid=0(root) groups=0(root),1000800000
    ```
    
    Your values might differ from the previous output.
    
    Notice that the user in the container has the same UID that is identified in the `pods-containers` project. However, the GID of the user in the container is `0`, which means that the user belongs to the `root` group. Any files and directories that the container processes might write to must have read and write permissions by `GID=0` and have the `root` group as the owner.
    
    Although the user in the container belongs to the `root` group, a UID value over `1000` means that the user is an unprivileged account. When a regular OpenShift user, such as the `developer` user, creates a pod, the containers within the pod run as unprivileged accounts.
    
2. Delete the pod.
    
    ```
    [student@workstation ~]$oc delete pod ubi9-user
    pod "ubi9-user" deleted
    ```
    
3. Log in as the `admin` user with the `redhatocp` password.
    
    ```
    [student@workstation ~]$oc login -u admin -p redhatocp
    Login successful.
    
    You have access to 71 projects, the list has been suppressed. You can list all projects with 'oc projects'
    
    Using project "pods-containers".
    ```
    
4. Re-create the `ubi9-user` pod as the `admin` user. Configure the pod to execute the `whoami` and `id` commands through an interactive bash shell session. Compare the values of the UID and GID for the container user to the values of the `ubi9-user` pod that the `developer` user created.
    
    <aside>
    <img src="https://www.notion.so/icons/light-bulb_blue.svg" alt="https://www.notion.so/icons/light-bulb_blue.svg" width="40px" /> It is safe to ignore pod security warnings for exercises in this course. OpenShift uses the Security Context Constraints controller to provide safe defaults for pod security.
    
    </aside>
    
    ```
    [student@workstation ~]$oc run -it ubi9-user --restart 'Never' \
      --image registry.ocp4.example.com:8443/ubi9/ubi \
      -- /bin/bash -c "whoami && id"
    Warning: would violate PodSecurity "restricted:v1.24": allowPrivilegeEscalation != false (container "ubi9-user" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "ubi9-user" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "ubi9-user" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "ubi9-user" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
    root
    uid=0(root) gid=0(root) groups=0(root)
    ```
    
    Notice that the value of the UID is `0`, which differs from the UID range value of the `pod-containers` project. The user in the container is the privileged account `root` user and belongs to the `root` group. When a cluster administrator creates a pod, the containers within the pod run as a privileged account by default.
    
5. Delete the `ubi9-user` pod.
    
    ```
    [student@workstation ~]$oc delete pod ubi9-user
    pod "ubi9-user" deleted
    ```
    

## Create Pod `ubi9-date`

As the `developer` user, use the `oc run` command to create a `ubi9-date` pod from a UBI9 base container image. The image is available in the `registry.ocp4.example.com:8443/ubi9/ubi` container registry. Set the restart policy to `Never`, and configure the pod to execute the `date` command. Retrieve the logs of the `ubi9-date` pod to confirm that the `date` command executed. Delete the pod afterward.

1. Log in as the `developer` user with the `developer` password.
    
    ```
    [student@workstation ~]$oc login -u developer -p developer
    Login successful.
    
    You have one project on this server: "pods-containers"
    
    Using project "pods-containers".
    ```
    
2. Create a pod called `ubi9-date` that executes the `date` command.
    
    ```
    [student@workstation ~]$oc run ubi9-date --restart 'Never' \
      --image registry.ocp4.example.com:8443/ubi9/ubi -- date
    pod/ubi9-date created
    ```
    
3. Wait a few moments for the creation of the pod. Then, retrieve the logs of the `ubi9-date` pod.
    
    ```
    [student@workstation ~]$oc logs ubi9-date
    Mon Nov 28 15:02:55 UTC 2022
    ```
    
4. Delete the `ubi9-date` pod.
    
    ```
    [student@workstation ~]$oc delete pod ubi9-date
    pod "ubi9-date" deleted
    ```
    

## Create Pod `ubi9-command`

Use the `oc run ubi9-command -it` command to create a `ubi9-command` pod with the  `registry.ocp4.example.com:8443/ubi9/ubi` container image. Add the `/bin/bash` in the `oc run` command to start an interactive shell. Exit the pods and view the logs for the `ubi9-command` pod with the `oc logs` command. Then, connect to the `ubi9-command` pod with the `oc attach` command, and issue the following command:

```
while true; do echo $(date); sleep 2; done
```

This command executes the `date` and `sleep` commands to generate output to the console every two seconds. Use the `oc logs` command to retrieve the logs of the `ubi9` pod, and confirm that the logs display the executed `date` and `sleep` commands.

1. Create a pod called `ubi9-command` and start an interactive shell.
    
    ```
    [student@workstation ~]$oc run ubi9-command -it \
      --image registry.ocp4.example.com:8443/ubi9/ubi -- /bin/bash
    If you don't see a command prompt, try pressing enter.
    bash-5.1$
    ```
    
2. Exit the shell session.
    
    ```
    bash-5.1$exit
    exit
    Session ended, resume using 'oc attach ubi9-command -c ubi9-command -i -t' command when the pod is running
    ```
    
3. Use the `oc logs` command to view the logs of the `ubi9-command` pod.
    
    ```
    [student@workstation ~]$oc logs ubi9-command
    bash-5.1$ [student@workstation ~]$
    ```
    
    The pod's command prompt is returned. The `oc logs` command displays the pod's current `stdout` and `stderr` output in the console. Because you disconnected from the interactive session, the pod's current `stdout` is the command prompt, and not the commands that you executed previously.
    
4. Use the `oc attach` command to connect to the `ubi9-command` pod again. In the shell, execute the `while true; do echo $(date); sleep 2; done` command to continuously generate `stdout` output.
    
    ```
    [student@workstation ~]$oc attach ubi9-command -it
    If you don't see a command prompt, try pressing enter.
    ```
    
    ```
    bash-5.1$while true; do echo $(date); sleep 2; done
    Mon Nov 28 15:15:16 UTC 2022
    Mon Nov 28 15:15:18 UTC 2022
    Mon Nov 28 15:15:20 UTC 2022
    Mon Nov 28 15:15:22 UTC 2022
    ...output omitted...
    ```
    
5. Open another terminal window and view the logs for the `ubi9-command` pod with the `oc logs` command. Limit the log output to the last 10 entries with the `-tail` option. Confirm that the logs display the results of the command that you executed in the container.
    
    ```
    [student@workstation ~]$oc logs ubi9-command --tail=10
    Mon Nov 28 15:15:16 UTC 2022
    Mon Nov 28 15:15:18 UTC 2022
    Mon Nov 28 15:15:20 UTC 2022
    Mon Nov 28 15:15:22 UTC 2022
    Mon Nov 28 15:15:24 UTC 2022
    Mon Nov 28 15:15:26 UTC 2022
    Mon Nov 28 15:15:28 UTC 2022
    Mon Nov 28 15:15:30 UTC 2022
    Mon Nov 28 15:15:32 UTC 2022
    Mon Nov 28 15:15:34 UTC 2022
    ```
    

## Identify thee PID of the `ubi9-command` Pod

Identify the name for the container in the `ubi9-command` pod. Identify the process ID (PID) for the container in the `ubi9-command` pod by using a debug pod for the pod's host node. Use the `crictl` command to identify the PID of the container in the `ubi9-command` pod. Then, retrieve the PID of the container in the debug pod.

1. Identify the container name in the `ubi9-command` pod with the `oc get` command. Specify the JSON format for the command output. Parse the JSON output with the `jq` command to retrieve the value of the `.status.containerStatuses[].name` object.
    
    ```
    [student@workstation ~]$oc get pod ubi9-command -o json | \
      jq .status.containerStatuses[].name
    "ubi9-command"
    ```
    
    The `ubi9-command` pod has a single container of the same name.
    
2. Find the host node for the `ubi9-command` pod. Start a debug pod for the host with the `oc debug` command.
    
    ```
    [student@workstation ~]$oc get pods ubi9-command -o wide
    NAME           READY STATUS  RESTARTS    AGE  IP         NODE     NOMINATED NODE READINESS GATES
    ubi9-command   1/1   Running 2 (16m ago) 27m  10.8.0.26  master01 <none>         <none>
    ```
    
    ```
    [student@workstation ~]$oc debug node/master01
    Error from server (Forbidden): nodes "master01" is forbidden: User "developer" cannot get resource "nodes" in API group "" at the cluster scope
    ```
    
    The debug pod fails because the `developer` user does not have the required permission to debug a host node.
    
3. Log in as the `admin` user with the `redhatocp` password. Start a debug pod for the host with the `oc debug` command. After connecting to the debug pod, run the `chroot /host` command to use host binaries, such as the `crictl` command-line tool.
    
    ```
    [student@workstation ~]$oc login -u admin -p redhatocp
    Login successful.
    ...output omitted...
    ```
    
    ```
    [student@workstation ~]$oc debug node/master01
    Starting pod/master01-debug ...
    To use host binaries, run `chroot /host`
    Pod IP: 192.168.50.10
    If you don't see a command prompt, try pressing enter
    ```
    
    ```
    sh-4.4#chroot /host
    ```
    
4. Use the `crictl ps` command to retrieve the `ubi9-command` container ID. Specify the `ubi9-command` container with the `-name` option and use the JSON output format. Parse the JSON output with the `jq -r` command to get the RAW JSON output. Export the container ID as the `$CID` environment variable.
    
    <aside>
    <img src="https://www.notion.so/icons/light-bulb_blue.svg" alt="https://www.notion.so/icons/light-bulb_blue.svg" width="40px" /> When using `jq` without the `-r` flag, the container ID is wrapped in double quotes, which does not work with `crictl` commands. If the `-r` flag is not used, then you can add `| tr -d '"'` to the end of the command to trim the double quotes.
    
    </aside>
    
    ```
    sh-5.1#crictl ps --name ubi9-command -o json | jq -r .containers[0].id
    81adbc6222d79ed9ba195af4e9d36309c18bb71bc04b2e8b5612be632220e0d6
    ```
    
    ```
    sh-5.1#CID=$(crictl ps --name ubi9-command -o json | jq -r .containers[0].id)
    ```
    
    ```
    sh-5.1#echo $CID
    81adbc6222d79ed9ba195af4e9d36309c18bb71bc04b2e8b5612be632220e0d6
    ```
    
    Your container ID value might differ from the previous output.
    
5. Use the `crictl inspect` command to find the PID of the `ubi9-command` container. The PID value is in the `.info.pid` object in the `crictl inspect` output. Export the `ubi9-command` container PID as the `$PID` environment variable.
    
    ```
    sh-5.1#crictl inspect $CID | grep pid
        "pid": 365297,
              "pids": {
                "type": "pid"
    ...output omitted...
              }
    ...output omitted...
    ```
    
    ```
    sh-5.1#PID=365297
    ```
    
    Your PID values might differ from the previous output.
    

## System Namespaces of the `ubi9-command` container

Use the `lsns` command to list the system namespaces of the `ubi9-command` container. Confirm that the running processes in the container are isolated to different system namespaces.

1. View the system namespaces of the `ubi9-command` container with the `lsns` command. Specify the PID with the `-p` option and use the `$PID` environment variable. In the resulting table, the `NS` column contains the namespace values for the container.
    
    ```
    sh-5.1#lsns -p $PID
            NS TYPE   NPROCS    PID USER       COMMAND
    4026531835 cgroup    540      1 root       /usr/lib/systemd/systemd --switched-root --system --deserialize 16
    4026531837 user      540      1 root       /usr/lib/systemd/systemd --switched-root --system --deserialize 16
    4026536117 uts         1 153168 1000800000 /bin/bash
    4026536118 ipc         1 153168 1000800000 /bin/bash
    4026536120 net         1 153168 1000800000 /bin/bash
    4026537680 mnt         1 153168 1000800000 /bin/bash
    4026537823 pid         1 153168 1000800000 /bin/bash
    ```
    
    Your namespace values might differ from the previous output.
    

## OS `ubi9-command`

Use the host debug pod to retrieve and compare the operating system (OS) and the GNU C Library (`glibc`) package version of the `ubi9-command` container and the host node.

1. Retrieve the OS for the host node with the `cat /etc/redhat-release` command.
    
    ```
    sh-5.1#cat /etc/redhat-release
    Red Hat Enterprise Linux CoreOS release 4.14
    ```
    
2. Use the `crictl exec` command and the `$CID` container ID variable to retrieve the OS of the `ubi9-command` container. Use the `it` options to create an interactive terminal to execute the `cat /etc/redhat-release` command.
    
    ```
    sh-5.1#crictl exec -it $CID cat /etc/redhat-release
    Red Hat Enterprise Linux release 9.1 (Plow)
    ```
    
    The `ubi9-command` container has a different OS from the host node.
    
3. Use the `ldd --version` command to retrieve the `glibc` package version of the host node.
    
    ```
    sh-5.1$ldd --version
    ldd (GNU libc) 2.28
    Copyright (C) 2018 Free Software Foundation, Inc.
    ...output omitted...
    ```
    
4. Use the `crictl exec` command and the `$CID` container ID variable to retrieve the `glibc` package version of the `ubi9-command` container. Use the `it` options to create an interactive terminal to execute the `ldd --version` command.
    
    ```
    sh-5.1#crictl exec -it $CID ldd --version
    ldd (GNU libc) 2.34
    Copyright (C) 2021 Free Software Foundation, Inc.
    ...output omitted...
    ```
    
    The `ubi9-command` container has a different version of the `glibc` package from its host.
    

## Exit

Exit the `master01-debug` pod and the `ubi9-command` pod.

1. Exit the `master01-debug` pod. You must issue the `exit` command to end the host binary access. Execute the `exit` command again to exit and remove the `master01-debug` pod.
    
    ```
    sh-5.1#exit
    exit
    ```
    
    ```
    sh-4.4#exit
    exit
    
    Removing debug pod ...
    Temporary namespace openshift-debug-bg7kn was removed.
    ```
    
2. Return to the terminal window that is connected to the `ubi9-command` pod. Press **Ctrl**+**C** and then execute the `exit` command. Confirm that the pod is still running.
    
    ```
    ...output omitted...^C
    bash-5.1$exit
    exit
    Session ended, resume using 'oc attach ubi9-command -c ubi9-command -i -t' command when the pod is running
    ```
    
    ```
    [student@workstation ~]$oc get pods
    NAME           READY   STATUS    RESTARTS     AGE
    ubi9-command   1/1     Running   2 (6s ago)   35m
    ```
    

# **Guided Exercise: Find and Inspect Container Images**

## Login and Authenticate

1. Log in to the OpenShift cluster and create the `pods-images` project.
    1. Log in to the OpenShift cluster as the `developer` user with the `oc` command.
        
        ```
        [student@workstation ~]$oc login -u developer -p developer \
          https://api.ocp4.example.com:6443...output omitted...
        ```
        
    2. Create the `pods-images` project.
        
        ```
        [student@workstation ~]$oc new-project pods-images...output omitted...
        ```
        
2. Authenticate to `registry.ocp4.example.com:8443`, which is the classroom container registry. This private registry hosts certain copies and tags of community images from Docker and Bitnami, as well as some supported images from Red Hat. Use `skopeo` to log in as the `developer` user, and then retrieve a list of available tags for the `registry.ocp4.example.com:8443/redhattraining/docker-nginx` container repository.
    1. Use the `skopeo login` command to log in as the `developer` user with the `developer` password.
        
        ```
        [student@workstation ~]$skopeo login registry.ocp4.example.com:8443
        Username:developer
        Password:developer
        Login Succeeded!
        ```
        
    2. The classroom registry contains a copy and specific tags of the `docker.io/library/nginx` container repository. Use the `skopeo list-tags` command to retrieve a list of available tags for the `registry.ocp4.example.com:8443/redhattraining/docker-nginx` container repository.
        
        ```
        [student@workstation ~]$skopeo list-tags \
          docker://registry.ocp4.example.com:8443/redhattraining/docker-nginx
        {
            "Repository": "registry.ocp4.example.com:8443/redhattraining/docker-nginx",
            "Tags": [
                "1.23",
                "1.23-alpine",
                "1.23-perl",
                "1.23-alpine-perl"
                "latest"
            ]
        }
        ```
        

## Create a `docker-nginx` Pod

1. Use the `oc run` command to create the `docker-nginx` pod.
    
    ```
    [student@workstation ~]$oc run docker-nginx \
      --image registry.ocp4.example.com:8443/redhattraining/docker-nginx:1.23
    pod/docker-nginx created
    ```
    
2. After a few moments, verify the status of the `docker-nginx` pod.
    
    ```
    [student@workstation ~]$oc get pods
    NAME           READY   STATUS   RESTARTS   AGE
    docker-nginx   0/1     Error    0          4s
    ```
    
    ```
    [student@workstation ~]$oc get pods
    NAME           READY   STATUS             RESTARTS      AGE
    docker-nginx   0/1     CrashLoopBackOff   2 (17s ago)   38s
    ```
    
    The `docker-nginx` pod failed to start.
    
3. Investigate the pod failure. Retrieve the logs of the `docker-nginx` pod to identify a possible cause of the pod failure.
    
    ```
    [student@workstation ~]$oc logs docker-nginx...output omitted...
    /docker-entrypoint.sh: Configuration complete; ready for start up
    2022/12/02 18:51:45 [warn] 1#1: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
    nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
    2022/12/02 18:51:45 [emerg] 1#1:**mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
    nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)**
    ```
    
    The pod failed to start because of permission issues for the `nginx` directories.
    
4. Create a debug pod for the `docker-nginx` pod.
    
    ```
    [student@workstation ~]$oc debug pod/docker-nginx
    Starting pod/docker-nginx-debug ...
    Pod IP: 10.8.0.72
    If you don't see a command prompt, try pressing enter.
    
    $
    ```
    
5. From the debug pod, verify the permissions of the `/etc/nginx` and `/var/cache/nginx` directories.
    
    ```
    $ls -la /etc/ | grep nginx
    drwxr-xr-x. 3 root root     132 Nov 15 13:14 nginx
    ```
    
    ```
    $ls -la /var/cache | grep nginx
    drwxr-xr-x. 2 root root   6 Oct 19 09:32 nginx
    ```
    
    Only the `root` user has permission to the `nginx` directories. The pod must therefore run as the privileged `root` user to work.
    
6. Retrieve the user ID (UID) of the `docker-nginx` user to determine whether the user is a privileged or unprivileged account. Then, exit the debug pod.
    
    ```
    $whoami
    1000820000
    ```
    
    ```
    $exit
    
    Removing debug pod ...
    ```
    
    Your UID value might differ from the previous output.
    
    A UID over `0` means that the container's user is a `non-root` account. Recall that OpenShift default security policies prevent regular user accounts, such as the `developer` user, from running pods and their containers as privileged accounts.
    
7. Confirm that the `docker-nginx:1.23` image requires the `root` privileged account. Use the `skopeo inspect --config` command to view the configuration for the image.
    
    ```
    [student@workstation ~]$skopeo inspect --config \
      docker://registry.ocp4.example.com:8443/redhattraining/docker-nginx:1.23...output omitted...
        "config": {
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.23.3",
                "NJS_VERSION=0.7.9",
                "PKG_RELEASE=1~bullseye"
            ],
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Labels": {
                "maintainer": "NGINX Docker Maintainers \u003cdocker-maint@nginx.com\u003e"
            },
            "StopSignal": "SIGQUIT"
        },
    ...output omitted...
    ```
    
    The image configuration does not define `USER` metadata, which confirms that the image must run as the `root` privileged user.
    
8. The `docker-nginx:1-23` container image must run as the `root` privileged user. OpenShift security policies prevent regular cluster users, such as the `developer` user, from running containers as the `root` user. Delete the `docker-nginix` pod.
    
    ```
    [student@workstation ~]$oc delete pod docker-nginx
    pod "docker-nginx" deleted
    ```
    

## Create a `bitnami-mysql` Pod

Create a `bitnami-mysql` pod, which uses a copy of the Bitnami community MySQL image. The image is available in the  `registry.ocp4.example.com:8443/redhattraining/bitnami-mysql` container repository.

1. A copy and specific tags of the `docker.io/bitnami/mysql` container repository are hosted in the classroom registry. Use the `skopeo list-tags` command to identify available tags for the Bitnami MySQL community image in the  `registry.ocp4.example.com:8443/redhattraining/bitnami-mysql` container repository.
    
    ```
    [student@workstation ~]$skopeo list-tags \
      docker://registry.ocp4.example.com:8443/redhattraining/bitnami-mysql
    {
        "Repository": "registry.ocp4.example.com:8443/redhattraining/bitnami-mysql",
        "Tags": [
            "8.0.31",
            "8.0.30",
            "8.0.29",
            "8.0.28",
            "latest"
        ]
    }
    ```
    
2. Retrieve the configuration of the `bitnami-mysql:8.0.31` container image. Determine whether the image requires a privileged account by inspecting image configuration for `USER` metadata.
    
    ```
    [student@workstation ~]$skopeo inspect --config \
      docker://registry.ocp4.example.com:8443/redhattraining/bitnami-mysql:8.0.31...output omitted...
        "config":
            "User":"**1001**",
            "ExposedPorts": {
                "3306/tcp": {}
            },
    ....output omitted...
    ```
    
    The image defines the `1001` UID, which means that the image does not require a privileged account.
    
3. Create the `bitnami-mysql` pod with the `oc run` command. Use the `registry.ocp4.example.com:8443/redhattraining/bitnami-mysql:8.0.31` container image. Then, wait a few moments and then retrieve the pod's status with the `oc get` command.
    
    ```
    [student@workstation ~]$oc run bitnami-mysql \
      --image registry.ocp4.example.com:8443/redhattraining/bitnami-mysql:8.0.31
    pod/bitnami-mysql created
    ```
    
    ```
    [student@workstation ~]$oc get pods
    NAME            READY   STATUS               RESTARTS      AGE
    bitnami-mysql   0/1     CrashLoopBackoff     2 (19s ago)   23s
    ```
    
    The pod failed to start.
    
4. Examine the logs of the `bitnami-mysql` pod to determine the cause of the failure.
    
    ```
    [student@workstation ~]$oc logs bitnami-mysql
    mysql 16:18:00.40
    mysql 16:18:00.40 Welcome to the Bitnami mysql container
    mysql 16:18:00.40 Subscribe to project updates by watching https://github.com/bitnami/containers
    mysql 16:18:00.40 Submit issues and feature requests at https://github.com/bitnami/containers/issues
    mysql 16:18:00.40
    mysql 16:18:00.41 INFO  ==> ** Starting MySQL setup **
    mysql 16:18:00.42 INFO  ==> Validating settings in MYSQL_*/MARIADB_* env vars
    mysql 16:18:00.42 ERROR ==>**The MYSQL_ROOT_PASSWORD environment variable is empty or not set**. Set the environment variable ALLOW_EMPTY_PASSWORD=yes to allow the container to be started with blank passwords. This is recommended only for development.
    ```
    
    The `MYSQL_ROOT_PASSWORD` environment variable must be set for the pod to start.
    
5. Delete and then re-create the `bitnami-mysql` pod. Specify `redhat123` as the value for the `MYSQL_ROOT_PASSWORD` environment variable. After a few moments, verify the status of the pod.
    
    ```
    [student@workstation ~]$oc delete pod bitnami-mysql
    pod "bitnami-mysql" deleted
    ```
    
    ```
    [student@workstation ~]$oc run bitnami-mysql \
      --image registry.ocp4.example.com:8443/redhattraining/bitnami-mysql:8.0.31 \
      --env MYSQL_ROOT_PASSWORD=redhat123
    pod/bitnami-mysql created
    ```
    
    ```
    [student@workstation ~]$oc get pods
    NAME            READY   STATUS    RESTARTS   AGE
    bitnami-mysql   1/1     Running   0          20s
    ```
    
    The `bitnami-mysql` pod successfully started.
    
6. Determine the UID of the container user in the `bitnami-mysql` pod. Compare this value to the UID in the container image and to the UID range of the `pods-images` project.
    
    ```
    [student@workstation ~]$oc exec -it bitnami-mysql -- /bin/bash -c "whoami && id"
    1000820000
    uid=1000820000(1000820000) gid=0(root) groups=0(root),1000820000
    ```
    
    ```
    [student@workstation ~]$oc describe project pods-images
    Name:			pods-images
    ...output omitted...
    Annotations:	openshift.io/description=
    ...output omitted...
                    openshift.io/sa.scc.supplemental-groups=1000820000/10000
    **openshift.io/sa.scc.uid-range=1000820000/10000**...output omitted...
    ```
    
    Your values for the UID of the container and the UID range of the project might differ from the previous output.
    
    The container user UID is the same as the specified UID range in the namespace. Notice that the container user UID does not match the `1001` UID of the container image. For a container to use the specified UID of a container image, the pod must be created with a privileged OpenShift user account, such as the `admin` user.
    

## MySQL Image from Red Hat

The private classroom registry hosts a copy of a supported MySQL image from Red Hat. Retrieve the list of available tags for the `registry.ocp4.example.com:8443/rhel9/mysql-80` container repository. Compare the `rhel9/mysql-80` container image release version that is associated with each tag.

1. Use the `skopeo list-tags` command to list the available tags for the `rhel9/mysql-80` container image.
    
    ```
    [student@workstation ~]$skopeo list-tags \
      docker://registry.ocp4.example.com:8443/rhel9/mysql-80
    {
        "Repository": "registry.ocp4.example.com:8443/rhel9/mysql-80",
        "Tags": [
            "1-237",
            "1-228",
            "1-228-source",
            "1-224",
            "1-224-source",
    	"latest",
            "1"
        ]
    }
    ```
    
    Several tags are available:
    
    - The `latest` and `1` tags are floating tags, which are aliases to other tags, such as the `1-237` tag.
    - The `1-228` and `1-224` tags are fixed tags, which point to a build of a container.
    - The `1-228-source` and `1-224-source` tags are source containers, which provide the necessary sources and license terms to rebuild and distribute the images.
2. Use the `skopeo inspect` command to compare the `rhel9/mysql-80` container image release version and SHA IDs that are associated with the identified tags.
    
    <aside>
    <img src="https://www.notion.so/icons/light-bulb_blue.svg" alt="https://www.notion.so/icons/light-bulb_blue.svg" width="40px" /> To improve readability, the instructions truncate the SHA-256 strings.
    
    </aside>
    
    On your system, the commands return the full SHA-256 strings.
    
    ```
    [student@workstation ~]$skopeo inspect \
      docker://registry.ocp4.example.com:8443/rhel9/mysql-80:latest...output omitted...
        "Name": "registry.ocp4.example.com:8443/rhel9/mysql-80",
        "Digest":"**sha256:d282...f38f**",
    ...output omitted...
        "Labels":
    ...output omitted...
            "name": "rhel9/mysql-80",
            "release":"**237**",
    ...output omitted...
    ```
    
    You can also format the output of the `skopeo inspect` command with a Go template. Append the template objects with `\n` to add new lines between the results.
    
    ```
    [student@workstation ~]$skopeo inspect --format \
      "Name: {{.Name}}\n Digest: {{.Digest}}\n Release: {{.Labels.release}}" \
      docker://registry.ocp4.example.com:8443/rhel9/mysql-80:latest
    Name: registry.ocp4.example.com:8443/rhel9/mysql-80
     Digest:**sha256:d282...f38f**
     Release:**237**
    ```
    
    ```
    [student@workstation ~]$skopeo inspect --format \
      "Name: {{.Name}}\n Digest: {{.Digest}}\n Release: {{.Labels.release}}" \
      docker://registry.ocp4.example.com:8443/rhel9/mysql-80:1
    Name: registry.ocp4.example.com:8443/rhel9/mysql-80
     Digest:**sha256:d282...f38f**
     Release:**237**
    ```
    
    ```
    [student@workstation ~]$skopeo inspect --format \
      "Name: {{.Name}}\n Digest: {{.Digest}}\n Release: {{.Labels.release}}" \
      docker://registry.ocp4.example.com:8443/rhel9/mysql-80:1-237
    Name: registry.ocp4.example.com:8443/rhel9/mysql-80
     Digest:**sha256:d282...f38f**
     Release:**237**
    ```
    
    The `latest`, `1`, and `1-237` tags resolve to the same release versions and SHA IDs. The `latest` and `1` tags are floating tags for the `1-237` fixed tag.
    

## `mysql-80:1-228` Container Image.

### Run the Pod

The classroom registry hosts a copy and certain tags of the `registry.redhat.io/rhel9/mysql-80` container repository. Use the  `oc run` command to create a `rhel9-mysql` pod from the  `registry.ocp4.example.com:8443/rhel9/mysql-80:1-228` container image. Verify the status of the pod and then inspect the container logs for any errors.

1. Create a `rhel9-mysql` pod with the  `registry.ocp4.example.com:8443/rhel9/mysql-80:1-237` container image.
    
    ```
    [student@workstation ~]$oc run rhel9-mysql \
      --image registry.ocp4.example.com:8443/rhel9/mysql-80:1-237
    pod/rhel9-mysql created
    ```
    
2. After a few moments, retrieve the pod's status with the `oc get` command.
    
    ```
    [student@workstation ~]$oc get pods
    NAME          READY   STATUS              RESTARTS      AGE
    bitnami-mysql 1/1     Running             0             5m16s
    rhel9-mysql   0/1     CrashLoopBackoff    2 (29s ago)   49s
    ```
    
    The pod failed to start.
    
3. Retrieve the logs for the `rhel9-mysql` pod to determine why the pod failed.
    
    ```
    [student@workstation ~]$oc logs rhel9-mysql
    => sourcing 20-validate-variables.sh ...
    **You must either specify the following environment variables:**
      MYSQL_USER (regex: '^[a-zA-Z0-9_]+$')
      MYSQL_PASSWORD (regex: '^[a-zA-Z0-9_~!@#$%^&*()-=<>,.?;:|]+$')
      MYSQL_DATABASE (regex: '^[a-zA-Z0-9_]+$')
    Or the following environment variable:
      MYSQL_ROOT_PASSWORD (regex: '^[a-zA-Z0-9_~!@\#$%^&*()-=<>,.?;:|]+$')
    Or both.
    Optional Settings:
      MYSQL_LOWER_CASE_TABLE_NAMES (default: 0)
    ...output omitted...
    ```
    
    The pod failed because the required environment variables were not set for the container.
    

### Solve Promblems

Delete the `rhel9-mysql` pod. Create another `rhel9-mysql` pod and specify the necessary environment variables. Retrieve the status of the pod and inspect the container logs to confirm that the new pod is working.

1. Delete the `rhel9-mysql` pod with the `oc delete` command. Wait for the pod to delete before continuing to the next step.
    
    ```
    [student@workstation ~]$oc delete pod rhel9-mysql
    pod "rhel9-mysql" deleted
    ```
    
2. Create another `rhel9-mysql` pod from the `registry.ocp4.example.com:8443/rhel9/mysql-80:1-237` container image. Use the `oc run` command with the `-env` option to specify the following environment variables and their values:
    
    
    | Variable | Value |
    | --- | --- |
    | MYSQL_USER | redhat |
    | MYSQL_PASSWORD | redhat123 |
    | MYSQL_DATABASE | worldx |
    
    ```
    [student@workstation ~]$oc run rhel9-mysql \
      --image registry.ocp4.example.com:8443/rhel9/mysql-80:1-237 \
      --env MYSQL_USER=redhat \
      --env MYSQL_PASSWORD=redhat123 \
      --env MYSQL_DATABASE=worldx
    pod/rhel9-mysql created
    ```
    
3. After a few moments, retrieve the status of the `rhel9-mysql` pod with the `oc get` command. View the container logs to confirm that the database on the `rhel9-mysql` pod is ready to accept connections.
    
    ```
    [student@workstation ~]$oc get pods
    NAME          READY   STATUS    RESTARTS   AGE
    bitnami-mysql 1/1     Running   0          10m
    rhel9-mysql   1/1     Running   0          20s
    ```
    
    ```
    [student@workstation ~]$oc logs rhel9-mysql...output omitted...
    2022-11-02T20:14:14.333599Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/lib/mysql/mysqlx.sock
    2022-11-02T20:14:14.333641Z 0 [System] [MY-010931] [Server] /usr/libexec/mysqld:**ready for connections**. Version: '8.0.30'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  Source distribution.
    ```
    
    The `rhel9-mysql` pod is ready to accept connections.
    

### Location of the MySQL Databes Files

Determine the location of the MySQL database files for the `rhel9-mysql` pod. Confirm that the directory contains the `worldx` database.

1. Use the `oc image` command to inspect the `rhel9/mysql-80:1-228` image in the `registry.ocp4.example.com:8443` classroom registry.
    
    ```
    [student@workstation ~]$oc image info \
      registry.ocp4.example.com:8443/rhel9/mysql-80:1-237
    Name:          registry.ocp4.example.com:8443/rhel9/mysql-80:1-237
    ...output omitted...
    Command:       run-mysqld
    Working Dir:   /opt/app-root/src
    User:          27
    Exposes Ports: 3306/tcp
    Environment:   container=oci
                   STI_SCRIPTS_URL=image:///usr/libexec/s2i
                   STI_SCRIPTS_PATH=/usr/libexec/s2i
                   APP_ROOT=/opt/app-root
                   PATH=/opt/app-root/src/bin:/opt/app-root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
                   PLATFORM=el9
                   MYSQL_VERSION=8.0
                   APP_DATA=/opt/app-root/src
                   HOME=**/var/lib/mysql**
    ```
    
    The container manifest sets the `HOME` environment variable for the container user to the `/var/lib/mysql` directory.
    
2. Use the `oc exec` command to list the contents of the `/var/lib/mysql` directory.
    
    ```
    [student@workstation ~]$oc exec -it rhel9-mysql -- ls -la /var/lib/mysql
    total 12
    drwxrwxr-x. 1 mysql root          102 Nov  2 20:41 .
    drwxr-xr-x. 1 root  root          19 Oct 24 18:47 ..
    drwxrwxr-x. 1 mysql root          4096 Nov  2 20:54 **data**
    srwxrwxrwx. 1 mysql 1000820000    0 Nov  2 20:41 mysql.sock
    -rw-------. 1 mysql 1000820000    2 Nov  2 20:41 mysql.sock.lock
    srwxrwxrwx. 1 mysql 1000820000    0 Nov  2 20:41 mysqlx.sock
    -rw-------. 1 mysql 1000820000    2 Nov  2 20:41 mysqlx.sock.lock
    ```
    
    A `data` directory exists in the `/var/lib/mysql` directory.
    
3. Use the `oc exec` command again to list the contents of the `/var/lib/mysql/data` directory.
    
    ```
    [student@workstation ~]$oc exec -it rhel9-mysql \
      -- ls -la /var/lib/mysql/data | grep worldx
    drwxr-x---. 2 1000820000 root     6 Nov  2 20:41 **worldx**
    ```
    
    The `/var/lib/mysql/data` directory contains the `worldx` database with the `worldx` directory.
    

### Determine the IP Address

Determine the IP address of the `rhel9-mysql` pod. Next, create another MySQL pod, named `mysqlclient`, to access the `rhel9-mysql` pod. Confirm that the `mysqlclient` pod can view the available databases on the `rhel9-mysql` pod with the `mysqlshow` command.

1. Identify the IP address of the `rhel9-mysql` pod.
    
    ```
    [student@workstation ~]$oc get pods rhel9-mysql -o json | jq .status.podIP
    "10.8.0.109"
    ```
    
    Note the IP address. Your IP address might differ from the previous output.
    
2. Use the `oc run` command to create a pod named `mysqlclient` that uses the `registry.ocp4.example.com:8443/rhel9/mysql-80:1-237` container image. Set the value of the `MYSQL_ROOT_PASSWORD` environment variable to `redhat123`, and then confirm that the pod is running.
    
    ```
    [student@workstation ~]$oc run mysqlclient \
      --image registry.ocp4.example.com:8443/rhel9/mysql-80:1-237 \
      --env MYSQL_ROOT_PASSWORD=redhat123
    pod/mysqlclient created
    ```
    
    ```
    [student@workstation ~]$oc get pods
    NAME          READY   STATUS    RESTARTS   AGE
    bitnami-mysql 1/1     Running   0          15m
    mysqlclient   1/1     Running   0          19s
    rhel9-mysql   1/1     Running   0          5m
    ```
    
3. Use the `oc exec` command with the `it` options to execute the `mysqlshow` command on the `mysqlclient` pod. Connect as the `redhat` user and specify the host as the IP address of the `rhel9-mysql` pod. When prompted, enter `redhat123` for the password.
    
    ```
    [student@workstation ~]$oc exec -it mysqlclient \
      -- mysqlshow -u redhat -p -h10.8.0.109
    Enter password:redhat123
    +--------------------+
    |     Databases      |
    +--------------------+
    | information_schema |
    | performance_schema |
    | worldx             |
    +--------------------+
    ```
    
    The `worldx` database on the `rhel9-mysql` pod is accessible to the `mysql-client` pod.
    

# **Guided Exercise: Troubleshoot Containers and Pods**

## Login

Log in to the OpenShift cluster and create the `pods-troubleshooting` project.

1. Log in to the OpenShift cluster as the `developer` user with the `oc` command.
    
    ```
    [student@workstation ~]$oc login -u developer -p developer \
    https://api.ocp4.example.com:6443...output omitted...
    ```
    
2. Create the `pods-troubleshooting` project.
    
    ```
    [student@workstation ~]$oc new-project pods-troubleshooting...output omitted...
    ```
    

## Create MySQL Pod

Create a MySQL pod called `mysql-server` with the `oc run` command. Use the `registry.ocp4.example.com:8443/rhel9/mysql-80:1228` container image for the pod. Specify the environment variables with the following values:

| Variable | Value |
| --- | --- |
| MYSQL_USER | redhat |
| MYSQL_PASSWORD | redhat123 |
| MYSQL_DATABASE | world |

Then, view the status of the pod with the `oc get` command.

1. Create the `mysql-server` pod with the `oc run` command. Specify the environment values with the `-env` option.
    
    ```
    [student@workstation ~]$oc run mysql-server \
      --image registry.ocp4.example.com:8443/rhel9/mysql-80:1228 \
      --env MYSQL_USER=redhat \
      --env MYSQL_PASSWORD=redhat123 \
      --env MYSQL_DATABASE=world
    pod/mysql created
    ```
    
2. After a few moments, retrieve the status of the pod. Execute the `oc get pods` command a few times to view the status updates for the pod.
    
    ```
    [student@workstation ~]$oc get pods
    NAME           READY   STATUS             RESTARTS   AGE
    mysql-server   0/1     ErrImagePull       0          30s
    ```
    
    ```
    [studet@workstation ~]$oc get pods
    NAME           READY   STATUS             RESTARTS   AGE
    mysql-server   0/1     ImagePullBackoff   0          45s
    ```
    
    The pod failed to start.
    
3. Retrieve the pod's logs with the `oc logs` command.
    
    ```
    [student@workstation ~]$oc logs mysql-server
    Error from server (BadRequest): container "mysql-server" in pod "mysql-server" is waiting to start: trying and failing to pull image
    ```
    
    The logs state that the pod cannot pull the container image.
    
4. Retrieve the events log with the `oc get events` command.
    
    ```
    student@workstation ~]$oc get events
    LAST SEEN   TYPE      REASON           OBJECT             MESSAGE
    33s         Normal    Scheduled        pod/mysql-server   Successfully assigned pods-troubleshooting/mysql-server to master01 by master01
    31s         Normal    AddedInterface   pod/mysql-server   Add eth0 [10.8.0.68/23] from ovn-kubernetes
    16s         Normal    Pulling          pod/mysql-server   Pulling image "registry.ocp4.example.com:8443/rhel9/mysql-80:1228"
    16s         Warning   Failed           pod/mysql-server  **Failed to pull image "registry.ocp4.example.com:8443/rhel9/mysql-80:1228": rpc error: code = Unknown desc = reading manifest 1228 in registry.ocp4.example.com:8443/rhel9/mysql-80: manifest unknown: manifest unknown**
    16s         Warning   Failed           pod/mysql-server   Error: ErrImagePull
    4s          Normal    BackOff          pod/mysql-server   Back-off pulling image "registry.ocp4.example.com:8443/rhel9/mysql-80:1228"
    4s          Warning   Failed           pod/mysql-server   Error: ImagePullBackOff
    ```
    
    The output states that the image pull failed because the `1228` manifest is unknown. This failure could mean that the manifest, or image tag, does not exist in the repository.
    
5. Inspect the available manifest in the  `registry.ocp4.example.com:8443/rhel9/mysql-80` container repository. Authenticate to the container repository with the `skopeo login` command as the `developer` user with the `developer` password. Then, use the `skopeo inspect` command to retrieve the available manifests in the `registry.ocp4.example.com:8443/rhel9/mysql-80` repository.
    
    ```
    [student@workstation ~]$skopeo login registry.ocp4.example.com:8443
    Username:**developer**
    Password:**developer**
    Login Succeeded!
    ```
    
    ```
    [student@workstation ~]$skopeo inspect \
      docker://registry.ocp4.example.com:8443/rhel9/mysql-80...output omitted...
        "Name": "registry.ocp4.example.com:8443/rhel9/mysql-80",
        "Digest": "sha256:d282...f38f",
        "RepoTags": [
            "1-237",
            "1-228",
            "1-228-source",
            "1-224",
            "1-224-source",
            "latest",
            "1"
        ],
    ...output omitted...
    ```
    
    The `1228` manifest, or tag, is not available in the repository, which means that the `registry.ocp4.example.com:8443/rhel9/mysql-80:1228` image does not exist. However, the `1-228` tag does exist.
    
6. The pod failed to start because of a typing error in the image tag. Update the pod's configuration to use the `registry.ocp4.example.com:8443/rhel9/mysql-80:1-228` container image. Confirm that the pod is re-created after editing the resource.
    1. Edit the pod's configuration with the `oc edit` command. Locate the `.spec.containers.image` object. Update the value to the `registry.ocp4.example.com:8443/rhel9/mysql-80:1-228` container image and save the change.
        
        ```
        [student@workstation ~]$oc edit pod/mysql-server
        ```
        
        ```
        ...output omitted...
        apiVersion: v1
        kind: Pod
        metadata:
        ...output omitted...
        spec:
          containers:
          - image: registry.ocp4.example.com:8443/rhel9/**mysql-80:1-228**...output omitted...
        ```
        
    2. Verify the status of the `mysql-server` pod with the `oc get` command. The pod's status might take a few moments to update after the resource edit. Repeat the `oc get` command until the pod's status changes.
        
        ```
        [student@workstation ~]$oc get pods
        NAME           READY   STATUS     RESTARTS      AGE
        mysql-server   1/1     Running    0             10m
        ```
        
        The `mysql-server` pod successfully pulled the image and created the container. The pod now shows a `Running` status.
        

## Prepare Database

Prepare the database on the `mysql-server` pod. Copy the `~/DO180/labs/pods-troubleshooting/world_x.sql` file to the `mysql-server` pod. Then, connect to the pod and execute the `world_x.sql` file to populate the `world` database.

1. Use the `oc cp` command to copy the `world_x.sql` file in the `~/DO180/labs/pods-troubleshooting` directory to the `/tmp/` directory on the `mysql-server` pod.
    
    ```
    [student@workstation ~]$oc cp ~/DO180/labs/pods-troubleshooting/world_x.sql \
      mysql-server:/tmp/
    ```
    
2. Confirm that the `world_x.sql` file is accessible within the `mysql-server` pod with the `oc exec` command.
    
    ```
    [student@workstation ~]$oc exec -it pod/mysql-server -- ls -la /tmp/
    total 0
    drwxrwxrwx. 1 root       root  22     Nov  4 14:45 .
    dr-xr-xr-x. 1 root       root  50     Nov  4 14:34 ..
    -rw-rw-r--. 1 1000680000 root  558791 Nov  4 14:45 world_x.sql
    ```
    
3. Connect to the `mysql-server` pod with the `oc rsh` command. Then, log in to MySQL as the `redhat` user with the `redhat123` password.
    
    ```
    [student@workstation ~]$**oc rsh mysql-server**
    ```
    
    ```
    sh-5.1$**mysql -u redhat -p**
    Enter password:**redhat123**
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    ...output omitted...
    mysql>
    ```
    
4. From the MySQL prompt, select the `world` database. Source the `world_x.sql` script inside the pod to initialize and populate the `world` database.
    
    ```
    mysql>**USE world**;
    Database changed
    ```
    
    ```
    mysql>**SOURCE /tmp/world_x.sql**;
    ...output omitted...
    ```
    
5. Execute the `SHOW TABLES;` command to confirm that the database now contains tables. Then, exit the database and the pod.
    
    ```
    mysql>SHOW TABLES;
    -----------------
    | Tables_in_test  |
    -----------------
    | city            |
    | country         |
    | countryinfo     |
    | countrylanguage |
    -----------------
    4 rows in set (0.00 sec)
    ```
    
    ```
    mysql>**exit**;
    Bye
    sh-5.1$**exit**
    exit
    [student@workstation ~]$
    ```
    

## Connect `workstation` with  `world` database

Configure port forwarding and then use the MySQL client on the `workstation` machine to connect to the `world` database on the `mysql-server` pod. Confirm that you can access data within the `world` database from the `workstation` machine.

1. From the `workstation` machine, use the `oc port-forward` command to forward the `3306` local port to the `3306` port on the `mysql-server` pod.
    
    ```
    [student@workstation ~]$oc port-forward mysql-server 3306:3306
    Forwarding from 127.0.0.1:3306 -> 3306
    Forwarding from [::1]:3306 -> 3306
    ```
    
2. Open another terminal window on the `workstation` machine. Connect to the `world` database with the local MySQL client on the `workstation` machine. Log in as the `redhat` user with the `redhat123` password. Specify the host as the localhost `127.0.0.1` IP address and use `3306` as the port.
    
    ```
    [student@workstation ~]$mysql -u redhat -p -h 127.0.0.1 -P 3306
    Enter password:redhat123
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    ...output omitted...
    mysql>
    ```
    
3. Select the `world` database and execute the `SHOW TABLES;` command.
    
    ```
    mysql>USE world;
    Database changed
    ```
    
    ```
    mysql>SHOW TABLES;
    -----------------
    | Tables_in_test  |
    -----------------
    | city            |
    | country         |
    | countryinfo     |
    | countrylanguage |
    -----------------
    4 rows in set (0.01 sec)
    ```
    
4. Confirm that you can retrieve data from the `country` table. Execute the `SELECT COUNT(*) FROM country` command to retrieve the number of countries within the `country` table.
    
    ```
    mysql>SELECT COUNT(*) FROM country;
    ----------
    | COUNT(*) |
    ----------
    |      239 |
    ----------
    1 row in set (0.01 sec)
    ```
    
5. Exit the database.
    
    ```
    mysql>exit;
    Bye
    [student@workstation ~]$
    ```
    
6. Return to the terminal that is executing the `oc port-forward` command. Press **Ctrl**+**C** to end the connection.
    
    ```
    [student@workstation ~]$ oc port-forward mysql-server 3306:3306
    Forwarding from 127.0.0.1:3306 -> 3306
    Forwarding from [::1]:3306 -> 3306
    Handling connection for 3306
    Handling connection for 3306
    ^C[student@workstation ~]$
    ```
    

# Lab

1. Log in to the OpenShift cluster and change to the `pods-review` project. 
    1. Log in to the OpenShift cluster.
        
        ```bash
        [student@workstation ~]$ **oc login -u developer -p developer \
        https://api.ocp4.example.com:6443***...output omitted...*
        ```
        
    2. Select the `pods-review` project.
        
        ```bash
        [student@workstation ~]$ **oc project pods-review**
        Now using project "pods-review" on server "https://api.ocp4.example.com:6443".
        *...output omitted...*
        ```
        
2. Deploy a pod named `webphp` that uses the  `registry.ocp4.example.com:8443/redhattraining/webphp:v1` container image. Determine why the pod fails to start.
    1. Deploy a pod named `webphp` that uses the `registry.ocp4.example.com:8443/redhattraining/webphp:v1` container image.
        
        ```bash
        [student@workstation ~]$ **oc run webphp --image=registry.ocp4.example.com:8443/redhattraining/webphp:v1**
        pod/webphp created
        ```
        
    2. After a few moments, observe the status of the `webphp` pod.
        
        ```bash
        [student@workstation ~]$ **oc get pods**
        NAME     READY   STATUS             RESTARTS     AGE
        webphp   0/1     CrashLoopBackOff   1 (4s ago)   7s
        [student@workstation ~]$ **oc get pods**
        NAME     READY   STATUS    RESTARTS     AGE
        webphp   0/1     Error     2 (24s ago)   7s
        The pod failed to start.
        ```
        
    3. Retrieve the cluster events.
        
        ```bash
        [student@workstation ~]$ **oc get events**
        LAST SEEN   TYPE      REASON           OBJECT       MESSAGE
        3m25s       Normal    Scheduled        pod/webphp   Successfully assigned pods-review/webphp to master01 by master01
        3m23s       Normal    AddedInterface   pod/webphp   Add eth0 [10.8.0.73/23] from ovn-kubernetes
        3m23s       Normal    Pulling          pod/webphp   Pulling image "registry.ocp4.example.com:8443/redhattraining/webphp:v1"
        3m15s       Normal    Pulled           pod/webphp   Successfully pulled image "registry.ocp4.example.com:8443/redhattraining/webphp:v1" in 7.894992669s
        104s        Normal    Created          pod/webphp   Created container webphp
        104s        Normal    Started          pod/webphp   Started container webphp
        104s        Normal    Pulled           pod/webphp   Container image "registry.ocp4.example.com:8443/redhattraining/webphp:v1" already present on machine
        103s        Warning   BackOff          pod/webphp   Back-off restarting failed container
        ```
        
    4. Retrieve the logs for the `webphp` pod.
        
        ```bash
        [student@workstation ~]$ **oc logs webphp**
        [20-Dec-2022 18:46:56] NOTICE: [pool www] 'user' directive is ignored when FPM is not running as root
        [20-Dec-2022 18:46:56] NOTICE: [pool www] 'group' directive is ignored when FPM is not running as root
        [20-Dec-2022 18:46:56] **ERROR: unable to bind listening socket for address '/run/php-fpm/www.sock': Permission denied (13)**
        [20-Dec-2022 18:46:56] ERROR: FPM initialization failed
        AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.8.0.62. Set the 'ServerName' directive globally to suppress this message
        **(13)Permission denied: AH00058: Error retrieving pid file run/httpd.pid**
        AH00059: Remove it before continuing if it is corrupted.
        ```
        
        The logs indicate permission issues with the /run directory within the pod.
        
3. Troubleshoot the failed `webphp` pod by creating a debug pod.
    1. Create a debug pod to troubleshoot the failed `webphp` pod.
        
        ```bash
        [student@workstation ~]$ **oc debug pod/webphp**
        Starting pod/webphp-debug ...
        Pod IP: 10.8.0.63
        If you don't see a command prompt, try pressing enter.
        sh-4.4$
        ```
        
    2. List the contents of the `/run` directory to retrieve the permissions, owners, and groups.
        
        ```bash
        sh-4.4$ **ls -la /run**
        total 0
        drwxr-xr-x. 1 root root   42 Dec 20 18:47 .
        dr-xr-xr-x. 1 root root   17 Dec 20 18:47 ..
        -rw-r--r--. 1 root root    0 Dec 20 18:47 .containerenv
        drwx--x---. 3 root apache 26 Dec 20 18:42 httpd
        drwxr-xr-x. 2 root root    6 Oct 26 11:10 lock
        drwxr-xr-x. 2 root root    6 Dec 20 18:42 php-fpm
        drwxr-xr-x. 4 root root   80 Dec 20 18:47 secrets
        ```
        
        <aside>
        <img src="https://www.notion.so/icons/light-bulb_blue.svg" alt="https://www.notion.so/icons/light-bulb_blue.svg" width="40px" /> drwxr-xr-x.
        
        - `d` indicates that it's a directory.
        - `rwx` (first triplet) indicates that the owner (`root`) has read, write, and execute permissions.
        - `r-x` (second triplet) indicates that the group (`root`) has read and execute permissions, but not write permissions.
        - `r-x` (third triplet) indicates that others (users who are not the owner or part of the group) have read and execute permissions, but not write permissions.
        </aside>
        
        The `/run/httpd` directory grants read, write, and execute permissions to the `root` user, but does not provide permissions for the `root` group.
        
    3. Retrieve the UID and GID of the user in the container. Determine whether the user is a privileged user and belongs to the `root` group.
        
        ```bash
        sh-4.4$ **id**
        uid=1000680000(1000680000) gid=0(root) groups=0(root),1000680000
        ```
        
    4. Your UID and GID values might differ from the previous output.
    The user is an unprivileged, `non-root` user and belongs to the `root` group, which does not have access to the `/run` directory. Therefore, the user in the container cannot access the files and directories that the container processes use, which is required for arbitrarily assigned UIDs.
    5. Exit the debug pod
        
        ```bash
        sh-4.4$ **exit**
        exit
        
        Removing debug pod ...
        ```
        
4. The application developer resolved the identified issue in the `registry.ocp4.example.com:8443/redhattraining/webphp:v2` container image. In a terminal window, edit the `webphp` pod resource to use the `v2` image tag. Retrieve the status of the `webphp` pod. Then, confirm that the user in the container is an unprivileged user and belongs to the `root` group. Confirm that the `root` group permissions are correct for the `/run/httpd` directory.
    1. Use the terminal to edit the `webphp` pod resource.
        
        ```bash
        [student@workstation ~]$ **oc edit pod/webphp**
        ```
        
    2. Update the `.spec.containers.image` object value to use the `:v2` image tag.
        
        ```bash
        *...output omitted...*
        spec:
          containers:
          - image: registry.ocp4.example.com:8443/redhattraining/webphp:**v2**
            imagePullPolicy: IfNotPresent
        *...output omitted...*
        ```
        
    3. Verify the status of the `webphp` pod.
        
        ```bash
        [student@workstation ~]$ **oc get pods**
        NAME          READY   STATUS    RESTARTS       AGE
        webphp        1/1     Running   9 (2m9s ago)   18m
        ```
        
    4. Retrieve the UID and GID of the user in the container to confirm that the user is an unprivileged user
        
        ```bash
        [student@workstation ~]$ **oc exec -it webphp -- id**
        uid=1000680000(1000680000) gid=0(root) groups=0(root),1000680000
        ```
        
        Your UID and GID values might differ from the previous output.
        
    5. Confirm that the permissions for the `/run/httpd` directory are correct.
        
        ```bash
        [student@workstation ~]$ **oc exec -it webphp -- ls -la /run/**
        total 0
        drwxr-xr-x. 1 root root 70 Dec 20 19:01 .
        dr-xr-xr-x. 1 root root 39 Dec 20 19:01 ..
        -rw-r--r--. 1 root root  0 Dec 20 18:45 .containerenv
        drwxrwx---. 1 root root 41 Dec 20 19:01 httpd
        drwxr-xr-x. 2 root root  6 Oct 26 11:10 lock
        drwxrwxr-x. 1 root root 41 Dec 20 19:01 php-fpm
        drwxr-xr-x. 4 root root 80 Dec 20 19:01 secrets
        ```
        
5. Connect port `8080` on the `Workstation` machine to port `8080` on the `webphp` pod. In a new terminal window, retrieve the content of the pod's `127.0.0.1:8080/index.php` web page to confirm that the pod is operational.
    
    <aside>
    <img src="https://www.notion.so/icons/light-bulb_blue.svg" alt="https://www.notion.so/icons/light-bulb_blue.svg" width="40px" /> The terminal window that you connect to the `webphp` pod must remain open for the remainder of the lab. This connection is necessary for the final lab step and for the `lab grade` command.
    
    </aside>
    
    1. Connect to port `8080` on the `webphp` pod
        
        ```bash
        [student@workstation ~]$ **oc port-forward pod/webphp 8080:8080**
        Forwarding from 127.0.0.1:8080 -> 8080
        Forwarding from [::1]:8080 -> 8080
        ```
        
    2. Open a second terminal window and then retrieve the `127.0.0.1:8080/index.php` web page on the `webphp` pod.
        
        ```bash
        [student@workstation ~]$ **curl 127.0.0.1:8080/index.php**
        <html>
        <body>
        Hello, World!
        </body>
        </html>
        ```
        
6. An issue occurs with the PHP application that is running on the `webphp` pod. To debug the issue, the application developer requires diagnostic and configuration information for the PHP instance that is running on the `webphp` pod.
The  `~/DO180/labs/pods-review` directory contains a `phpinfo.php` file to generate debugging information for a PHP instance. Copy the `phpinfo.php` file to the  `/var/www/html/` directory on the `webphp` pod.
Then, confirm that the PHP debugging information is displayed when accessing the `127.0.0.1:8080/phpinfo.php` from a web browser.
    1. In the second terminal, copy the `~/DO180/labs/pods-review/phpinfo.php` file to the `webphp` pod as the `/var/www/html/phpinfo.php` file.
        
        ```bash
        [student@workstation ~]$ **oc cp ~/DO180/labs/pods-review/phpinfo.php webphp:/var/www/html/phpinfo.php**
        ```
        
    2. Open a web browser and access the `127.0.0.1:8080/phpinfo.php` web page. Confirm that PHP debugging information is displayed.PHP debugging information
        
        ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/pods/review/assets/php-debug-console.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/pods/review/assets/php-debug-console.png)
        
    3. Return to the terminal that is executing the `oc port-forward` command. Press **Ctrl**+**C** to end the connection
        
        ```bash
        [student@workstation ~]$ **oc port-forward pod/webphp 8080:8080**
        Forwarding from 127.0.0.1:8080 -> 8080
        Forwarding from [::1]:8080 -> 8080
        Handling connection for 8080
        **^C**[student@workstation ~]$
        ```