# Kubic Game Hosting

The project is a top level repo for the Kubic-style game hosting projects like [KubicArk](https://github.com/Cervator/KubicArk/) and [KubicTerasology](https://github.com/Cervator/KubicTerasology/)

It intends to hold any common base and template files, along with global utilities like a shared file server in case such a thing is needed (like for ARK map clustering)


## NFS support

Some games may need a shared file directory where multiple processes can read and write files to. Unfortunately the `ReadWriteMany` file access mode in Kubernetes is still mildly uncommon supported, oddly enough. Once upon a time in [GKE](https://cloud.google.com/kubernetes-engine) a lesser access mode worked enough to coincidentally support the ARK share but that was patched out in a later Kubernetes version.

So if that setup is needed for a game server we must come up with a more portable alternative: enter the NFS provisioner which will set up a Network File System on regular storage and present it to other pods with `ReadWriteMany` support.


### Installation

Roughly followed original docs from [Deployment](https://github.com/kubernetes-retired/external-storage/blob/master/nfs/docs/deployment.md) and [Usage](https://github.com/kubernetes-retired/external-storage/blob/master/nfs/docs/usage.md)

* Adjust files as needed (final versions included in this repo, including a PVC for the provisioner to use as base storage)
  * Went with provisioner name `terasology.io/nfs`
  * Went with namespace `nfs` which means the `rbac.yaml` had to list the namespace explicitly. Also had to add a couple `endpoint` verbs
  * Note that the `PodSecurityPolicy` in `psp.yaml` is likely to deprecated (maybe removed in k8s 1.22) - had to update its `apiVersion` to get it recognized for now
* `kubectl apply` the files in the namespace `nfs` (after creating it, if needed)
  * `kubectl create ns nfs`
  * `kubectl apply -f psp.yaml`
  * `kubectl apply -f rbac.yaml`
  * `kubectl apply -f base-nfs-pvc.yaml -n nfs`
  * `kubectl apply -f statefulset.yaml -n nfs`
  * `kubectl apply -f class.yaml`
* At this point you should have the needed stuff in place. Can validate real quick by applying (and then deleting) the following:
  * `kubectl apply -f claim.yaml -n nfs`
  * `kubectl apply -f write-pod.yaml -n nfs`
  * `kubectl apply -f write-pod2.yaml -n nfs`
* Try to exec to either of the write pods and check out the contents of `/mnt` - not that they'll only stay live for a few minutes. Don't forget to clean up after validating!
  
Note that the included `base-nfs-pvc.yaml` is set to `5Gi` meaning dynamic NFS storage will only allocate up to that amount. If that is later exceeded the NFS pod may enter a crash loop until more space is added (the volume can be resized on the fly). 

For actual usage in other projects simply declare a PVC with storage class `dynamic-nfs` and make sure it can be satisfied by the available space.

If the base volume runs out of space bad things may happen to the virtual volumes it hosts! One event caused a virtual volume's PVC to go back, with an associated pod using it timing out trying to mount the volume. Deleting the volume also hung, and needed [this solution](https://veducate.co.uk/kubernetes-pvc-terminating/) to clear out (in short patch the PVC with `kubectl patch pvc [PVC name] -p '{"metadata":{"finalizers":null}}'`) - but even after that the provisioner pod failed. Cleared out everything since the shared volume was just meant for backups and utility to try fresh.


## License & attribution

This project is licensed under Apache v2.0 with contributions and forks welcome.

The `nfs-provisioner` Kubernetes files came from https://github.com/kubernetes-retired/external-storage/tree/master/nfs, admittedly after its deprecation, but before its replacement at https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner was seemingly ready.

There was also a [Helm chart](https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner) setting it all up automatically, now also deprecated, and a [nice blog entry](https://medium.com/platformer-blog/nfs-persistent-volumes-with-kubernetes-a-case-study-ce1ed6e2c266) that explained the concept with a manual setup, pointing to the Helm chart and so om.
