# 42KL-Born2beRoot

In this project I am using:
1. Apple M1 MacOs Ventura 13.0.1
2. UTM 4.0.9
3. Debian 11.6.0 amd64-netinst

###Virtual Machine
A Virtual Machine (VM) is a compute resource that uses software instead of a physical computer to run programs and deploy apps. Each virtual machine runs its own operating system and functions separately from the other VMs, even when they are all running on the same host. For example, you can run a virtual MacOS machine on a physical PC. 

VMs deployed to accomodate different levels of processing power needs, to run software that requires a different operating system, or to test applications in a safe, sandboxed environment. It is working through "Virtualization" technology. Virtualization uses software to simulate virtual hardware that allows 
VMs to run on a single host machine.

In this project I am using UTM since Virtial Box doesn't fully support Apple M1 chip yet. 

###Debian
Debian is a Linux-based Operating System for a wide range of devices including laptops, desktops and servers. It's easy to keep the Operating System up-to-date, whether you want to upgrade to a completely new release or just update a single package. 

In Debian-based OS distributions, the default package manager we can use is dpkg. This tool allows us to install, remove and manage programs on our operating system. However, in most cases, these programs come with a list of dependencies that must be installed for the main program to function properly. One option is to manually install these dependencies. However, APT (Advanced Package Tool), which is a tool that uses dpkg, can be used to install all the necessary dependencies when installing a program. So now we can install a useful program with a single command.

APT can work with different back-ends and fron-ends to make use of its services. One of them is apt-get, which allows us to install and remove packages. Along with apt get, there are also many tools like apt-cache to manage programs. In this case, apt get and apt-cache are used by apt. Thanks to apt we can install .deb programs easily and without worrying about dependencies. But in case we want to use a graphical interface, we will have to use aptitude. Aptitude also does better control of dependencies, allowing the user to choose between different dependencies when installing a program.

####User Management
