# K8S cluster with Private docker registry


## Requirements
* VirtualBox
* Vagrant

## Clone the repo
```bash
git clone https://github.com/spareslant/VagrantFiles.git
cd VagrantFiles
git checkout tags/k8s_cluster_private_registry -b k8s_cluster_private_registry
cd build_cluster
```

## Deployment
Run the following command to deploy 1 master, 1 node kubernetes cluster along with private docker registry in separate VM. If you want more workers, then change the `vagrantfile` accordingly.

```bash
vagrant up master1 worker1 registry
```
Above command will setup the whole cluster. When above command is finished, run the following command to verify the cluster state.

```bash
vagrant ssh master
kubectl get nodes -o wide
```

you shall see following output:
```
NAME                      STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE          KERNEL-VERSION          CONTAINER-RUNTIME
master.virtual.machine    Ready    control-plane,master   43m   v1.21.4   10.0.0.10     <none>        CentOS Stream 8   4.18.0-277.el8.x86_64   cri-o://1.21.2
worker1.virtual.machine   Ready    <none>                 40m   v1.21.4   10.0.0.12     <none>        CentOS Stream 8   4.18.0-277.el8.x86_64   cri-o://1.21.2
```

## Adding more nodes
* If you want to add more nodes later on, you just need to add the host information in the top of `Vagrantfile` in `all_hosts` array.
* You need to generate the fresh token for new node to join the cluster, as the previous one might have been expired.
```bash
vagrant provision master --provision-with=generate_join_cmd
```
Above command will generate a new join command in a file in master node. Master node exports this file as NFS share which new node mounts it.
* Start the new node
```bash
vagrant up worker3
```
If you had started the new node prior to generating the join command, then new node may not join the cluster. In this case then generate the token on `master` node by running `vagrant provision master --provision-with=generate_join_cmd` command. and then run `vagrant up worker --provision`

## Verify private docker registry
* Pull an image (nginx) from internet (docker HUB)
```bash
vagrant ssh registry
sudo su -
docker pull nginx
```

* Push this image to the private registry
```bash
docker tag nginx registry.virtual.machine:5000/mynginx
docker push registry.virtual.machine:5000/mynginx
```

* Deploy this image in K8S cluster now
```bash
vagrant ssh master
kubectl run myngnix --image=registry.virtual.machine:5000/mynginx
```

* Verify deployed pod
```bash
$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
myngnix   1/1     Running   0          9s
```

## Notes
* `master` node exposes a NFS share, so it should be brought up first. `vagrant master worker1 registry` will automatically bring up VMs in specified order (left to right)
* you can run `vagrant up master --provision` or `vagrant up worker1 --provision` as many times as you like without any harm and in any order.
* You can create worker node without the existence of master node (`vagrant up worker2`) even, ofcourse it will not join the cluster then. You can create `master` node afterwards and then run command `vagrant up worker2 --provision` to join the cluster.
* GitHUB repo: https://github.com/spareslant/VagrantFiles.git (Checkout Tag: k8s_cluster_private_registry)
