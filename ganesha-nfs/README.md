Based on https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner/tree/master

To install:

* `helm repo add nfs-ganesha-server-and-external-provisioner https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner/`
* `helm install my-nfs-release nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner -n nfs  --create-namespace -f values.yaml`

To test:

* `kubectl apply -f example-volume.yaml -n nfs` - this creates a sample volume that uses the new `nfs` storage class (itself provisioned out of a config volume handled by the Helm chart with parameters from the values file)
* `kubectl apply -f write-pod.yaml -n nfs` - creates a pod that then claims the sample volume and writes a thing to it. Additionally it sleeps for a bit to hang on to the volume.
* `kubectl apply -f write-pod2.yaml -n nfs` - creates a second test pod that does the same to then show the `ReadWriteMany` is working (otherwise it couldn't claim the same volume)
* `kubectl exec -it write-pod -n nfs -- /bin/sh` - to get into one test pod, then `ls /mnt` to see what's in there

To use create a real volume meant to be your NFS provisioning home that includes this snippet as seen in the example:

```
spec:
  storageClassName: "nfs" 
  accessModes:
    - ReadWriteMany
```

To uninstall:

* `helm uninstall my-nfs-release -n nfs`
