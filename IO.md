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

´´´
lspci
´´´
The "Virtio" states, that as an example the "Ethernet controller" is a virtual IO device.

![Screenshot from 2023-04-21 11-06-33](https://user-images.githubusercontent.com/79651776/233595454-666b0a24-6e81-4da1-a711-44295c68ce6b.png)

1. Type -> Ethernet Controller
2. vendor -> Red Hat
3. description

**Synchronous vs Asynchronous Access**
*Understand the output of /proc/interrupts and discuss the columns in detail.*

Use the following to open /proc/interrupts:
´´´
vim /proc/interrupts
´´´
![Screenshot from 2023-04-21 11-11-16](https://user-images.githubusercontent.com/79651776/233596592-f1ffd7bc-7fd1-4058-ba6e-d12fd5cb0cb8.png)

1. CPU: Shows the the number of interupts per CPU
2. IRQ: unction block over which the interrupts run. 
3. Device: Name of the device

*Print the evolution of this file in real-time onto your screen. Hint, you may find the “watch” tool useful.*

´´´
watch -n 1 cat /proc/interrupts
´´´
*Verify your understanding using “lspci -vvv”, check for instance, the USB configuration. What can
you tell about the interrupt (IRQ) configuration?*

´´´
lspci -vvv
´´´
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


```
sudo dd if=/dev/zero of=testfile bs=1G count=10
```


.......

## Task 3 – Linux Device Model and UDEV
*Navigate to /sys and familiarize yourself with the structure. Look for, and understand the content of
the “uevent” file for the block device (volume) created for the IO performance task.*  

*Enter the /dev directory and look for the device file that represents the volume. What can you see
from the file attributes?*  

*UDEV uses /sys to create devices files in /dev upon the appearance (e.g. hotplugged) of a device.
List the rule files of UDEV that define this on-demand behavior. What happens if you attach a mouse
as IO input device?*  

*Remove the volume (detach it via OpenStack User Interface) created for the IO performance testing
before and look for changes in the /sys and /dev directory.*  






## Task 4 – Example IO Device: Hard Drive
iostat (see man iostat ) is a useful tool to provide low-level disk statistics.  

● Pick one hard drive of your system and display only the device utilization report in a human
readable format and using megabytes per second.
```
iostat -f directory -d  -h -m
```
● What does the output tell you?  
● How can you only show one specific device?  
● How can you show further extended details of that specific device i.e. sub-devices (hint:
/proc)?  
● Investigate in the /proc file system where disk statistics are taken by iostat for display.  
● What is the await field and why is it important?  
  The average time (in milliseconds) for I/O requests  issued to the device to be served.   
  This includes  the time spent by the requests in queue and the time spent servicing them.    
 
● If rqm/s is > 0 what does this indicate? What does it hint about the workload?  
The number of I/O requests merged per second that were queued to the device. Indicates a higher workload

*Now, measure your disk write output with dd (c.f. man dd):  
dd if=/dev/zero of=speedtest bs=10M count=100  
rm speedtest  
dd if=/dev/zero of=speedtest bs=10M count=100 conv=fdatasync*  
● How can you explain the difference in output?    
conv=fdatasync ensures, that all the data from cache has been written to the drive before the process is shown as completed. Important when device is removable and data could be lost when it is removed to early.     
● What is a fsync() operation?    
In addition to the function of fdatasync, it also transfers all the metadata information associated with the transfered files.  



