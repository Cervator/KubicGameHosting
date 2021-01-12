# Kubic Game Hosting

The project is a top level repo for the Kubic-style game hosting projects like [KubicArk](https://github.com/Cervator/KubicArk/) and [KubicTerasology](https://github.com/Cervator/KubicTerasology/)

It intends to hold any common base and template files, along with global utilities like a shared file server in case such a thing is needed (like for ARK map clustering)


## NFS support

Some games may need a shared file directory where multiple processes can read and write files to. Unfortunately the `ReadWriteMany` file access mode in Kubernetes is still mildly uncommon supported, oddly enough. Once upon a time in [GKE](https://cloud.google.com/kubernetes-engine) a lesser access mode worked enough to coincidentally support the ARK share but that was patched out in a later Kubernetes version.

So if that setup is needed for a game server we must come up with a more portable alternative: enter the NFS provisioner which will set up a Network File System on regular storage and present it to other pods with `ReadWriteMany` support.


### Installation




## License & attribution

This project is licensed under Apache v2.0 with contributions and forks welcome.

The `nfs-provisioner` Kubernetes files came from https://github.com/kubernetes-retired/external-storage/tree/master/nfs, admittedly after its deprecation, but before its replacement at https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner was seemingly ready.

There was also a [Helm chart](https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner) setting it all up automatically, now also deprecated, and a [nice blog entry](https://medium.com/platformer-blog/nfs-persistent-volumes-with-kubernetes-a-case-study-ce1ed6e2c266) that explained the concept with a manual setup, pointing to the Helm chart and so om.
