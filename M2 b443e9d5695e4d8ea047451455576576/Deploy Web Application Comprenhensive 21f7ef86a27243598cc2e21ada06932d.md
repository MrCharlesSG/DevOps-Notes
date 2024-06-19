# Deploy Web Application Comprenhensive

# Deploy Web Application Step By Step

1. Login
    
    ```bash
    oc login -u developer -p developer https://api.ocp4.example.com:6443
    ```
    
2. Create project
    
    ```bash
    oc new-project projectName
    ```
    

## Database

1. Create Images Stream if necessary.  Can use as imageName  the official url
2. Secrets if needed
3. Make deployment ( Set the number of replicas to zero, to prevent OpenShift from deploying before you finish its configuration.)
    
    ```bash
    oc create deployment deploymentName --image imageName:ImageVersion --replicas numOfReplicas
    ```
    
    Check
    
    ```bash
    oc get deployment deploymentName -o wide
    ```
    
4. Add trigger to image if neccesary
5. Set eviroment variables if neccesary
6. Set volumes if neccessary
7. Scale deployment 
    
    ```bash
    oc scale deployment/deploymentName --replicas newNumOfReplicas
    ```
    
    For checking
    
    ```bash
    oc get pods
    ```
    
8. Expose deployment by creating a service
    
    ```bash
    oc expose deployment deploymentName --port listeningPort
    ```
    
    check
    
    ```bash
    oc describe service deploymentName 
    ```
    

## Frontend

1. Create Images Stream if necessary.  Can use as imageName  the official url
2. Make deployment ( Set the number of replicas to zero, to prevent OpenShift from deploying before you finish its configuration.)
    
    ```bash
    oc create deployment deploymentName --image imageName --replicas numOfReplicas
    ```
    
    Check
    
    ```bash
    oc get deployment deploymentName -o wide
    ```
    
3. Add trigger to image if neccesary
4. Set eviroment variables to connect to the database
5. Scale deployment 
    
    ```bash
    oc scale deployment/deploymentName --replicas newNumOfReplicas
    ```
    
    For checking
    
    ```bash
    oc get pods
    ```
    
6. Expose deployment by creating a service
    
    ```bash
    oc expose deployment deploymentName --port listeningPort
    ```
    
    check
    
    ```bash
    oc describe service deploymentName 
    ```
    
7. Create the route by exposing the service
    
    ```
    oc expose service deploymentName 
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
    

## Commands

### Image Stream

- Create Image Stream.
    
    An image stream stores references to specific images of external or internal repositories
    
    ```java
    oc create istag imageName:ImageVersion --from-image imageRepo
    ```
    
    Check
    
    ```bash
    oc get istag
    oc describe istag imageName:ImageVersion
    ```
    
- Create image lookup
    
    Lookup is a functionality for OpenShift to resolve which Image Streams are we refering
    
    ```bash
    oc set image-lookup Imagename
    ```
    
    Check
    
    ```bash
     oc set image-lookup
    ```
    
- Change version of current project image. 2 Options
    1. Tag the old version
        
        ```bash
        oc tag newImageRepo oldImageName:oldImageVersion
        ```
        
    2. Tag a new version and say to the deployment
        
        ```bash
        oc tag registry newImageRepo oldImageName:newImageVersion
        oc set image deployment/deploymentName oldImageName=oldImageName:newImageVersion
        # Make sure the image is with a roll out if is a database
        ```
        

### Secrets

- Create Secrets from scrach
    
    ```bash
    oc create secret generic secretName --from-literal paramName1=paramValue1 --from-literal paramName2=paramValue2...
    ```
    
    Check
    
    ```bash
    oc get secret secretName 
    oc describe secret secretName 
    ```
    

### Triggers

- Create a trigger of an image stream tag so it rollout everytime it changes
    
    ```bash
    oc set triggers deployment/deploymentName --from-image imageName:imageVersion --containers containerOfImageName
    ```
    
    Check
    
    ```bash
    oc get deployment deploymentName -o jsonpath='{.spec.triggers}'
    oc describe deployment deploymentName
    ```
    

### Enviroment Variables

- Set enviroment variables from secrets
    
    ```bash
    oc set env deployment/deploymentName --from secret/secretName --prefix prefix
    ```
    
    Check 
    
    ```bash
    oc set env deployment/deploymentName --list
    ```
    
- Declare enviroment variable
    
    ```bash
    oc set env deployment/deploymentName variableName=variableValue
    ```
    
    Check 
    
    ```bash
    oc set env deployment/deploymentName --list
    ```
    

### Volumes

- Create volume
    
    ```bash
    oc set volumes deployment/deploymentName --add --claim-class storageClass --claim-size diskSpace(2Gi) --mount-path dataDirectory 
    ```
    
    Check
    
    ```bash
    oc get deployment deploymentName -o jsonpath='{.spec.template.spec.volumes}'
    oc describe deployment deploymentName 
    ```
    
    remove a volume from a deployment.
    
    ```bash
    oc set volumes deployment/deploymentName --remove --name=volumeName
    ```
    
    Check
