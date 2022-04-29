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
- Both of us worked together to configure the appropriate nested VM environment. My role in this assignment was to mainly focus on encoding cpuid.c and vmx.c to implement the correct code and ensure that the registers output the correct value based on the inputs. Primarily, this involved adding the leaf nodes of 0x4fffffff and 0x4ffffffe. Then in order to calculate the number of exits, we decided to implement a counter to calculate each call of any exit. Then, generated code to calculate the duration of exits that is called in cpuid.c
	
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
    Sudo apt-get install libvirt-bin 
    Sudo apt-get install cpu-checker
    
    sudo apt install qemu qemu-kvm libvirt-clients libvirt-daemon-system virtinst bridge-utils

```

10. Created an Inner VM using ​​virt-install 
```
virt-install \
--name cmpe283 \
--ram 1024 \
--disk path=/var/lib/libvirt/images/cmpe283.img,size=8 \
--vcpus 1 \
--virt-type kvm \
--os-type linux \
--os-variant ubuntu18.04 \
--graphics none \
--location 'http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/' \
--extra-args "console=tty0 console=ttyS0,115200n8"
```

11. Install CPUID package for testing

12. Test cpuid leaf nodes inside inner VM using cpuid package
- Total exits using: cpuid -l 0x4FFFFFFF
- Cpu cycles using: cpuid -l 0x4FFFFFFE

<br/>



## Assignment 3
by:
Parmeet Singh, 
Ashwin Ramaswamy


### Question 1.
					
Parmeet:

- We made sure our configuration for the VM is carried over from Assignment 2 I handled the building kernel and installing modules. Researched about nested virtualization and how to use it for our assignment testing. Helped on coding as we zoomed my screen for the GCP VM. Installed all the necessary libraries such as virtint etc. to handle the inner VM. Performed testing using cpuid package.


Ashwin:
- Both of us worked together to configure the appropriate VM environment. My role in this assignment was to mainly focus on encoding cpuid.c and vmx.c to implement the correct code and ensure that the registers output the correct value based on the inputs. Primarily, this involved adding the leaf nodes of 0x4ffffffd and 0x4ffffffc. Then in order to calculate the number of exits for a particular exit, we decided to implement a counter to calculate each call of exit stored as a value in an array, indexed by the exit reason. We created a similar array to store the durations for each exit type.
	
### Question 2:
Steps used to complete the assignment:
		
1. Made sure the VM was configured from assignment two. (This step was done in assignment one, but mentioning for fulfillment purposes).
- Git clone linux repo
- Fix new config file
- Build entire kernel and make modules
- Install the kernel

2. Edited the cpuid.c and vmx.c files
- Added 2 cpuid leaf nodes for 0x4FFFFFFD and 0x4FFFFFFC

3. In cpuid.c we created a uint64_t array variable “exit_reason_counts” to store an array with the index corresponding to the value of the exit. The value stored at each index location represents the local counter for the number of times the particular exit was handled. Another array “exit_reason_times” was added in the exact same manner as the first array, but this instead stores the duration that the exit was processed. Then we added the line “EXPORT_SYMBOL();” for each additional variable added.

4. In vmx.c the the total number of exits for each particular type of exit was referenced and then was incremented upon each function call of __vmx_handle_exits. This allowed us to have a localcounter for each time any exit was called. We then used the “extern” argument to reference all defined variables from cpuid.c.

5. Checked if %eax == 0x4ffffffd, checked SDM Appendix C 'VMX basic exit reasons' for exits not defined, then set %eax to the number of exits for the particular exit reason, available for view.

6. Utilized the rdtsc() function to set begin and end times when processing the duration of an exit handle. The end of __vmx_handle_exit() returns the difference of begin and end. The resulting number of cycles was stored for the specific exit reason.

7. Checked if %eax == 0x4ffffffc. Retrieved the number of cycles duration from “exit_reason_times” and then set %ebx to the low 32 bits of “cycles” and %ecx to the high 32 bits of “cycles”.

8. Installed the necessary libraries needed for Nested Virtualization (Done in Assignment 2)

```
	Sudo apt-get install libvirt-bin 
	Sudo apt-get install cpu-checker
	sudo apt-get install qemu-kvm libvirt-bin virtinst bridge-utils cpu-checker
```

9. Created an Inner VM using ​​virt-install 

```
	sudo virt-install \
	--name cmpe283 \
	--ram 1024 \
	--disk path=/var/lib/libvirt/images/falcon1.img,size=8 \
	--vcpus 1 \
	--virt-type kvm \
	--os-type linux \
	--os-variant ubuntu18.04 \
	--graphics none \
	--location 'http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/'
	 \
	--extra-args "console=tty0 console=ttyS0,115200n8"
```

10. Install CPUID package for testing (Done in Assignment 2)

11. Test cpuid leaf nodes inside inner VM using cpuid package
- Number of exits per exit reason using: cpuid -l 0x4FFFFFFD -s [exit reason number]
- Time spent executing an exit reason using: cpuid -l 0x4FFFFFFC -s [exit reason number]

- Testing was done for the exit reasons for both the leaf nodes 0x4FFFFFFD and 0x4FFFFFFC using the cpuid package as such:
	- cpuid -l 0x4FFFFFFD -s <exit_number>
	- cpuid -l 0x4FFFFFFC -s <exit_number>


### Question 3: 
Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there more exits performed during certain VM operations? Approximately how many exits does a full VM boot entail?

ANS: The frequency of exits varies significantly based on the different types of exit. While many exit cases occur 0 times, there are certain exits that occur over 10,000 times, like External Interrupt and EPT Violation. Many exits have a relatively stable occurrence as the number of exits increases, for example, after 100,000 exits, many of the exit types have very similar number of exits. After adding up all of the exit outputs, this VM displays a total of 627,978 exits upon a full boot up.				

### Question 4: 
Of the exit types defined in the SDM, which are the most frequent? Least? 		

ANS: <br/>
Most frequent exit types:<br/>
- Exit # 1: External Interrupt<br/>
- Exit # 10: CPUID<br/>
- Exit # 28: Control Register Accesses<br/>
- Exit # 48: EPT Violation<br/>

Least Frequent exit types:<br/>
- Exit # 29: MOV DR<br/>
- Exit # 44: APIC Access<br/>
- Exit # 55: XSETBV<br/>
 
 <br/>



## Assignment 4
by:
Parmeet Singh, 
Ashwin Ramaswamy


### Question 1.

- Ashwin: Both of us were involved with all components of the lab as we collaborated over a shared screen and discussed the implementation together. For this lab, I was responsible for running the assignment 3 code and booting a test VM using KVM. After booting the inner VM, I obtained the counts of the number of exits after repeated CPUID queries to 0x4ffffffd, with different parameters as the input for %ecx.

- Parmeet: We shared my screen over Google Meets. I was responsible for operating the GCP machine we used for all the assignments, as my partner was guiding me through the instructions for the assignment such as removing and reloading the kernel modules for the different ept configurations.


### Question 2
Include a sample of your print of exit count output from dmesg from “with ept” and “without ept”.
- with EPT

![Output 1](/cmpe283/images/assg4-ept.png)

<br/>
- without EPT

![Output 2](/cmpe283/images/assg4-noept.png)

<br/>

### Question 3.
What did you learn from the count of exits? Was the count what you expected? If not, why not?

ANS: The number of exits counts significantly varied upon each boot up of the VM. As explained in HW3 Q3, many exit types yielded no occurrences, while other exit types. For certain exit types such as when accessing control registers increases the number of exits to over 50,000, while exits like “page-modification log full” always remained 0 as expected with no modifications to the page-modification log. We also learned that commonly used operations like CPUID, HLT, External Interrupt, RDTSC, VMREAD, and VMWRITE, among others occur frequently because they are used for basic VMM operations. Specific exit instructions are expectedly less frequent, as they would only occur when specific programs are running and would warrant a specific fault type.

4. What changed between the two runs (ept vs no-ept)? 

ANS:
EPT (Nested paging)
Most exit types returned 0 exits during this step. With EPT on, the host VM allows control over the CR3 register to the guest VM. In nested paging, 2 tables are used for translations, and when the second translation is not found in the VMM table, it causes a nested Page Fault. Due to this architecture, most of the “exits” do not require a proper VM exit and therefore the number of exits in our output is much lower.

No-EPT (Shadow Paging)
More exits occur with EPT = 0. This is because the guest VM has no control over the CR3 register and has to exit in order to access memory, therefore executing an “exit”.
 						

