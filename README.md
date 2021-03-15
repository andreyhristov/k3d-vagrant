# k3d-vagrant
Run k3d in Vagrant for experimentation


## What is Vagrant
Vagrant is a tool for VM usage automation. It uses different VM backends to do its job. Typically used with VirtualBox(r) backend.

## Why K3d
K3d is a nice tool to run k3s (a distribution of K8s that is user friendly) in Docker Containers.

## Why K3d in Vagrant
Clearly the idea is to be able to automate the quick creation and disposal of a K3d clusters without tainting your host operating system. Indeed, spinning up a cluster in the host will be much quicker but OTOH you can scrap the whole project by just calling `vagrant destroy -f` and then recreate with `vagrant up`.

## How to use?
1. Change into `basebox/` directory and run `make build`. This will build the basebox. The idea behind the basebox is to set up all dependecies (Docker, K3s, k3d, container images, and other package) so you don't need to download them every time you spin off a child box. Very similar to the way base container images are used. You need to run this only once. You can re-run the command without creating problems. Once a week the basebox (Ubuntu 20.04 vagrant box) is updated so if you rebuild your basebox image you will get the updates. Update whenever you feel ok. This project is for experimentation purposes.
2. Edit Vagrantfile to adjust the CPU and RAM settings of your child box.
3. After successful basebox build, go one directory up and run `vagrant up`. A child box will be spinned up. It will take some time, depending on how fast your machine is. In the Vagrantfile a k3d cluster with name `cluster1` consisting of 3 master(control plane) nodes and 3 worker nodes is started.
4. To shell into the box, once ready, use vagrant ssh
5. If you want to ssh from outside but not by using vagrant, then you need to have a Public/Private Key pair (existing or generate one with `ssh-keygen -f ./.ssh/id_rsa`). Then uncomment two lines in Vagrantfile to enable port forwarding (in the example port 12222 on the host is opened) as well as addition of the public key to `/home/vagrant/.ssh/authorized_keys`. If the box is running then destroy it (`vagrant destroy -f`) and create it a new (`vagrant up`).

## What if I want to copy files from the host into the VM?
### Use `vagrant scp`
[Using scp and vagrant scp](https://medium.com/@smartsplash/using-scp-and-vagrant-scp-in-virtualbox-to-copy-from-guest-vm-to-host-os-and-vice-versa-9d2c828b6197). In short, run `vagrant plugin install vagrant-scp` and then copy with `vagrant scp`
### Use `sshfs`
[How to copy files from one machine to another using ssh](https://unix.stackexchange.com/questions/106480/how-to-copy-files-from-one-machine-to-another-using-ssh). For this you need to have your public key injected (change Vagrantfile) and if you are going to do it remote a remote machine and not the host machine, then enable Port Forwarding too (change Vagrantfile). In short, perform on the machine from which you want to initiate the copy operation:
1. `sudo apt-get install sshfs`  # get the package
2. `mkdir /home/user/local_mountpoint`  # creates local dir, which will be the mountpoint
3. `sshfs user@host:/home/vagrant /home/user/local_mountpoint`
4. `cp file.txt /home/user/local_mountpoint`    
5. `fusermount -u /home/user/local_mountpoint`

## Why is `openconnect` pre-installed in the VM?
Some of you might have a Docker registry behind a Cisco VPN, like a do. If you use different kind of VPN you can easily install it's client sofware

## I need to use VPN but how to get into it?
1. Your VM needs to be running (or use `vagrant up`)
2. Shell into the box (`vagrant ssh`) from a console.
3. Start a `screen` (`sreen -t vpn`). Here `vpn` is the name of the screen session. We will use it later.
4. A blank `screen` screen is started. Now execute the command(s) needed to log into your VPN. `openconnect -u xxx vpnhost.example.com`
5. Once logged in, just press Ctrl+A and then press D. This will detach you from the screen but the screen (shell) will run in background
6. To get back into the screen, execute `screen -r vpn` (vpn is the name of the session).
7. Stop your VPN connection by pressing Ctrl+C or whatever the way is
8. Press Ctrl+D to end the screen session

## Misc
1. [Operator Lifecycle Manager](https://github.com/operator-framework/operator-lifecycle-manager) is already preinstalled, so you can install operators pretty quickly


## A typical session
```
k8s@p700:~/k3d-vagrant$ cd basebox/
k8s@p700:~/k3d-vagrant/basebox$ make build 
vagrant destroy -f
==> default: VM not created. Moving on...
vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/focal64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'ubuntu/focal64' version '20210311.0.0' is up to date...
==> default: Setting the name of the VM: basebox_default_1615808240071_61758
==> default: Fixed port collision for 22 => 2222. Now on port 2200.
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2200 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2200
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: 
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default: 
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
==> default: Mounting shared folders...
    default: /vagrant => /home/k8s/k3d-vagrant/basebox
==> default: Running provisioner: shell...
    default: Running: inline script
    default: WARNING: 
    default: apt
    default:  
    default: does not have a stable CLI interface. 
    default: Use with caution in scripts.
    default: Reading package lists...
    default: Building dependency tree...
    default: Reading state information...
    default: wget is already the newest version (1.20.3-1ubuntu1).
    default: wget set to manually installed.                                                                                                                                                          
    default: apt is already the newest version (2.0.4).                                                                                                                                               
    default: apt set to manually installed.                                                                                                                                                           
    default: curl is already the newest version (7.68.0-1ubuntu2.4).                                                                                                                                  
    default: curl set to manually installed.                                                                                                                                                          
    default: 0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.                                                                                                                           
    default: deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu focal stable
    default: Warning: apt-key output should not be parsed (stdout is not a terminal)
    default: OK
    default: Get:1 https://download.docker.com/linux/ubuntu focal InRelease [36.2 kB]
    default: Get:2 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages [8458 B]
    default: Hit:4 http://archive.ubuntu.com/ubuntu focal InRelease
    default: Get:3 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [9383 B]
    default: Get:5 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 Packages [44.8 kB]
    default: Get:6 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
    default: Get:7 http://security.ubuntu.com/ubuntu focal-security InRelease [109 kB]
    default: Get:8 http://archive.ubuntu.com/ubuntu focal-backports InRelease [101 kB]
    default: Get:9 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [8628 kB]
    default: Get:10 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [550 kB]
    default: Get:11 http://security.ubuntu.com/ubuntu focal-security/universe Translation-en [80.7 kB]
    default: Get:12 http://security.ubuntu.com/ubuntu focal-security/universe amd64 c-n-f Metadata [10.6 kB]
    default: Get:13 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [14.8 kB]
    default: Get:14 http://security.ubuntu.com/ubuntu focal-security/multiverse Translation-en [3160 B]
    default: Get:15 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 c-n-f Metadata [340 B]
    default: Get:16 http://archive.ubuntu.com/ubuntu focal/universe Translation-en [5124 kB]
    default: Get:17 http://archive.ubuntu.com/ubuntu focal/universe amd64 c-n-f Metadata [265 kB]
    default: Get:18 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [144 kB]
    default: Get:19 http://archive.ubuntu.com/ubuntu focal/multiverse Translation-en [104 kB]
    default: Get:20 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 c-n-f Metadata [9136 B]
    default: Get:21 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [863 kB]
    default: Get:22 http://archive.ubuntu.com/ubuntu focal-updates/main Translation-en [204 kB]
    default: Get:23 http://archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [162 kB]
    default: Get:24 http://archive.ubuntu.com/ubuntu focal-updates/restricted Translation-en [24.2 kB]
    default: Get:25 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [750 kB]
    default: Get:26 http://archive.ubuntu.com/ubuntu focal-updates/universe Translation-en [157 kB]
    default: Get:27 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 c-n-f Metadata [16.3 kB]
    default: Get:28 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [21.6 kB]
    default: Get:29 http://archive.ubuntu.com/ubuntu focal-updates/multiverse Translation-en [5508 B]
    default: Get:30 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 c-n-f Metadata [596 B]
    default: Get:31 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 c-n-f Metadata [112 B]
    default: Get:32 http://archive.ubuntu.com/ubuntu focal-backports/restricted amd64 c-n-f Metadata [116 B]
    default: Get:33 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [4032 B]
    default: Get:34 http://archive.ubuntu.com/ubuntu focal-backports/universe Translation-en [1448 B]
    default: Get:35 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 c-n-f Metadata [224 B]
    default: Get:36 http://archive.ubuntu.com/ubuntu focal-backports/multiverse amd64 c-n-f Metadata [116 B]
    default: Fetched 17.6 MB in 8s (2176 kB/s)
    default: Reading package lists...                                                                                                                                                                 
    default: 
    default: WARNING: apt does not have a stable CLI interface. Use with caution in scripts.                                                                                                          
    default: Reading package lists...
    default: Building dependency tree...
    default: Reading state information...
    default: git is already the newest version (1:2.25.1-1ubuntu3.1).
    default: git set to manually installed.                                                                                                                                                           
    default: The following additional packages will be installed:                                                                                                                                     
    default:   docker-ce-rootless-extras libopenconnect5 libpcsclite1 libstoken1
    default:   libtomcrypt1 libtommath1 pigz slirp4netns vpnc-scripts
    default: Suggested packages:
    default:   aufs-tools cgroupfs-mount | cgroup-lite pcscd dnsmasq resolvconf
    default: The following NEW packages will be installed:
    default:   containerd.io docker-ce docker-ce-cli docker-ce-rootless-extras joe kubectl
    default:   libopenconnect5 libpcsclite1 libstoken1 libtomcrypt1 libtommath1 openconnect
    default:   pbzip2 pigz slirp4netns vpnc-scripts
    default: 0 upgraded, 16 newly installed, 0 to remove and 5 not upgraded.
    default: Need to get 113 MB of archives.                                                                                                                                                          
    default: After this operation, 498 MB of additional disk space will be used.                                                                                                                      
    default: Get:1 https://download.docker.com/linux/ubuntu focal/stable amd64 containerd.io amd64 1.4.4-1 [28.3 MB]                                                                                  
    default: Get:2 http://archive.ubuntu.com/ubuntu focal/universe amd64 pigz amd64 2.4-1 [57.4 kB]
    default: Get:4 http://archive.ubuntu.com/ubuntu focal/universe amd64 joe amd64 4.6-1build2 [509 kB]
    default: Get:5 http://archive.ubuntu.com/ubuntu focal/main amd64 libpcsclite1 amd64 1.8.26-3 [22.0 kB]
    default: Get:6 http://archive.ubuntu.com/ubuntu focal/main amd64 libtommath1 amd64 1.2.0-3 [53.0 kB]
    default: Get:7 http://archive.ubuntu.com/ubuntu focal/universe amd64 libtomcrypt1 amd64 1.18.2-3 [360 kB]
    default: Get:3 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.20.4-00 [7944 kB]
    default: Get:8 http://archive.ubuntu.com/ubuntu focal/universe amd64 libstoken1 amd64 0.92-1 [26.2 kB]
    default: Get:9 http://archive.ubuntu.com/ubuntu focal/universe amd64 libopenconnect5 amd64 8.05-1 [134 kB]
    default: Get:10 http://archive.ubuntu.com/ubuntu focal/universe amd64 vpnc-scripts all 0.1~git20190117-1 [13.8 kB]
    default: Get:11 http://archive.ubuntu.com/ubuntu focal/universe amd64 openconnect amd64 8.05-1 [464 kB]
    default: Get:12 http://archive.ubuntu.com/ubuntu focal/universe amd64 pbzip2 amd64 1.1.13-1build1 [40.0 kB]
    default: Get:13 http://archive.ubuntu.com/ubuntu focal/universe amd64 slirp4netns amd64 0.4.3-1 [74.3 kB]
    default: Get:14 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-ce-cli amd64 5:20.10.5~3-0~ubuntu-focal [41.4 MB]
    default: Get:15 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-ce amd64 5:20.10.5~3-0~ubuntu-focal [24.8 MB]
    default: Get:16 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-ce-rootless-extras amd64 5:20.10.5~3-0~ubuntu-focal [8966 kB]
    default: Fetched 113 MB in 11s (10.3 MB/s)
    default: Selecting previously unselected package pigz.
    default: (Reading database ...                                                                                                                                                                    
(Reading database ... 55%abase ... 5%
    default: (Reading database ... 60%
    default: (Reading database ... 65%
    default: (Reading database ... 70%
    default: (Reading database ... 75%
    default: (Reading database ... 80%
    default: (Reading database ... 85%
    default: (Reading database ... 90%
    default: (Reading database ... 95%
(Reading database ... 63093 files and directories currently installed.)
    default: Preparing to unpack .../00-pigz_2.4-1_amd64.deb ...
    default: Unpacking pigz (2.4-1) ...
    default: Selecting previously unselected package containerd.io.
    default: Preparing to unpack .../01-containerd.io_1.4.4-1_amd64.deb ...
    default: Unpacking containerd.io (1.4.4-1) ...
    default: Selecting previously unselected package docker-ce-cli.
    default: Preparing to unpack .../02-docker-ce-cli_5%3a20.10.5~3-0~ubuntu-focal_amd64.deb ...
    default: Unpacking docker-ce-cli (5:20.10.5~3-0~ubuntu-focal) ...
    default: Selecting previously unselected package docker-ce.
    default: Preparing to unpack .../03-docker-ce_5%3a20.10.5~3-0~ubuntu-focal_amd64.deb ...
    default: Unpacking docker-ce (5:20.10.5~3-0~ubuntu-focal) ...
    default: Selecting previously unselected package docker-ce-rootless-extras.
    default: Preparing to unpack .../04-docker-ce-rootless-extras_5%3a20.10.5~3-0~ubuntu-focal_amd64.deb ...
    default: Unpacking docker-ce-rootless-extras (5:20.10.5~3-0~ubuntu-focal) ...
    default: Selecting previously unselected package joe.
    default: Preparing to unpack .../05-joe_4.6-1build2_amd64.deb ...
    default: Unpacking joe (4.6-1build2) ...
    default: Selecting previously unselected package kubectl.
    default: Preparing to unpack .../06-kubectl_1.20.4-00_amd64.deb ...
    default: Unpacking kubectl (1.20.4-00) ...
    default: Selecting previously unselected package libpcsclite1:amd64.
    default: Preparing to unpack .../07-libpcsclite1_1.8.26-3_amd64.deb ...
    default: Unpacking libpcsclite1:amd64 (1.8.26-3) ...
    default: Selecting previously unselected package libtommath1:amd64.
    default: Preparing to unpack .../08-libtommath1_1.2.0-3_amd64.deb ...
    default: Unpacking libtommath1:amd64 (1.2.0-3) ...
    default: Selecting previously unselected package libtomcrypt1:amd64.
    default: Preparing to unpack .../09-libtomcrypt1_1.18.2-3_amd64.deb ...
    default: Unpacking libtomcrypt1:amd64 (1.18.2-3) ...
    default: Selecting previously unselected package libstoken1:amd64.
    default: Preparing to unpack .../10-libstoken1_0.92-1_amd64.deb ...
    default: Unpacking libstoken1:amd64 (0.92-1) ...
    default: Selecting previously unselected package libopenconnect5:amd64.
    default: Preparing to unpack .../11-libopenconnect5_8.05-1_amd64.deb ...
    default: Unpacking libopenconnect5:amd64 (8.05-1) ...
    default: Selecting previously unselected package vpnc-scripts.
    default: Preparing to unpack .../12-vpnc-scripts_0.1~git20190117-1_all.deb ...
    default: Unpacking vpnc-scripts (0.1~git20190117-1) ...
    default: Selecting previously unselected package openconnect.
    default: Preparing to unpack .../13-openconnect_8.05-1_amd64.deb ...
    default: Unpacking openconnect (8.05-1) ...
    default: Selecting previously unselected package pbzip2.
    default: Preparing to unpack .../14-pbzip2_1.1.13-1build1_amd64.deb ...
    default: Unpacking pbzip2 (1.1.13-1build1) ...
    default: Selecting previously unselected package slirp4netns.
    default: Preparing to unpack .../15-slirp4netns_0.4.3-1_amd64.deb ...
    default: Unpacking slirp4netns (0.4.3-1) ...
    default: Setting up joe (4.6-1build2) ...
    default: update-alternatives: 
    default: using /usr/bin/joe to provide /usr/bin/editor (editor) in auto mode
    default: Setting up slirp4netns (0.4.3-1) ...
    default: Setting up libtommath1:amd64 (1.2.0-3) ...
    default: Setting up pbzip2 (1.1.13-1build1) ...
    default: Setting up kubectl (1.20.4-00) ...
    default: Setting up containerd.io (1.4.4-1) ...
    default: Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /lib/systemd/system/containerd.service.
    default: Setting up libpcsclite1:amd64 (1.8.26-3) ...
    default: Setting up docker-ce-cli (5:20.10.5~3-0~ubuntu-focal) ...
    default: Setting up pigz (2.4-1) ...
    default: Setting up vpnc-scripts (0.1~git20190117-1) ...
    default: Setting up libtomcrypt1:amd64 (1.18.2-3) ...
    default: Setting up libstoken1:amd64 (0.92-1) ...
    default: Setting up docker-ce (5:20.10.5~3-0~ubuntu-focal) ...
    default: Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /lib/systemd/system/docker.service.
    default: Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /lib/systemd/system/docker.socket.
    default: Setting up docker-ce-rootless-extras (5:20.10.5~3-0~ubuntu-focal) ...
    default: Setting up libopenconnect5:amd64 (8.05-1) ...
    default: Setting up openconnect (8.05-1) ...
    default: Processing triggers for mime-support (3.64ubuntu1) ...
    default: Processing triggers for libc-bin (2.31-0ubuntu9.2) ...
    default: Processing triggers for systemd (245.4-4ubuntu3.4) ...
    default: Processing triggers for man-db (2.9.1-1) ...
    default: Adding user `vagrant' to group `docker' ...
    default: Adding user vagrant to group docker
    default: Done.
    default: Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install.
    default: Executing: /lib/systemd/systemd-sysv-install enable docker
    default: Preparing to install k3d into /usr/local/bin
    default: k3d installed into /usr/local/bin/k3d
    default: Run 'k3d --help' to see what you can do with it.
    default: INFO[0000] Prep: Network                                
    default: INFO[0000] Created network 'k3d-warmup-cluster'         
    default: INFO[0000] Created volume 'k3d-warmup-cluster-images'   
    default: INFO[0001] Creating node 'k3d-warmup-cluster-server-0'  
    default: INFO[0003] Pulling image 'docker.io/rancher/k3s:v1.20.4-k3s1' 
    default: INFO[0013] Creating LoadBalancer 'k3d-warmup-cluster-serverlb' 
    default: INFO[0015] Pulling image 'docker.io/rancher/k3d-proxy:v4.3.0' 
    default: INFO[0019] Starting cluster 'warmup-cluster'            
    default: INFO[0019] Starting servers...                          
    default: INFO[0019] Starting Node 'k3d-warmup-cluster-server-0'  
    default: INFO[0027] Starting agents...                           
    default: INFO[0027] Starting helpers...                          
    default: INFO[0027] Starting Node 'k3d-warmup-cluster-serverlb'  
    default: INFO[0029] (Optional) Trying to get IP of the docker host and inject it into the cluster as 'host.k3d.internal' for easy access 
    default: INFO[0033] Successfully added host record to /etc/hosts in 2/2 nodes and to the CoreDNS ConfigMap 
    default: INFO[0033] Cluster 'warmup-cluster' created successfully! 
    default: INFO[0033] --kubeconfig-update-default=false --> sets --kubeconfig-switch-context=false 
    default: INFO[0033] You can now use it like this:                
    default: kubectl config use-context k3d-warmup-cluster
    default: kubectl cluster-info
    default: INFO[0000] Deleting cluster 'warmup-cluster'            
    default: INFO[0000] Deleted k3d-warmup-cluster-serverlb          
    default: INFO[0000] Deleted k3d-warmup-cluster-server-0          
    default: INFO[0000] Deleting cluster network 'k3d-warmup-cluster' 
    default: INFO[0000] Deleting image volume 'k3d-warmup-cluster-images' 
    default: INFO[0000] Removing cluster details from default kubeconfig... 
    default: INFO[0000] Removing standalone kubeconfig file (if there is one)... 
    default: INFO[0000] Successfully deleted cluster warmup-cluster! 
    default: dd: error writing '/EMPTY': No space left on device
    default: 37594+0 records in                                                                                                                                                                       
    default: 37593+0 records out                                                                                                                                                                      
    default: 39419437056 bytes (39 GB, 37 GiB) copied, 45.2057 s, 872 MB/s                                                                                                                            
    default: dd exit code 1 is suppressed
vagrant halt
==> default: Attempting graceful shutdown of VM...
vagrant package --output basebox_k3d_focal.box
==> default: Clearing any previously set forwarded ports...
==> default: Exporting VM...
==> default: Compressing package to: /home/k8s/k3d-vagrant/basebox/basebox_k3d_focal.box
vagrant box remove basebox_k3d_focal -f ; vagrant box add basebox_k3d_focal basebox_k3d_focal.box
Removing box 'basebox_k3d_focal' (v0) with provider 'virtualbox'...
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'basebox_k3d_focal' (v0) for provider: 
    box: Unpacking necessary files from: file:///home/k8s/k3d-vagrant/basebox/basebox_k3d_focal.box
==> box: Successfully added box 'basebox_k3d_focal' (v0) for 'virtualbox'!
rm basebox_k3d_focal.box
vagrant destroy -f
==> default: Destroying VM and associated drives...
k8s@p700:~/k3d-vagrant/basebox$ less Vagrantfile ^C
k8s@p700:~/k3d-vagrant/basebox$ cd ../
k8s@p700:~/k3d-vagrant$ vagrant destroy -f
==> k3d1: VM not created. Moving on...
k8s@p700:~/k3d-vagrant$ vagrant up
Bringing machine 'k3d1' up with 'virtualbox' provider...
==> k3d1: Preparing master VM for linked clones...
    k3d1: This is a one time operation. Once the master VM is prepared,
    k3d1: it will be used as a base for linked clones, making the creation
    k3d1: of new VMs take milliseconds on a modern system.
==> k3d1: Importing base box 'basebox_k3d_focal'...
==> k3d1: Cloning VM...
==> k3d1: Matching MAC address for NAT networking...
==> k3d1: Setting the name of the VM: k3d-vagrant_k3d1_1615808969720_86221
==> k3d1: Fixed port collision for 22 => 2222. Now on port 2200.
==> k3d1: Clearing any previously set network interfaces...
==> k3d1: Preparing network interfaces based on configuration...
    k3d1: Adapter 1: nat
    k3d1: Adapter 2: hostonly
==> k3d1: Forwarding ports...
    k3d1: 22 (guest) => 2200 (host) (adapter 1)
==> k3d1: Running 'pre-boot' VM customizations...
==> k3d1: Booting VM...
==> k3d1: Waiting for machine to boot. This may take a few minutes...
    k3d1: SSH address: 127.0.0.1:2200
    k3d1: SSH username: vagrant
    k3d1: SSH auth method: private key
==> k3d1: Machine booted and ready!
==> k3d1: Checking for guest additions in VM...
==> k3d1: Setting hostname...
==> k3d1: Configuring and enabling network interfaces...
==> k3d1: Mounting shared folders...
    k3d1: /vagrant => /home/k8s/k3d-vagrant
==> k3d1: Running provisioner: shell...
    k3d1: Running: inline script
    k3d1: INFO[0000] Prep: Network                                
    k3d1: INFO[0001] Created network 'k3d-cluster1'               
    k3d1: INFO[0001] Created volume 'k3d-cluster1-images'         
    k3d1: INFO[0001] Creating initializing server node            
    k3d1: INFO[0001] Creating node 'k3d-cluster1-server-0'        
    k3d1: INFO[0002] Creating node 'k3d-cluster1-server-1'        
    k3d1: INFO[0003] Creating node 'k3d-cluster1-server-2'        
    k3d1: INFO[0003] Creating node 'k3d-cluster1-agent-0'         
    k3d1: INFO[0003] Creating LoadBalancer 'k3d-cluster1-serverlb' 
    k3d1: INFO[0003] Starting cluster 'cluster1'                  
    k3d1: INFO[0003] Starting the initializing server...          
    k3d1: INFO[0003] Starting Node 'k3d-cluster1-server-0'        
    k3d1: INFO[0006] Starting servers...                          
    k3d1: INFO[0006] Starting Node 'k3d-cluster1-server-1'        
    k3d1: INFO[0045] Starting Node 'k3d-cluster1-server-2'        
    k3d1: INFO[0059] Starting agents...                           
    k3d1: INFO[0059] Starting Node 'k3d-cluster1-agent-0'         
    k3d1: INFO[0069] Starting helpers...                          
    k3d1: INFO[0069] Starting Node 'k3d-cluster1-serverlb'        
    k3d1: INFO[0071] (Optional) Trying to get IP of the docker host and inject it into the cluster as 'host.k3d.internal' for easy access 
    k3d1: INFO[0078] Successfully added host record to /etc/hosts in 5/5 nodes and to the CoreDNS ConfigMap 
    k3d1: INFO[0078] Cluster 'cluster1' created successfully!     
    k3d1: INFO[0078] --kubeconfig-update-default=false --> sets --kubeconfig-switch-context=false 
    k3d1: INFO[0078] You can now use it like this:                
    k3d1: kubectl config use-context k3d-cluster1
    k3d1: kubectl cluster-info                                                                                                                                                                        
    k3d1: INFO[0000] Starting Node 'k3d-cluster1-agent-1-0'       
    k3d1: INFO[0001] Starting Node 'k3d-cluster1-agent-1-1'       
    k3d1: INFO[0002] Starting Node 'k3d-cluster1-agent-1-2'       
    k3d1: customresourcedefinition.apiextensions.k8s.io/catalogsources.operators.coreos.com created
    k3d1: customresourcedefinition.apiextensions.k8s.io/clusterserviceversions.operators.coreos.com created
    k3d1: customresourcedefinition.apiextensions.k8s.io/installplans.operators.coreos.com created
    k3d1: customresourcedefinition.apiextensions.k8s.io/operatorgroups.operators.coreos.com created
    k3d1: customresourcedefinition.apiextensions.k8s.io/operators.operators.coreos.com created
    k3d1: customresourcedefinition.apiextensions.k8s.io/subscriptions.operators.coreos.com created
    k3d1: namespace/olm created
    k3d1: namespace/operators created
    k3d1: serviceaccount/olm-operator-serviceaccount created
    k3d1: clusterrole.rbac.authorization.k8s.io/system:controller:operator-lifecycle-manager created
    k3d1: clusterrolebinding.rbac.authorization.k8s.io/olm-operator-binding-olm created
    k3d1: deployment.apps/olm-operator created
    k3d1: deployment.apps/catalog-operator created
    k3d1: clusterrole.rbac.authorization.k8s.io/aggregate-olm-edit created
    k3d1: clusterrole.rbac.authorization.k8s.io/aggregate-olm-view created
    k3d1: operatorgroup.operators.coreos.com/global-operators created
    k3d1: operatorgroup.operators.coreos.com/olm-operators created
    k3d1: clusterserviceversion.operators.coreos.com/packageserver created
    k3d1: catalogsource.operators.coreos.com/operatorhubio-catalog created
k8s@p700:~/k3d-vagrant$ vagrant ssh
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-66-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Mar 15 12:02:22 UTC 2021

  System load:  1.91               Users logged in:                  0
  Usage of /:   11.1% of 38.71GB   IPv4 address for br-79e68560e8dc: 172.18.0.1
  Memory usage: 56%                IPv4 address for docker0:         172.17.0.1
  Swap usage:   0%                 IPv4 address for enp0s3:          10.0.2.15
  Processes:    334                IPv4 address for enp0s8:          10.0.0.11


9 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


vagrant@k3d1:~$ kubectl get nodes
NAME                     STATUS   ROLES                       AGE   VERSION
k3d-cluster1-agent-0     Ready    <none>                      11m   v1.20.4+k3s1
k3d-cluster1-agent-1-0   Ready    <none>                      10m   v1.20.4+k3s1
k3d-cluster1-agent-1-1   Ready    <none>                      10m   v1.20.4+k3s1
k3d-cluster1-agent-1-2   Ready    <none>                      10m   v1.20.4+k3s1
k3d-cluster1-server-0    Ready    control-plane,etcd,master   12m   v1.20.4+k3s1
k3d-cluster1-server-1    Ready    control-plane,etcd,master   11m   v1.20.4+k3s1
k3d-cluster1-server-2    Ready    control-plane,etcd,master   11m   v1.20.4+k3s1

```

## Similar work
Similar work is [Vagrant K3s Cluster](https://github.com/teddy-ferdinand/vagrant-k3s-cluster) by [Teddy Ferdinand](https://github.com/teddy-ferdinand). It allows to spin up multiple boxes where per box there is one k3s node. It simulates better a real environment.



