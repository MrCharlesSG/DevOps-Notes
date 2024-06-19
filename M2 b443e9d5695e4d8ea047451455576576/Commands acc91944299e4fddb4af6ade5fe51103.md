# Commands

# General

```bash
oc get pods
```

# Deployment

- Deploy Application
    
    ```bash
    oc create deployment *deploymentName* --image *imageName*:*ImageVersion* --replicas *numOfReplicas*
    ```
    
    Check
    
    ```bash
    oc get deployment *deploymentName* -o wide
    ```
    
- Scale application
    
    ```bash
    oc scale deployment/*deploymentName* --replicas *numReplicas*
    ```
    
    Check
    
    ```bash
    oc get pods
    ```
    
- Expose Deployment
    
    ```bash
    oc expose deployment *deploymentName* --port *listeningPort*
    ```
    
    check
    
    ```bash
    oc describe service *deploymentName* 
    ```
    
- Expose Service
    
    ```
    oc expose service *deploymentName* 
    ```
    
    check
    
    ```bash
    oc get route
    ```
    
    ```bash
    # result
    NAME       HOST/PORT       PATH   SERVICES ...
    ```
    
    Check in browser or in console by
    
    ```bash
    curl http://HOST/PORT   
    ```
    

# Images

- Create Image Stream.
    
    An image stream stores references to specific images of external or internal repositories
    
    ```java
    oc create istag *imageName:ImageVersion* --from-image i*mageRepo*
    ```
    
    Check
    
    ```bash
    oc get istag
    oc describe istag *imageName:ImageVersion*
    ```
    
- Create image lookup
    
    Lookup is a functionality for OpenShift to resolve which Image Streams are we refering
    
    ```bash
    oc set image-lookup Image*name*
    ```
    
    Check
    
    ```bash
     oc set image-lookup
    ```
    
- Change Image of Deployment
    
    ```bash
    oc set image deployment/*deploymentName containerName*=*newIageRepo*
    ```
    
- Inpect Available image in repository
    
    ```
    [student@workstation ~]$skopeo login registry.ocp4.example.com:8443
    Username:developer
    Password:developer
    Login Succeeded!
    ```
    
    ```
    [student@workstation ~]$skopeo inspect *ImageRepo*
    ...output omitted...
        "Name": "*ImageRepo*",
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
    

# Special

### Secrets

- Create Secrets from scrach
    
    ```bash
    oc create secret generic *secretName* --from-literal *paramName1=paramValue1* --from-literal *paramName2=paramValue2...*
    ```
    
    Check
    
    ```bash
    oc get secret *secretName* 
    oc describe secret *secretName* 
    ```
    

### Triggers

- Create a trigger of an image stream tag so it rollout everytime it changes
    
    ```bash
    oc set triggers deployment/*deploymentName* --from-image *imageName:imageVersion --containers* containerOfImageName
    ```
    
    Check
    
    ```bash
    oc describe deployment *deploymentName*
    ```
    

### Enviroment Variables

- Set enviroment variables from secrets
    
    ```bash
    oc set env deployment/deploymentName --from secret/*secretName* --prefix *prefix*
    ```
    
    Check 
    
    ```bash
    oc set env deployment/deploymentName --list
    ```
    
- Declare enviroment variable
    
    ```bash
    oc set env deployment/*deploymentName* *variableName=variableValue*
    ```
    
    Check 
    
    ```bash
    oc set env deployment/deploymentName --list
    ```
    

### Volumes

- Create volume
    
    ```bash
    oc set volumes deployment/*deploymentName* --add --claim-class *storageClass* --claim-size *diskSpace(2Gi)* --mount-path *dataDirectory* 
    ```
    
    Check
    
    ```bash
    oc get deployment *deploymentName* -o jsonpath='{.spec.template.spec.volumes}'
    oc describe deployment *deploymentName* 
    ```
    
    remove a volume from a deployment.
    
    ```bash
    oc set volumes deployment/*deploymentName* --remove --name=*volumeName*
    ```
    
    Check
    

### Resources

```bash
oc set resources deployment/*deploymentsName* --requests cpu=*cpu*,memory=*memory* --limits cpu=cpu,memory=memory
# 5 militrones = 5m, 5GB = 5Gi, 5MB = 5Mi
```

# Pods

- Create Pod from Image
    
    ```bash
    oc run *podName* --image *imageRepo*
    ```
    
- Exec command in pod
    
    ```bash
    oc exec -it *podmName* -- *command*
    ```
    

# Troubleshoot

```bash
oc logs *podName*
```

```bash
oc get events
```

```bash
oc describe pod *podName*
```

# Probe

```bash
oc set probe deployment/*deploymentName* --*option* -- *command*
```

```bash
oc set probe deployment/*deploymentName* --*option* --get-url *url*
```

- Options
    - `readiness` → determines whetther the application is ready to serve request. This probe enable OpenShift to detect if the deployment is ready
    - `liveness` → regularly verifies the status of the deployment
    - `startup` → determines when an application’s stratup is completed. Consider this when application has a long start time.