<img src="img/tux.jpg">
© 2021 Ch4r0nN 

https://github.com/Ch4r0nN/LKM-Exploration/blob/main/LICENSE[LICENSE]


# Dependencies :
## linux-headers-generic

## C compiler I recommend ```gcc or cc```

# Table of contents :

## 1 What is LKM and how it works

## 2 LKM makefile example

## 3 How modules are loaded into the kernel

## 4 LKM Hellow World

## 5 Changes

## 6 Change in the Syscalls Table in Kernel 5.7

## 7 LKM Process monitoring

# ¹ What is LKM and how it works :

#LKMs are Loadable Kernel Modules used by the Linux kernel to enarge his functionality*. May or may not be dynamically allocated; there must be no recompilation of the whole kernel. Because of those features they are often used for specific device drivers (or filesystems) such as soundcards etc.
Every LKM consist of two basic functions (*minimum*) :

```c
static int __init module_init(void)
{
    return 0;
}


static void __exit module_exit(void)
{

}
```

* ² LKM makefile example :

```c
obj-m := example.o
KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

all:
	$(MAKE) -C $(KDIR) M=$(PWD) modules

clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean

```

# ³ How modules are loaded into the kernel:

You can see the modules being loaded into the kernel with the ```lsmod``` command it checks the information in ```/proc/modules```. And there are some ways modules use to find the kernel:
 
 ## [x] - ```alias char-major-10-30 softdog```

The generic identifier *char-major-10-30* refers to the *softdog.o* module

Afterwards, modprobe looks for and examines ```/lib/modules/version/modules.dep``` to check if other modules need to be loaded before the ```softdog.o``` module. The module's dependencies are in a file created by ```depmod -a```!

# ⁴ LKM Hellow World:


```c
#ifndef _KERNEL_MODULE_
#define _KERNEL_MODULE_

#endif /* _KERNEL_MODULE_ */

#include <linux/module.h> /* included for all kernel modules       */
#include <linux/kernel.h> /* included for KERN_INFO                */
#include <linux/init.h>   /* Needed for the macros */

module_init(hello_init);
module_exit(hello_exit);

static int __init hello_init(void)
{
    printk(KERN_INFO "<1>Hello World\n");
    return 0;
}

static void __exit hello_exit(void)
{
    printk(KERN_INFO"<1> Bye bye!");
}

MODULE_AUTHOR("Ch4r0nN");
MODULE_LICENSE("MIT");
```

# ⁵ Changes :

It is undeniable that there have been changes in the LKM over all these years, some more subtle than others. Here I will list the majors and linux versions that are affected!

## 1. Kernel 2.x (Old up to 2.6)
- Initial LKM support: The 2.x series located in the base for module support, allowing dynamic loading and unloading of modules in the kernel.
- Improved OOPS and PANIC: Improved debugging tools to help developers identify and fix issues in modules.

## 2. Kernel 2.6.x
- Udev introduced: Better device management with the introduction of udev, replacing devfs.
- Preemptive Kernel: Improvements to the kernel's preemption capability, allowing it to respond more quickly to interrupts.
- Native Posix Thread Library (NPTL): Improved thread support, affecting the way modules can handle multithreaded processes.

## 3. Kernel 3.x
- Namespace support: Introduction of namespaces, allowing better isolation and support for containers (such as Docker).
- Filesystem performance: Significant performance improvements for filesystems such as Btrfs and Ext4.
- Graphics and drivers: Improved support for GPUs, including integrated and discrete graphics, impacting video driver modules.

## 4. Kernel 4.x
- Security Enhancements: Introduction of Kernel Address Space Layout Randomization (KASLR) to improve security against kernel exploits.
- Virtuozzo and better container support: Significant improvements in container support, including improvements in Cgroups and namespaces.
- Support for new hardware: Introducing support for new hardware, including new processors and storage devices.

## 5. Kernel 5.x
- Fscrypt and casefolding: Improvements in file system encryption and casefolding support in Ext4.
- Improved Live Patching: Improved live patching support, enabling security updates without reboots.
- BPF (Berkeley Packet Filter): Expansion of the use of BPF beyond networks, allowing the creation of more advanced and efficient modules.
- Support for new architectures: Better support for new architectures like RISC-V and continuous improvements for ARM.

## 5. Kernel 5.7x
- In Linux kernel 5.7, there was a significant change in the structure and visibility of the syscall table, which made it difficult for modules to find this table, especially those that used methods like kprobes.
- Prior to kernel 5.7, the syscall table ```(sys_call_table)``` was exposed in a way that allowed kernel modules to find and modify it directly. This could be used to intercept syscalls or create syscall hooks, but it also posed a security risk as it allowed malicious modules to modify the kernel's behavior.
- Starting with kernel 5.7, sys_call_table was made less accessible to improve kernel security. Specifically, the syscall table has been moved to the Read-Execute (RX) data section of the kernel, which prevents it from being modified directly. This change is part of a larger effort to protect the kernel against unauthorized modifications and exploits.

## 6. Kernel 6.x
- Performance and energy efficiency improvements: Tweaks to improve performance and reduce energy consumption, particularly on mobile devices.
- Rust in the Kernel: First experimental support for the Rust language, allowing the development of kernel modules in Rust to improve security and robustness.
- Security and isolation improvements: Continued security improvements, including better process isolation and strengthening security policies.


## ⁶ Change in the Syscalls Table in Kernel 5.7

Modules that relied on ```kprobes.h``` to find the ```sys_call_table``` were directly impacted by this change. kprobes is a powerful tool used for kernel debugging and instrumentation, allowing you to insert probes into various parts of the kernel code without having to recompile the kernel.

## With the sys_call_table moved to a read/execute-only section of memory, finding and modifying the table has become much more difficult. Specifically:
- Write Protection: The table is now in a memory section that does not allow writing, preventing direct modifications.
- Obfuscation and Hiding: The table is no longer exported directly, making it more difficult to locate using traditional methods.

## Technical details:
- 1. Protected Memory: The mapping of sys_call_table has been changed to a Write-Protected (WP) memory region. This means that any direct modification attempt will result in a segmentation fault.
   
- 2. Removal of Exported Symbols: Symbols that directly exported the location of the sys_call_table were removed or obfuscated, making it difficult to locate the table via normal kernel introspection mechanisms.

- 3. Enhanced Security: This change is part of a larger set of security improvements designed to protect the kernel from rootkits and other types of malware that attempt to modify system behavior by intercepting syscalls.

## Consequences:
- Legitimate Modules: Legitimate tools and modules that needed to intercept syscalls needed to adapt their approaches or find new ways to achieve their goals without direct access to the sys_call_table.
- Malicious Modules: Rootkits and malware that depended on modifying sys_call_table for their malicious activities were significantly harmed, improving overall system security.

## LKM process monitoring:

- LKM process monitoring is a module that can be dynamically loaded into the operating system kernel to monitor process activities. The main objective of monitoring LCM processes is to observe and, possibly, control the operations carried out by the processes using a timer, such as creating and closing processes, opening files, using the network, among others.

## Header:

```
#include <linux/module.h>
#include <linux/sched.h>
#include <linux/timer.h>
#include <linux/cred.h>
```

## 1: <linux/module.h>: Required for the basic structure of the module, including initialization and termination.
## 2: <linux/sched.h>: Required to access process information (task_struct structures) and iterate over all processes.
## 3: <linux/timer.h>: Required to configure and manipulate timers, allowing code to execute a function periodically.
## 3: <linux/cred.h>: Required for access and manipulation of process credentials (although not used explicitly in the example given).

## Timer Statement

```static struct timer_list procmonitor_timer;```

- Here, a timer is declared. procmonitor_timer will be used to run a function periodically.

## Timer Callback Function

```
static void procmonitor_check_proc_tree(unsigned long unused)
{
    int ret;
    struct task_struct *task;

    /* Traversing all tasks */
    for_each_process(task)
        printk(KERN_INFO "process: %s, PID: %d\n", task->comm, task->pid);

    /* Update the expiration time so that the callback got called again */
    ret = mod_timer(&procmonitor_timer, jiffies + msecs_to_jiffies(2000));

    if (ret)
        printk(KERN_INFO "Error when setting timer\n");
}
```

- This function is called whenever the timer expires.

** Go through all processes:

```
for_each_process(task)
    printk(KERN_INFO "process: %s, PID: %d\n", task->comm, task->pid);
```
- ``for_each_process`` is a macro that iterates through all processes in the system. For each process, print the name (comm) and process ID (pid).

## Reset the Timer:

```
ret = mod_timer(&procmonitor_timer, jiffies + msecs_to_jiffies(2000));
```

- This line resets the timer so that the function is called again after 2000 milliseconds (2 seconds). jiffies is a global variable that represents the time since the system was started, in ticks.

## Module Initialization

```
static int __init procmonitor_init(void)
{
    int ret;

    printk(KERN_INFO "Starting module.\n");

    /* Setting up our timer */
    setup_timer(&procmonitor_timer, procmonitor_check_proc_tree, 0);

    ret = mod_timer(&procmonitor_timer, jiffies + msecs_to_jiffies(200));

    if (ret) {
        printk(KERN_INFO "Error when setting timer\n");
        return -1;
    }
    return 0;
}
```


## Configure the Timer

```setup_timer(&procmonitor_timer, procmonitor_check_proc_tree, 0);```

- Initializes the procmonitor_timer timer, associating it with the procmonitor_check_proc_tree function.

## Start the Timer:

```ret = mod_timer(&procmonitor_timer, jiffies + msecs_to_jiffies(200));```

- Sets the timer to expire after 200 milliseconds. If an error occurs, it prints a message and returns -1 to indicate failure.

## Module Finalization

```
static void __exit procmonitor_exit(void)
{
    int ret;

    ret = del_timer_sync(&procmonitor_timer);

    if (ret)
        printk("Error when removing timer\n");

    printk(KERN_INFO "Cleaning up module.\n");
}
```

## Remove the Timer

ret = del_timer_sync(&procmonitor_timer);

- Removes the timer synchronously, ensuring that the callback function is not running on another CPU.


# References :

http://www.ouah.org/LKM_HACKING.html[The Hackers Choice]

https://elinux.org/Deferred_Initcalls[elinux]

https://github.com/kernelbr[Kernel Br]

https://xcellerator.github.io/posts/linux_rootkits_11/[linux kernel hacking]