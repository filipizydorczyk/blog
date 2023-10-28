I will be honest. I don't really like virtual machines. To me, it was always a painful and unstable experience. I have never used that on a server, though. Only times I use virtualization is during the development and usually I create `docker-compose.yaml` file to set up services given app is using, and I start them for development. Recently I had a bit of different situations because I couldn't use docker. The reason why is that I was developing for [yunohost](https://yunohost.org/#/) operating system. It's a Linux distribution based on Ubuntu for hosting and managing server applications. And they don't provide any docker image. I could try to create my own image based on Ubuntu, but it was too much for what I needed it for. So I just had to clench my teeth and use [Virtual Box](https://www.virtualbox.org/). But I wanted to try to avoid all the pain I had in the past using it, so I decided on two things.

1. To create VM for development I will use only CLI so that I can easily recreate it if I change my machine (it very common to me)
2. To use it in headless mode to avoid all the little annoyances that comes with Virtual Box window's capturing your keyboard and mouse

At first, I wasn't sure if that is achievable. I knew that some kind of CLI is present in Virtual Box, but I didn't really know what you can do with it. I started with a simple thing. Furthermore, I have added script to download `.iso` file that is required to create new VM. Thanks to that, every developer that joins the project can just run this script instead of looking for it. You will also get exact version other developers are using

```sh
#!/bin/bash

wget https://build.yunohost.org/yunohost-bullseye-11.0.9-amd64-stable.iso
```

When we have the `iso` we can run our script that will create a machine. First, we need to create a new VM

```sh
VBoxManage createvm --name "yunohost-phorge-1.0.0" --ostype Ubuntu_64 --register
```

You will also need some disk space to keep your vm at. Let's create virtual HDD drive

```sh
VBoxManage createhd --filename "/home/filip/VirtualBox VMs/yunohost-phorge-1.0.0/yunohost-phorge-1.0.0.vdi" --size 20480 --variant Standard
VBoxManage storagectl "yunohost-phorge-1.0.0" --name "SATA Controller" --add sata --bootable on
VBoxManage storageattach "yunohost-phorge-1.0.0" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "/home/filip/VirtualBox VMs/yunohost-phorge-1.0.0/yunohost-phorge-1.0.0.vdi"
```

I use absolute paths here, but you can surely use relative paths so that it's working on other machines as well. Keep in mind 2 things.

1. You can choose where `.vdi` file will be created. Virtual Box with the UI will use `~/VirtualBox VMs/<vm>/<vm>.vdi` by default, so I recommend following this approach to keep things clean.
2. Make sure you provide the same path for `.vdi` file when you are attaching the storage with `VBoxManage storageattach` that you provided while creating this storage with `VBoxManage createhd`

Then we need to "insert" our `.iso` file to "DVD drive" so that we can run OS installer on first lunch. If you ever worked with VMS before, you are familiar with choosing `.iso` with the UI. This is how it's done with CLI


```sh
VBoxManage storagectl "yunohost-phorge-1.0.0" --name "IDE Controller" --add ide
VBoxManage storageattach "yunohost-phorge-1.0.0" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium "/home/filip/Development/phorge_ynh/yunohost-bullseye-11.0.9-amd64-stable.iso"
```

The machine is ready for the first boot. But you can modify its settings even further. You can also update it in the future if you need to change some settings, so it's not like you are blocked, and you need to redo the whole thing if you need an extra setting that you didn't know before. Examples:

```sh
# Set how much of your hadware you will assign to the vm
VBoxManage modifyvm "yunohost-phorge-1.0.0" --memory 2048 --vram 256 --graphicscontroller vmsvga
# Add shared folder to copy files from host machine
VBoxManage sharedfolder add "yunohost-phorge-1.0.0" --name phorge_ynh --hostpath "/home/filip/Development/phorge_ynh"
# Change network adapter to bridged (so that you can access it from host machine via ip)
VBoxManage modifyvm "yunohost-phorge-1.0.0" --nic1 bridged --nictype1 82545EM --bridgeadapter1 vmtestbr1
```

To use shared folder, you will have to mount it first in the VM with `sudo mount -t vboxsf <share-name> <mount-point>`. To change vm setting you will run `VBoxManage modifyvm "<vm-name>" --<seting value>` and you can specify multiple settings at single command.

And if you want to destroy your VM and recreate it, just run

```sh
VBoxManage unregistervm yunohost-phorge-1.0.0 --delete
```

Now if your machine will break you can just run this command and re-run the entire setup script to have fresh VM, and you will not miss a step! However, I would recommend to use to create snapshots of your machine from time to time. It's much quicker to restore, and you will keep the files if you produce any. Generally speaking it's a better way to back up your machine and writing the script is good for initial setup.

First run you will need to run with the "Normal Start" which means you will get the VM window. This window is known for being annoying with capturing your mouse and keyboard, which makes it hard to switch between host machine and vm machine. So to make it less of the pain, we are going to run it only once. You should follow with the installation and once it's done run `ip addr` to see your vm's IP. Once you have it, you can shut down the machine and run it in "Headless" mode.

This will keep the window shut and to access the VM we will use ssh

```sh
ssh <vmuser>@<vmip>
```

Now you can use the VM from the terminal, and you won't get annoyed with not being able to use your keyboard. Of course, it means that you will not get to use the desktop environment, but if you set up the machine only for development to host some app or something you probably won't need it. If you do need it, then just run it in "Normal" mode. It's the best you can do...

Sources: [thumbnail](https://ghacks.net/wp-content/uploads/2022/10/oracle-virtualbox-7.png)