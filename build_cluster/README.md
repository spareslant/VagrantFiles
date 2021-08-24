## Requirements
* VirtualBox
* Vagrant

## Clone the repo
```bash
git clone https://github.com/spareslant/VagrantFiles.git
cd VagrantFiles
```

## Deployment
Run the following command to deploy 1 master and 2 node kubernetes cluster. If you want more workers, then change the `vagrantfile` accordingly.

```bash
vagrant up master1 worker1 worker2
```
Above command will setup the whole cluster. When above command is finished, run the following command to verify the cluster state.

```bash
vagrant ssh master
kubectl get nodes -o wide
```

you shall see following output:
```
NAME                      STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE          KERNEL-VERSION          CONTAINER-RUNTIME
master.virtual.machine    Ready    control-plane,master   28m   v1.21.4   10.0.0.10     <none>        CentOS Stream 8   4.18.0-277.el8.x86_64   cri-o://1.21.2
worker1.virtual.machine   Ready    <none>                 25m   v1.21.4   10.0.0.12     <none>        CentOS Stream 8   4.18.0-277.el8.x86_64   cri-o://1.21.2
worker2.virtual.machine   Ready    <none>                 22m   v1.21.4   10.0.0.14     <none>        CentOS Stream 8   4.18.0-277.el8.x86_64   cri-o://1.21.2
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

## Notes
* All provisioning scripts are idempotent.
* you can run `vagrant up master --provision` or `vagrant up worker1 --provision` as many times as you like without any harm and in any order.
* You can create worker node without the existence of master node (`vagrant up worker2`) even, ofcourse it will not join the cluster then. You can create `master` node afterwards and then run command `vagrant up worker2 --provision` to join the cluster.
* GitHUB repo: https://github.com/spareslant/VagrantFiles.git
