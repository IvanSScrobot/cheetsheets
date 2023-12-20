# Ephemeral Containers

```
# Save the Pod name for future reference:
$ POD_NAME=$(kubectl get pods -l app=my_app_name -o jsonpath='{.items[0].metadata.name}')
```

## Inspect Pods:

```
kubectl debug -it --attach=false -c debugger --image=busybox ${POD_NAME}
```

To get debug containers and their status:
```
kubectl get pod ${POD_NAME} \
  -o jsonpath='{.spec.ephemeralContainers}' \
  | python3 -m json.tool


kubectl get pod ${POD_NAME} \
  -o jsonpath='{.status.ephemeralContainerStatuses}' \
  | python3 -m json.tool  
```

To attach to the debug container:
```
kubectl attach -it -c debugger ${POD_NAME}
```

## Using ephemeral containers with a shared pid namespace

To make processes in main containers visible from an ephemeral one, patch the Pod template:
```
kubectl patch deployment slim --patch '
spec:
  template:
    spec:
      shareProcessNamespace: true'

kubectl debug -it -c debugger --image=busybox ${POD_NAME} 
```

## Using ephemeral containers with a shared filesystems 
To access filesystems of misbehaving containers, 
1) Get PIDs of processs from the containers you want to explore
2) Run inside the ephemeral container
```
ls /proc/<PID>/root/usr/bin
```

