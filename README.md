# Vagrant IaaC (Infra As A Code) on AWS - Demo

Provision a new VM via IaC, and deploy a webserver
via SCM tool
### Write Infrastructure-as-code and configuration-as-code recipes (using your preferred orchestration software) to:
- Create the server (can be AWS or Azure based)
- Configure an OS image (your choice) appropriately.
- Deploy Nginx webserver.
- Make the Nginx webserver available on port 80.
- Ensure that the server is locked down and secure.

### Provide documentation:
- Instructions for the reviewer which explain how your code should beexecuted
- Requirements for running. (Azure account? Base images? Other tooling pre-installed?)
- Explanation of assumptions and design choices.

Test can be submitted via Github, Public Repo URL

## Pre-requisites
To test this demo, you need to follow below items.

### 1. Vagrant Installation
[Download](https://www.vagrantup.com/downloads.html) and install vagrant for on your host/workstation. (Your laptop or a control server)

Refer [Vagrant Documentation](https://www.vagrantup.com/docs/installation/) as well.

### 2. Plugin Installation - vagrant-aws
Vagrant has default support for VirtualBox, Hyper-V, and Docker. If you want to create your virtual machine on any other environment (like AWS or Azure) still Vagrant has the ability to manage this but only by using  providers plugins. For this case we are using aws and we use **vagrant-aws** plugin.
You may refer [vagrant-aws](https://github.com/mitchellh/vagrant-aws) in github for the same.

```
# vagrant plugin install vagrant-aws
Installing the 'vagrant-aws' plugin. This can take a few minutes...
Installed the plugin 'vagrant-aws (0.7.2)'!
```

*If any issues during installation, then try with installing dependencies. (Depends on the workstation machine you are usuing, packages and version may change)*
```
yum -y install gcc ruby-devel rubygems compass
```

#### Notes
For other providers, you can find and install respective plugins; eg:
```
vagrant plugin install vagrant-digitalocean 
vagrant plugin install vagrant-omnibus
```

### 3. Setup Provider Environment - AWS
- Make sure you have a proper **security group** created in your VPC (under your AWS account) with SSH, HTTP/HTTPS allowed.
- Make sure you have created a **keypair** for this purpose and key file (**.pem** format) has been kept at a secure location on your machine.
- Get your **access credentials** from AWS console. ([Refer my AWI CLI installation article](https://www.techbeatly.com/2018/03/how-to-install-and-configure-aws-command-line-interface-cli.html/#how-to-get-aws-credentials)). Add the same in ```~/.aws/credentials``` file.

```
# aws configure --profile devops
AWS Access Key ID [None]: AKIAJVOHCIQ72EXAMPLE
AWS Secret Access Key [None]: 7l/j/hxXeEA77/7e+7ZvLLBQW9SxdcEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

Verify content in the files
```
# mkdir ~/.aws
# cd ~/.aws/
# cat credentials 
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[devops]
aws_access_key_id=AKIAJVOHCIQ72EXAMPLE
aws_secret_access_key=7l/j/hxXeEA77/7e+7ZvLLBQW9SxdcEXAMPLEKEY

# cat config 
[default] 
region=us-west-2 output=json 
[devops] 
region=us-west-2 output=json
```

#### Box Image 
In normal case with VirtualBox or HyberV, we need to give proper box details to load the image (like a template or clone). But in this case we are using AWS AMI (Amazon Machine Images) and config.vm.box is just for a vagrant syntax purpose. 

You can either add a dummy box(``` vagrant box add aws-dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box ```) or just use any available box image, just like what I did in Vagrantfile.

(You can choose any box by searching [here](https://app.vagrantup.com/boxes/search?provider=aws) for working with VirtualBox, Hyper-V or Docker)

## Create our Virtual Machine - Instance in AWS
Vagrant is managed inside a project directory (anywhere at your convenience, eg: your home dir) where we save Vagrantfile, other provisioning scripts (bash or ansible playbooks etc).

### Create Vagrantfile
We need to create a **Vagrantfile** where we specify what type of VM we are creating, what are the specifications needed etc. You may refer [Vagrantfile](Vagrantfile) in this project for reference. (Items are explained inside the file)

### Provisioning 
Vagrant Provisioners will help to automatically install software, update configurations etc as part of the vagrant up process. You can use any available provisioning method as Vagrant will support most of the basic told and configuration management softwares. (eg: bash, ansible, puppet, chef etc). 
Refer [Provisioning doc](https://www.vagrantup.com/docs/provisioning/)

We have used **ansible** as provisioner and create a playbook called [deploy-infra.yaml](deploy-infra.yaml) in which we have mentioned what are the configurations we need on the server (VM) once its created. (All tasks in playbook are self explanatory)

### 


On the first vagrant up that creates the environment, provisioning is run. If the environment was already created and the up is just resuming a machine or booting it up, they will not run unless the --provision flag is explicitly provided.

When vagrant provision is used on a running environment.

When vagrant reload --provision is called. The --provision flag must be present to force provisioning.



Now, we will switch to the Vagrant project directory and create VM.
```
# cd vagrant-web
# vagrant destroy
```
Wait for vagrant to create instance and provision software/configurations using ansible (we have used ansible as provision method).

#### Note 
If you have changes in provisioning configuration (when VM is up and running), you can try to apply the same by below method (without killing and starting VM again) 
```
vagrant reload --provision
```

#### Stop or Delete VM
```
vagrant destroy
```

### Troubleshooting
#### vagrant up hang at "==> default: Waiting for SSH to become available..."
This is due to wrong ssh configurations; you need to make sure
- Your .pem key has correct permission and ownership
- You have used correct Security Group (with ssh access) in Vagrantfile
- You have used correct keypair details in Vagrantfile
