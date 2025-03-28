# Lab CH3: Run Applications as Containers and Pods

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