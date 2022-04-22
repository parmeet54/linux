# CMPE 283 Assignments

NOTE: Assignments 2,3 and 4 were completed as a team (Parmeet Singh and Ashwin Ramaswamy). The Questions are below assignment 1.

## Assignment 1
#### by Parmeet Singh

**note: makefile and cmpe281-1.c were retrieved from https://github.com/mlarkin2015/linux/tree/master/cmpe283 , as by the time of the submission the files were not available through canvas.**

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
gcloud compute instances create cmpe283-vm --enable-nested-virtualization --zone=us-west1-b --machine-type=n2-standard-8 --network-interface=network-   tier=PREMIUM,subnet=default --create-disk=auto-delete=yes,boot=yes,device-name=instance-1,image=projects/ubuntu-os-cloud/global/images/ubuntu-2004-     focal-v20220204,mode=rw,size=200,type=projects/sjsu-spring-2022/zones/us-central1-a/diskTypes/pd-ssd --metadata=ssh-keys=mlarkin:"...paste ssh key here..."
```

<br/>
b. Git Clone the forked linux repository:

```
git clone https://github.com/parmeet54/linux.git
```

- Upload assignment files and move to cmpe283 folder
<br/>

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
<br/>

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
<br/>


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
<br/>

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

![Build Files](/cmpe283/images/%20assg1-files.png)

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



<br/>
<br/>

## Assignment 2
by:
Parmeet Singh, 
Ashwin Ramaswamy


### Question 1.
					
Parmeet:

- Both members configured the VM for the assignment before editing the code. I handled the building kernel and installing modules. Researched about nested virtualization and how to use it for our assignment testing. Installed all the necessary libraries such as virtint etc. to handle the inner VM. Performed testing using cpuid package.


Ashwin:
- Both of us worked together to configure the appropriate VM environment on GCP. My role in this assignment was to mainly focus on encoding cpuid.c and vmx.c to implement the correct code and ensure that the registers output the correct value based on the inputs. Primarily, this involved adding the leaf nodes of 0x4fffffff and 0x4ffffffe. Then in order to calculate the number of exits, we decided to implement a counter to calculate each call of any exit. Then, generated code to calculate the duration of exits that is called in cpuid.c
	
### Question 2:
Steps used to complete the assignment:
		
1. Made sure the VM was configured from assignment one.
- Git clone linux repo
- Fix new config file
- Build entire kernel and make modules
- Install the kernel

2. Edited the cpuid.c and vmx.c files
- Added 2 cpuid leaf nodes for 0x4FFFFFFF and 0x4FFFFFFE

3. In cpuid.c we created a u32 variable “total_exits” to store a global counter for the number of exits handled. Then we added the line “EXPORT_SYMBOL(total_exits);”

4. In vmx.c the variable “total_exits” was incremented upon each function call of __vmx_handle_exits. This allowed us to have a counter for each time any exit was called. We then used the “extern” argument to reference “total_exits” from cpuid.c.

5. Checked if %eax == 0x4fffffff, and then set %eax to total_exits, available for view.

6. Created a uint64_t variable “cycles” in cpuid.c to store the total number of cycles as the duration of an exit handle.

7. Utilized the rdtsc() function to set begin and end times when processing the duration of an exit handle. The end of __vmx_handle_exit() returns the difference of begin and end.

8. Checked if %eax == 0x4ffffffe, and then set %ebx to the low 32 bits of “cycles” and %ecx to the high 32 bits of “cycles”.

9. Installed the necessary libraries needed for Nested Virtualization
```
    sudo apt-get install qemu-kvm 
    Sudo apt-get install libvirt-bin 
    Sudo apt-get install cpu-checker
    Sudo apt-get install virtinst 
    Sudo apt-get install bridge-utils 
```

10. Download ubuntu iso image onto the main GCP VM.

11. Created an Inner VM using ​​virt-install 
```
virt-install \
--name falcon-1 \
--ram 1024 \
--disk path=/var/lib/libvirt/images/falcon1.img,size=8 \
--vcpus 1 \
--virt-type kvm \
--os-type linux \
--os-variant ubuntu20.04 \
--graphics none \
--location (---path of iso here—--) \
--extra-args "console=tty0 console=ttyS0,115200n8"
```

12. Install CPUID package for testing

13. Test cpuid leaf nodes inside inner VM using cpuid package
- Total exits using: cpuid -l 0x4FFFFFFF
- Cpu cycles using: cpuid -l 0x4FFFFFFE
