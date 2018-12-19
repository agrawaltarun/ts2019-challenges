# Generate vagrant box for laptops

Create vagrant box for laptop installation

Goal is to create ready to use (probably even w/o interenet) Vagrant image that will include preconfigured kubernetes, docker, local registry with all required docker images.

## Built With

* [microk8s](https://microk8s.io/) - Kubernetes for workstations and appliances
* [Private registry](https://microk8s.io/docs/registry) - microk8s private registry addon
* [Dockerd](https://microk8s.io/docs/dockerd) - The docker daemon used by MicroK8s

### Deploy to laptop

After completing this guide you will have laptop.box file.
This file should be distributed to computers and following steps are required to install it:

1. `vagrant box add laptop.box --name laptop`
2. `vagrant init laptop`
3. update new Vagrantfile with: 

    `config.vm.network "private_network", ip: "192.168.33.11"`
    
    Where 192.168.33.11 is IP address that does not conflict with local network where vagrnt deployed
4. `vagrant up`

System will be ready to use after completing these steps. Now user can start playing with kubernetes:

1. Login to the vagrant box `vagrant ssh laptop` 
2. run `kubectl` commads inside vagrant box
3. When VM is no longer required , adminstrator run `vagrant destroy` and deploy new VM for next user `vagrant up`


### Creating/Updating new box

Run `vagrant up` in the current directory

New vagrant box will be started and provisioned using scripts from the [provision\_scripts/](provision_scripts/) directory

Generate new vagrant box. *This box will used on laptops*:

`vagrant package --output laptop.box --vagrantfile Vagrantfile.laptop`

### Adding new docker images to the local registry

Update [provision\_scripts/create\_local\_registry.sh](provision_scripts/create_local_registry.sh)  if you want to add new docker image to the local registry 

Follow **Creating/Updating new box** to generate new box

### Testing laptop box

Add box generated using  **Creating/Updating new box**

`vagrant box add laptop.box --name laptop`

Create new vagrant project
```bash
mkdir /tmp/laptop && cd /tmp/laptop
vagrant init laptop
```

update new Vagrantfile with:
`config.vm.network "private_network", ip: "192.168.33.11"`

NOTE: **192.168.33.11** IP should not conflict with local network. Change it if necessary

k8s NodePorts will be avalable on this IP from host system

Finish deployment

```bash
vagrant up
vagrant ssh
kubectl get pods --all-namespaces
```

### Deploy k8s app/challenge

Copy/clone challange to the directory where "laptop" box started

SSH to the vagrant box

```bash
cd /tmp/laptop
vagrant ssh
```

Go to mounted $PWD

`cd /vagrant`

And you will see k8s files to apply...

**NOTE:**

Your deployment/pod should use local registry

Change container image to `localhost:32000/<image>:<version>`

Local registry should be updated as well (see **Adding new docker images to the local registry**) 

### Deploy Jenkins to test new box

To deploy [test app](test_app) 

```bash
cp -r test_app /tmp/laptop
cd /tmp/laptop
vagrant ssh
```

Now apply k8s stuff

```bash
cd /vagrant/test_app/
./start.sh
```

Open [http://192.168.33.11:31193/](http://192.168.33.11:31193/) in your browser

Cleanup
```
vagrant destroy
vagrant box remove laptop
```
