# Chapter 1.  Introduction to Kubernetes and OpenShift

## Quiz 1

![Untitled](Chapter%201%20Introduction%20to%20Kubernetes%20and%20OpenShift/1322179b-a222-4154-a9db-157f509630bf.png)

## Quiz 2

1. What is the installed version for the OpenShift cluster
    1. 4.14.0
    2. 3.9.1
    3. 4.3.2
    4. 5.4.5
2. Which severity types are available for the alerts in the cluster
    1. Warning
    2. info
    3. critical 
    4. Firing
    5. urgent
    6. Oops
3. Which labels are on the `thanos-querier` route in the `openshift-monitoring` namespace? 
    1. app.kubernetes.io/component=query-layer
    2. app.kubernetes.io/instance=thanos-querier
    3. app.kubernetes.io/version=0.30.2
    4. app.kubernetes.io/part-of=openshift-storage
4. Which objects are listed as the `StorageClasses` objects for the cluster?
    1. ceph-storage
    2. nfs-storage
    3. lvms-vg1
    4. k8s-lvm-vg1
    5. local-volume
5. When **All Projects** is selected at the top, which three operators are installed in the cluster?
    1. MongoDB Operator
    2. MetalLB Operator
    3. Red Hat Fuse
    4. LVM Storage
    5. Package Server

# Lab: Introduction to Kubernetes and OpenShift

### Log in to the Red Hat OpenShift Container Platform web console, with Red Hat Identity Management as the `admin` user with the `redhatocp` password, and review the answers for the preceding quiz.

1. Use a browser to view the login page at the `https://console-openshift-console.apps.ocp4.example.com` address.
    
    ![Untitled](Chapter%201%20Introduction%20to%20Kubernetes%20and%20OpenShift/Untitled.png)
    
2. Click Red Hat Identity Management, and supply the `admin` username and the `redhatocp` password, and then click `Log in` to access the home page.
    
    ![Untitled](Chapter%201%20Introduction%20to%20Kubernetes%20and%20OpenShift/Untitled%201.png)
    

### View the cluster version on the Overview page for the cluster.

1. From the **Home** → **Overview** page, scroll down to view the cluster details.
    
    ![Untitled](Chapter%201%20Introduction%20to%20Kubernetes%20and%20OpenShift/Untitled%202.png)
    
2. Locate the OpenShift version in the Details section.

![Untitled](Chapter%201%20Introduction%20to%20Kubernetes%20and%20OpenShift/Untitled%203.png)

### View the available alert severity types within the filters on the Alerting page.

1. Navigate to the **Observe** → **Alerting** page.
    
    ![Untitled](Chapter%201%20Introduction%20to%20Kubernetes%20and%20OpenShift/Untitled%204.png)
    
2. Click the Filter drop-down to view the available severity options.
    
    ![Untitled](Chapter%201%20Introduction%20to%20Kubernetes%20and%20OpenShift/Untitled%205.png)
    

### View the labels for the `thanos-querier` route.

1. Navigate to the **Networking** → **Routes** page.
    
    ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/routes.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/routes.png)
    
2. Type the `thanos` keyword in the text search field.
    
    ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/thanos.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/thanos.png)
    
3. Select the `thanos-querier` route in the Name column.Scroll down on the `thanos-querier` route details page to view the labels.
    
    ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/select.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/select.png)
    
4. Scroll down on the `thanos-querier` route details page to view the labels.
    
    ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/labels.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/labels.png)
    

### View the available storage classes in the cluster.

1. Navigate to the **Storage** → **StorageClasses** page.
    
    ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/Storage.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/Storage.png)
    
2. View the available storage classes in the cluster.

![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/Classes.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/Classes.png)

### View the installed operators for the cluster.

1.Navigate to the **Operators** → **Installed Operators** page.

![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/operators.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/operators.png)

1. View the list of installed operators in the cluster.
    
    ![https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/installed.png](https://rha.ole.redhat.com/rha/static/static_file_cache/do180-4.14/intro/review/assets/installed.png)