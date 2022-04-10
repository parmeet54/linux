# CMPE 283 Assignments

## Assignment 1
#### by Parmeet Singh

In this assignment, we write a build kernel module that queries the VMX features present in our CPU.

The 5 features queried are:

- Pinbased Controls
- Procbased Controls
- Procbased Secondary Controls
- Exit Controls
- Entry Controls


### Questions:

1. I did the assignment by myself

2. Steps to complete the assignment

a. Setup a VM in GCP using the following commnand:

```
gcloud compute instances create cmpe283-vm --enable-nested-virtualization --zone=us-west1-b --machine-type=n2-standard-8 --network-interface=network-tier=PREMIUM,subnet=default --create-disk=auto-delete=yes,boot=yes,device-name=instance-1,image=projects/ubuntu-os-cloud/global/images/ubuntu-2004-focal-v20220204,mode=rw,size=200,type=projects/sjsu-spring-2022/zones/us-central1-a/diskTypes/pd-ssd --metadata=ssh-keys=mlarkin:"...paste ssh key here..."

```

b. Git Clone the forked linux repository:

```
git clone https://github.com/parmeet54/linux.git
```

- Upload assignment files and move to cmpe283 folder


c. Install all the dependencies and packages required for building kernel and other functions

```
    sudo apt-get update

    sudo apt install gcc

    sudo apt install make

    sudo apt-get install libncurses-dev

    sudo apt-get install libssl-dev

    sudo apt install libelf-dev

    sudo apt install make

    sudo apt-get install gcc

    sudo apt-get install flex

    sudo apt-get install bison

    sudo apt install dwarves
```

d. Make a new configuration file:


- Copy the config to a new .config file:

```   
   cp/boot/config-5.11.0-1029-gcp .config

```

- Disable the system keys to avoid build errors:

```   
    scripts/config --disable SYSTEM_TRUSTED_KEYS

    scripts/config --disable SYSTEM_REVOCATION_KEYS

```

- Renew the configuration

```
    make oldconfig
```

- Finalize the config
```
    make prepare
```


e. Build kernel

- Build all the modules
```
    make -j 8 modules
```


- Build the kernel
```
    make -j 8
```

- Package kernel and modules for booting
```
    sudo make INSTALL_MOD_STRIP=1 modules_install
```

- Install kernel
```
    Sudo make install
```

- Reboot to update bootloader to choose the correct kernel
```
    sudo reboot 
```

f. Make the assignment module and review Output

- Change Directory
```
    cd linux/cmpe283
```

- **Make** the makefile provided for assignment
```
    make
```
- We build the following files as a result

![Build Files](/cmpe283/images/assg1-files.png)

- Load the kernel object into kernel

```
    sudo insmod cmpe283-1.ko
```
- Display the output messages
```
    dmesg
```

![Output 1](/cmpe283/images/assg1-output1.png)
![Output 2](/cmpe283/images/assg1-output2.png)
