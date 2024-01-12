# Debugging
 
If your container has previously crashed, you can access the previous container's crash log with:
```
kubectl logs --previous ${POD_NAME} ${CONTAINER_NAME}
```

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

## Debug a single container
```
kubectl debug -it -c debugger --target=target_container --image=busybox ${POD_NAME}
```
Ephemeral container can see processes of the target container

## Debug a copy of target Pod
```
kubectl debug -it -c debugger --image=busybox \
  --copy-to test-pod \
  --share-processes \
  ${POD_NAME}
```

## Ephemeral Containers Without kubectl debug
https://iximiuz.com/en/posts/kubernetes-ephemeral-containers/
