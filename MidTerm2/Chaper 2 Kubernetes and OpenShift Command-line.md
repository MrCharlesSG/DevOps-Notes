# Chaper 2.  Kubernetes and OpenShift Command-line Interfaces and APIs

# Guided Exercise

## **The Kubernetes and OpenShift Command-line Interfaces**

### Login

1. To access you execute in your terminal: `oc login -u <USERNAME> -p <PASSWORD> <API_ENDPOINT>:<PORT>`
    
    ```bash
    [student@workstation ~]$ oc login -u developer -p developer https://api.ocp4.example.com:6443
    Login successful.
    
    ...output omitted...
    ```
    
2. The you execute the  `oc whoami --show-console` command to retrieve the web console URL
    
    ```bash
    [user@host ~]$ oc whoami --show-console
    https://console-openshift-console.apps.ocp4.example.com
    ```
    
3. Open a web browser and then navigate to `https://console-openshift-console.apps.ocp4.example.com`.
4. Click **Red Hat Identity Management** and log in as the `developer` user with the `developer` password.
    
    ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/cli/interfaces/assets/login_screen.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/cli/interfaces/assets/login_screen.png)
    
5. Locate the installation file for the `oc` CLI. From the OpenShift web console, select **Help** → **Command line tools**. The **Help** menu is represented by a `?` icon.
    
    ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/cli/interfaces/assets/tools_screen.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/cli/interfaces/assets/tools_screen.png)
    
    The `oc` binary is available for multiple operating systems and architectures. For each operating system and architecture, the `oc` binary also includes the `kubectl` binary.
    

### Authorization Token

Download an authorization token from the web console. Then, use the token and the `oc` command to log in to the OpenShift cluster.

1. From the **Command Line Tools** page, click the **Copy login command** link.
2. The link opens a login page. Click **Red Hat Identity Management** and log in as the `developer` user with the `developer` password.
3. A web page is displayed. Click the **Display token** link to show your API token and the login command.
    
    ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/cli/interfaces/assets/token.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/cli/interfaces/assets/token.png)
    
    ---
    
4. Copy the `oc login` command to your clipboard. Open a terminal window and then use the copied command to log in to the cluster with your token.
    
    ```
    [student@workstation ~]$oc login --token=sha256-fypX...Ot6A \
      --server=https://api.ocp4.example.com:6443
    Logged into "https://api.ocp4.example.com:6443" as "developer" using the token provided.
    ...output omitted...
    ```
    

# **Lab: Kubernetes and OpenShift Command-line Interfaces and APIs**

Log in to the OpenShift cluster as the `developer` user with the `developer` password. Use the `cli-review` project for your work.

1. Log in to the OpenShift cluster and create the `cli-review` project.
    
    1. Log in to the OpenShift cluster.
    
    ```bash
    [student@workstation ~]$ **oc login -u developer -p developer \
      https://api.ocp4.example.com:6443***...output omitted...*
    ```
    
    1. Create the `cli-review` project.
    
    ```bash
    [student@workstation ~]$ **oc new-project cli-review**
    Now using project "cli-review" on server "https://api.ocp4.example.com:6443".
    *...output omitted...*
    ```
    
2. Use the `oc` command to list the following information for the cluster:
    - Retrieve the cluster version.
    - Identify the supported API versions.
    - Identify the fields for the `pod.spec.securityContext` object.
    
    1. Identify the cluster version.
    
    ```bash
    [student@workstation ~]$ **oc version**
    Client Version: 4.14.0
    Kustomize Version: v5.0.1
    Kubernetes Version: v1.27.6+f67aeb3
    ```
    
    1. Identify the supported API versions.
    
    ```bash
    [student@workstation ~]$ **oc api-versions**
    admissionregistration.k8s.io/v1
    apiextensions.k8s.io/v1
    apiregistration.k8s.io/v1
    apiserver.openshift.io/v1
    apps.openshift.io/v1
    apps/v1
    *...output omitted...*
    ```
    
    1. Identify the fields for the `pod.spec.securityContext` object.
    
    ```bash
    [student@workstation ~]$ **oc explain pod.spec.securityContext**
    KIND:     Pod
    VERSION:  v1
    
    FIELD: securityContext <PodSecurityContext>
    
    DESCRIPTION:
    *...output omitted...*
    ```
    
3. From the terminal, log in to the OpenShift cluster as the `admin` user with the `redhatocp` password. Then, use the command line to identify the following cluster resources:
    - List the cluster operators.
    - Identify the available namespaced resources.
    - Identify the resources that belong to the core API group.
    - List the resource types that the `oauth.openshift.io` API group provides.
    - List the events in the `openshift-kube-controller-manager` namespace.
    
    1. Log in to the OpenShift cluster.
    
    ```bash
    [student@workstation ~]$ **oc login -u admin -p redhatocp \
      https://api.ocp4.example.com:6443***...output omitted...*
    ```
    
    1. List the cluster operators.
    
    ```bash
    [student@workstation ~]$ **oc get clusteroperators**
    NAME                         VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
    authentication               4.14.0    True        False         False      12h
    baremetal                    4.14.0    True        False         False      31d
    cloud-controller-manager     4.14.0    True        False         False      31d
    cloud-credential             4.14.0    True        False         False      31d
    cluster-autoscaler           4.14.0    True        False         False      31d
    config-operator              4.14.0    True        False         False      31d
    console                      4.14.0    True        False         False      31d
    *...output omitted..*
    ```
    
    1. List the available namespaced resources.
    
    ```bash
    [student@workstation ~]$ **oc api-resources --namespaced**
    NAME                    SHORTNAMES  APIVERSION  NAMESPACED  KIND
    bindings                            v1          true        Binding
    configmaps              cm          v1          true        ConfigMap
    endpoints               ep          v1          true        Endpoints
    events                  ev          v1          true        Event
    limitranges             limits      v1          true        LimitRange
    persistentvolumeclaims  pvc         v1          true        PersistentVolumeClaim
    pods                    po          v1          true        Pod
    *...output omitted...*
    ```
    
    1. Identify the resources that belong to the core API group.
    
    ```bash
    [student@workstation ~]$ **oc api-resources --api-group ''**
    NAME                     SHORTNAMES   APIVERSION   NAMESPACED   KIND
    bindings                              v1           true         Binding
    componentstatuses        cs           v1           false        ComponentStatus
    configmaps               cm           v1           true         ConfigMap
    endpoints                ep           v1           true         Endpoints
    events                   ev           v1           true         Event
    limitranges              limits       v1           true         LimitRange
    namespaces               ns           v1           false        Namespace
    nodes                    no           v1           false        Node
    *...output omitted...*
    ```
    
    1. List the resource types that the `oauth.openshift.io` API group provides
    
    ```bash
    [student@workstation ~]$ **oc api-resources --api-group oauth.openshift.io**
    NAME                 SHORTNAMES  APIVERSION            NAMESPACED KIND
    oauthaccesstokens                oauth.openshift.io/v1 false      OAuthAccessToken
    oauthauthorizationtokens         oauth.openshift.io/v1 false      OAutheAuthorizationToken
    *...output omitted...*
    ```
    
    1. Retrieve the events for the `openshift-kube-controller-manager` namespace
    
    ```bash
    [student@workstation ~]$ **oc get events -n openshift-kube-controller-manager**
    LAST SEEN TYPE    REASON                  OBJECT                              ...
    48m       Normal  CreatedSCCRanges        pod/kube-controller-manager-master ...
    21m       Normal  CreatedSCCRanges        pod/kube-controller-manager-master ...
    14m       Normal  CreatedSCCRanges        pod/kube-controller-manager-master ...
    ```
    
4. Identify the following information about the cluster services and its nodes:
    - Retrieve the `conditions` status of the `etcd-master01` pod in the `openshift-etcd` namespace by using `jq` filters to limit the output.
    - List the compute resource usage of the containers in the `etcd-master01` pod in the `openshift-etcd` namespace.
    - Get the number of allocatable pods for the `master01` node by using a JSONPath filter.
    - List the memory and CPU usage of all pods in the cluster.
    - Retrieve the compute resource consumption of the `master01` node.
    - Retrieve the capacity and allocatable CPU for the `master01` node by using a JSONPath filter.
    
    1. Retrieve the `conditions` status of the `etcd-master01` pod in the `openshift-etcd` namespace. Use `jq` filters to limit the output to the `.status.conditions` attribute of the pod.
    
    ```bash
    [student@workstation ~]$ **oc get pods etcd-master01 -n openshift-etcd \
      -o json | jq .status.conditions**
    [
      {
        "lastProbeTime": null,
        "lastTransitionTime": "2023-03-12T16:40:35Z",
        "status": "True",
        "type": "Initialized"
      },
      {
        "lastProbeTime": null,
        "lastTransitionTime": "2023-03-12T16:40:47Z",
        "status": "True",
        "type": "Ready"
      },
      {
        "lastProbeTime": null,
        "lastTransitionTime": "2023-03-12T16:40:47Z",
        "status": "True",
        "type": "ContainersReady"
      },
      {
        "lastProbeTime": null,
        "lastTransitionTime": "2023-03-12T16:40:23Z",
        "status": "True",
        "type": "PodScheduled"
      }
    ]
    ```
    
    1.  List the resource usage of the containers in the `etcd-master01` pod in the `openshift-etcd` namespace.
        
        ```bash
        [student@workstation ~]$ **oc adm top pods etcd-master01 \
          -n openshift-etcd --containers**
        POD             NAME           CPU(cores)   MEMORY(bytes)
        etcd-master01   POD            0m           0Mi
        etcd-master01   etcd           54m          1513Mi
        etcd-master01   etcd-metrics   5m           24Mi
        etcd-master01   etcd-readyz    4m           39Mi
        etcd-master01   etcdctl        0m           0Mi
        ```
        
    2. Use a JSONPath filter to determine the number of allocatable pods for the `master01` node.
        
        ```bash
        [student@workstation ~]$ **oc get node master01 \
          -o jsonpath='{.status.allocatable.pods}{"\n"}'**
        250
        ```
        
    3. List the memory and CPU usage of all pods in the cluster. Use the `--sum` option to print the sum of the resource usage. The resource usage on your system probably differs
        
        ```bash
        [student@workstation ~]$ **oc adm top pods -A --sum**
        NAMESPACE        NAME                                    CPU(cores)  MEMORY(bytes)
        metallb-system   controller-5f6dfd8c4f-ddr8v             0m          56Mi
        metallb-system   metallb-operator-controller-manager-... 0m          50Mi
        metallb-system   metallb-operator-webhook-server-...     0m          26Mi
        metallb-system   speaker-2dds4                           9m          210Mi
        *...output omitted...*
                                                                 --------    --------
                                                                 505m        8982Mi
        ```
        
    4. Retrieve the resource consumption of the `master01` node.
        
        ```bash
        [student@workstation ~]$ **oc adm top node**
        NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
        master01   1199m        15%    12555Mi         66%
        ```
        
    5. Use a JSONPath filter to determine the capacity and allocatable CPU for the `master01` node.
        
        ```bash
        [student@workstation ~]$ **oc get node master01 -o jsonpath=\
        'Allocatable: {.status.allocatable.cpu}{"\n"}'\
        'Capacity: {.status.capacity.cpu}{"\n"}'**
        Allocatable: 7500m
        Capacity: 8
        ```
        
5. Retrieve debugging information for the cluster. Specify the `/home/student/DO180/labs/cli-review/debugging` directory as the destination directory.
    
    Then, generate debugging information for the `kube-apiserver` cluster operator. Specify the `/home/student/DO180/labs/cli-review/inspect` directory as the destination directory. Limit the debugging information to the last five minutes.
    
    1. Retrieve debugging information for the cluster. Save the output to the `/home/student/DO180/labs/cli-review/debugging` directory.
    
    ```bash
    [student@workstation ~]$ **oc adm must-gather \
      --dest-dir /home/student/DO180/labs/cli-review/debugging**
    [must-gather      ] OUT Using must-gather plug-in image: quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:07d3...e94c
    *...output omitted...*
    Reprinting Cluster State:
    When opening a support case, bugzilla, or issue please include the following summary data along with any other requested information:
    ClusterID: 94ff22c1-88a0-44cf-90f6-0b7b8b545434
    ClusterVersion: Stable at "4.14.0"
    ClusterOperators:
            All healthy and stable
    ```
    
    2. Generate debugging information for the `kube-apiserver` cluster operator. Save the output to the `/home/student/DO180/labs/cli-review/inspect` directory, and limit the debugging information to the last five minutes.
    
    ```bash
    [student@workstation ~]$ **oc adm inspect clusteroperator kube-apiserver \
      --dest-dir /home/student/DO180/labs/cli-review/inspect --since 5m**
    Gathering data for ns/metallb-system...
    *...output omitted...*
    Wrote inspect data to /home/student/DO180/labs/cli-review/inspect.
    ```
    

# Quiz

1. Which supported API resource is a member of the `oauth.openshift.io` api-group?
    1. groupoauthaccesstokens
    2. tokenreviews
    3. cloudcredentials
    4. networkpolicies
2. Which two fields are members of the pod.spec.securityContext object? (Choose two.)
    1. readinessGates
    2. runAsUser
    3. sysctls
    4. priorityClassName
3. Which three commands display the conditions of the `master01` node? (Choose three.)
    1. oc get node/master01 -o json | jq '.status.conditions’
    2. oc get node/master01 -o wide
    3. oc get node/master01 -o yaml
    4. oc get node/master01 -o json
4. Select the two valid condition types for a control plane node. (Choose two)
    1. PIDPressure
    2. DiskIOPressure
    3. OutOfMemory
    4. Ready
5. Select three valid options for the oc adm top pods command. (Choose three.)
    1. -A
    2. —sum
    3. --pod-selector
    4. --containers
6. Which command determines the assigned internal IP address on the master01 node?
    1. oc get nodes master01 -o=jsonpath='{.status.addresses}{"\n"}’
    2. oc get nodes master01 -o json | jq .status.nodeInfo
    3. oc get nodes master01 -o=jsonpath='{.status.conditions}{"\n"}’
7. Which command displays only the conditions for the `authentication` cluster operator?
    1. oc get [clusteroperators.config.openshift.io](http://clusteroperators.config.openshift.io/) authentication -o json | jq .status.versions
    2. oc get [clusteroperators.config.openshift.io](http://clusteroperators.config.openshift.io/) authentication -o json | jq .status
    3. oc get [clusteroperators.config.openshift.io](http://clusteroperators.config.openshift.io/) authentication -o json | jq .status.conditions