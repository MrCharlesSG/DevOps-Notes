# Commands

# General

# Deployment

- Scale application
    
    ```bash
    oc scale deployment/*deploymentName* --replicas *numReplicas*
    ```
    

## Probe

```
[student@workstation ~]$oc set probe deployment/quotesdb \--readiness -- mysqladmin ping
deployment.apps/quotesdb probes updated
```

```
oc set probe deployment/*deploymentName* --readiness -- mysqladmin ping
```

# Images

- Change Image

```bash
oc set image deployment/*deploymentName containerName*=*newIageRepo*
```

# Special

- Set Resources
    
    ```bash
    oc set resources deployment/*deploymentsName* --requests cpu=*cpu*,memory=*memory* --limits cpu=cpu,memory=memory
    # 5 militrones = 5m, 5GB = 5Gi
    ```
    

# Enviroment

# Pods

# Troubleshoot

```bash
oc logs *podName*
```

# Probe

```bash
oc set probe deployment/*deploymentName* --*option* -- *command*
```

readiness and liveness  as options