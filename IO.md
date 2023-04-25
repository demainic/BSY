# Lab BSY IO
## Task 1 – IO Hardware and Architecture
Overview about the available hardware in your system
``` 
  lshw
``` 
![lshw_klein](https://user-images.githubusercontent.com/127535426/233590198-88e5f9ad-de23-4460-b0e9-ce6176cbb458.png)

``` 
  lspci
  ``` 
![grafik](https://user-images.githubusercontent.com/127535426/233587077-b9474ce3-b9e0-47b1-be70-759db509b65c.png)
``` 
  lshw bridge
``` 
![grafik](https://user-images.githubusercontent.com/127535426/233589375-4c43c59d-c5e3-4f4b-b6a6-36ea257bab28.png)


## Task 2 – Principles of IO

**Physical vs Logical (Virtual) Devices**
*Double-check your hardware configuration, can you identify logical (virtual) IO devices? If, identify
and analyse some and discuss their origin and features.*

To identfy logical IO devices use the following command

```
lspci
```
The "Virtio" states, that as an example the "Ethernet controller" is a virtual IO device.

![Screenshot from 2023-04-21 11-06-33](https://user-images.githubusercontent.com/79651776/233595454-666b0a24-6e81-4da1-a711-44295c68ce6b.png)

1. Type -> Ethernet Controller
2. vendor -> Red Hat
3. description

**Synchronous vs Asynchronous Access**
*Understand the output of /proc/interrupts and discuss the columns in detail.*

Use the following to open /proc/interrupts:
``
vim /proc/interrupts
``
![Screenshot from 2023-04-21 11-11-16](https://user-images.githubusercontent.com/79651776/233596592-f1ffd7bc-7fd1-4058-ba6e-d12fd5cb0cb8.png)

1. CPU: Shows the the number of interupts per CPU
2. IRQ: unction block over which the interrupts run. 
3. Device: Name of the device

*Print the evolution of this file in real-time onto your screen. Hint, you may find the “watch” tool useful.*

```
watch -n 1 cat /proc/interrupts
```
*Verify your understanding using “lspci -vvv”, check for instance, the USB configuration. What can
you tell about the interrupt (IRQ) configuration?*

```
lspci -vvv
```

THE IRQ is 11, it's shared between multiple devices like USB, ethernet controller or SCSI storage controller

**Shared IO Access**

*As soon as access to IO is shared, performance becomes a real issue. There are several tools to
analyze IO performance, like iostat and iotop. Familiarize yourself with these two by running them
and understanding their outputs.*

```
iostat
```

![Screenshot from 2023-04-21 11-39-33](https://user-images.githubusercontent.com/79651776/233603220-afd8ef69-eb58-4593-b43e-37aa5c637395.png)


```
iotop
```

![Screenshot from 2023-04-21 11-38-27](https://user-images.githubusercontent.com/79651776/233603006-0b712851-061a-4a0d-8e69-f18c36d5ec04.png)

*Now create a scenario in which you create IO load. Try to understand and use the following
command: “wc /dev/zero &” . Then analyze the impact via iostat and iotop for several scenarios.
Record and discuss what you notice*

use: 
```
wc /dev/zero &
```
& to run it in the background

Using iotop and iostat shows, that there is no load. The reason behind that is, it's a virtual device, there is no physical workload on an io device.


*Now understand what the “dd” tool (man dd) is for. Now run a scenario in which you run several
parallel instances of dd, each creating a file of 10G each. Before that make sure that you have
enough disk space by creating a new volume via the Openstack user interface, attaching it to your
Lab Instance, and mounting it into the filesystem, using /mnt as mountpoint. Hint, you can use
“dmesg”, “fdisk” to get info about your new virtual disk, and “mkfs.ext4” and “mount” to format and
mount it.*

The dd tool is a command-line utility for Unix-like operating systems used to convert and copy files. It can also be used for low-level operations such as copying data between devices, creating disk images, and benchmarking disk performance.


1. Create a new Volume in openstack: <br>
a. ![image](https://user-images.githubusercontent.com/79651776/234093992-90ea27f7-a5d3-4820-8ff4-81bfaefad427.png) <br>
b. ![image](https://user-images.githubusercontent.com/79651776/234094016-e3a76321-9093-4602-b0c6-893fec60c043.png) <br>
c. ![image](https://user-images.githubusercontent.com/79651776/234094155-43930e9f-9d4c-4f3e-94da-401873d8e324.png) <br>
d. ![image](https://user-images.githubusercontent.com/79651776/234094310-8fab6d3c-27ae-4817-89f5-d43d4c6d58e5.png) <br>
e. ![image](https://user-images.githubusercontent.com/79651776/234094271-8c9fcde9-e4b9-43de-9fb2-2c8b879a4300.png) <br>

2. Use the following command to get informations about the volumes 
```
dmesg
```
or 
```
lsblk
```
to show all volumes

3. Next is creating a new partition with:
```
sudo fdisk /dev/vdb
```
![image](https://user-images.githubusercontent.com/79651776/234096199-2bcc3512-7e02-4230-a5bb-a03e671b5365.png)

4. Now the new partition should be formated:
```
sudo mkfs.ext4 /dev/vdb1
```
![image](https://user-images.githubusercontent.com/79651776/234096419-7cc41c9b-2d37-4b1b-b0f2-2cc8b2b2f317.png)

5. Next it should be mounted:
```
sudo mkdir /mnt
sudo mount /dev/vdb1 /mnt
```
6. In the new partition the 10gb files can be created:

```
sudo dd if=/dev/zero of=/mnt/file1 bs=1G count=10
sudo dd if=/dev/zero of=/mnt/file2 bs=1G count=10
```

*On your new disk, create in parallel a number of files, using the following command. “dd
if=/dev/zero of=/mnt/iotest/testfile_n bs=1024 count=1000000 &” What
exactly is this command doing? What do you have to modify in order to use this command in
parallel? Perhaps you want to run this as a shell script, simply put this command line-by-line into a
text file and execute the text file (aka bash script file) on the shell. Once load is created, check IO
performance. Discuss the results and verify your understanding*


This Command creates a file named "testfile_n". The if option sets the input file, in this case alle zeros. The of sets the output file. bs = blocksize and count = number of blocks.
To Use this command first you neet to create the folder and five the ubuntu user ownership. To make it parallel you can use a loop:

```
sudo mkdir /mnt/iotest
sudo chown ubuntu:ubuntu /mnt/iotest
for n in {1..5}; do dd if=/dev/zero of=/mnt/iotest/testfile_$n bs=1024 count=1000000 & done
```
*Now experiment with the command “ionice”. Add different schedulers with different priorities to your
operations and verify the result. Can you actually run one process faster than another? If yes,
explain, if not explain too. Hint, “less /sys/block/vXXX/queue/scheduler” and
https://wiki.ubuntu.com/Kernel/Reference/IOSchedulers*

ionice is a command that allows to set the priority of a process.
```
ionice -c <class> -n <priority> <command>
```
**-c class** 
The scheduling class. 0 for none, 1 for real time, 2 for best-effort, 3 for idle.
<br>
**-n classdata** 
The scheduling class data. This defines the class data, if the class accepts an argument. For real time and best-effort, 0-7 is valid data.
<br>
**p pid** 
Pass in process PID(s) to view or change already running processes. If this argument is not given, ionice will run the listed program with the given parameters.
<br>
**-t**
Ignore failure to set requested priority. If COMMAND or PID(s) is specified, run it even in case it was not possible to set desired scheduling priority, what can happen due to insufficient privilegies or old kernel version.

(from https://linux.die.net/man/1/ionice)

*Blocking and Non-Blocking Devices
Interpret and understand the code below. What is the use case of the function select()?*
```

#include <stdio.h>, #include <stdlib.h>, #include <sys/time.h>, #include
<sys/types.h>, #include <unistd.h>
int main(void)
{
fd_set rfds;
struct timeval tv;
int retval;
/* Set rfds to STDIN (fd 0) */
FD_ZERO(&rfds); FD_SET(0, &rfds);
/* Wait up to five seconds. */
tv.tv_sec = 5;
tv.tv_usec = 0;
retval = select(1, &rfds, NULL, NULL, &tv);
if (retval == -1)
perror("select()");
else if (retval)
printf("OK");
/* FD_ISSET(0, &rfds) will be true. */
else
printf("Not OK within five seconds.\n");
exit(EXIT_SUCCESS);
}
```

The code perform a non-blocking I/O operation. It waits for 5 seconds to get userinput, without blocking the device. 
.......

## Task 3 – Linux Device Model and UDEV
*Navigate to /sys and familiarize yourself with the structure. Look for, and understand the content of
the “uevent” file for the block device (volume) created for the IO performance task.*

```
cd /sys/block/vdb/vdb1
```
```
cat uevent
```
MAJOR=252  
MINOR=17  
DEVNAME=vdb1  
DEVTYPE=partition  
PARTN=1  


MAJOR/MINOR: device identifier  
PARTN: partition index  



*Enter the /dev directory and look for the device file that represents the volume. What can you see
from the file attributes?*
```
ls -l | grep vdb
```
brw-rw---- 1 root disk    252,  16 Apr 25 15:14 vdb
brw-rw---- 1 root disk    252,  17 Apr 25 15:15 vdb1

b = block device rw = read and write, vdb1 = 1 partition  

*UDEV uses /sys to create devices files in /dev upon the appearance (e.g. hotplugged) of a device.
List the rule files of UDEV that define this on-demand behavior. What happens if you attach a mouse
as IO input device?*
 
```cd /lib/udev/rules.d/```  
```ls | grep hot```  

10-cloud-init-hook-hotplug.rules   
40-vm-hotadd.rules

 When udev is notified by the kernel of the appearance of a new device, it collects various information on the given device by consulting the corresponding entries in /sys/, especially those that uniquely identify it (MAC address for a network card, serial number for some USB devices, etc.).
Armed with all of this information, udev then consults all of the rules contained in /etc/udev/rules.d/ and /lib/udev/rules.d/. In this process it decides how to name the device, what symbolic links to create (to give it alternative names), and what commands to execute. All of these files are consulted, and the rules are all evaluated sequentially   Source: https://debian-handbook.info/browse/stable/sect.hotplug.html

In short: creates new mouse device in /dev/input  

*Remove the volume (detach it via OpenStack User Interface) created for the IO performance testing
before and look for changes in the /sys and /dev directory.*  

vdb and vdb1 are no longer visible in /dev




## Task 4 – Example IO Device: Hard Drive
iostat (see man iostat ) is a useful tool to provide low-level disk statistics.  

● *Pick one hard drive of your system and display only the device utilization report in a human
readable format and using megabytes per second.*
```
iostat -d -h -m [device]
```
● *What does the output tell you?*  
tps:        Transfers (I/O requests) per second issued to the device  
MB_read/s:  Amount of data read from the device in megabytes per second.  
MB_wrtn/s:  Amount of data written to the device in megabytes per second.    
MB_read:    The total number of megabytes read since system was booted.  
MB_wrtn:    The total number of megabytes written since system was booted.  
MB_dscd/s:  The amount of discarded requests in megabyte per second.   
MB_dscd:    The amount of discarded requests in megabyte since system was booted.

● *How can you only show one specific device?*
```
iostat -d -h -m [device]
```
● *How can you show further extended details of that specific device i.e. sub-devices (hint:
/proc)?*
```
iostat -d -h -m -p [device]
```

● *Investigate in the /proc file system where disk statistics are taken by iostat for display.*  
```
root@lab-io:/proc# cat diskstats
   7       0 loop0 1654 0 3894 9260 0 0 0 0 0 1084 7328 0 0 0 0
   7       1 loop1 15157 0 30906 13717 0 0 0 0 0 2400 8804 0 0 0 0
   7       2 loop2 69 0 2102 659 0 0 0 0 0 412 576 0 0 0 0
   7       3 loop3 2 0 10 0 0 0 0 0 0 8 0 0 0 0 0
   7       4 loop4 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
   7       5 loop5 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
   7       6 loop6 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
   7       7 loop7 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
 252       0 vda 8592 1887 685628 54594 2230 4024 2679330 20797 0 48012 62752 0 0 0 0
 252       1 vda1 7385 1887 661142 52729 2203 4016 2679248 19950 0 47232 61088 0 0 0 0
 252      14 vda14 214 0 1960 134 0 0 0 0 0 200 20 0 0 0 0
 252      15 vda15 886 0 17966 1647 2 0 2 0 0 672 776 0 0 0 0
 ```
● *What is the await field and why is it important?*  
  The average time (in milliseconds) for I/O requests  issued to the device to be served.   
  This includes  the time spent by the requests in queue and the time spent servicing them.    
 
● *If rqm/s is > 0 what does this indicate? What does it hint about the workload?*  
The number of I/O requests merged per second that were queued to the device. Indicates a higher workload

*Now, measure your disk write output with dd (c.f. man dd):  
dd if=/dev/zero of=speedtest bs=10M count=100  
rm speedtest  
dd if=/dev/zero of=speedtest bs=10M count=100 conv=fdatasync*  
● *How can you explain the difference in output?*    
conv=fdatasync ensures, that all the data from cache has been written to the drive before the process is shown as completed. Important when device is removable and data could be lost when it is removed too early.     
● *What is a fsync() operation?*    
In addition to the function of fdatasync, it also transfers all the metadata information associated with the transfered files.  



