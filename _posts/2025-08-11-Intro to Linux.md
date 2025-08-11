---
title: Intro to Linux
date: 2025-08-11 10:50:00
categories: [linux]
tags: [linux]     # TAG names should always be lowercase
---
Linux, what's the deal?
Well, although I am sure most of you are familiar with windows and MacOS, many of you might not realize that Android is based off linux.
Not only android, but something like 96% of the world's top 1 million websites all run on linux..
Why is that? How did we get here?
Simple, economics and community

Unlike MacOS and Windows, Linux is open source software, which means in the most simple terms, that it is free to use, free to modify, and free to share.
That means unlike commercially available software, if anyone wants to build anything off what already exists, it is relatively easy and cheap to do so with existing software, and if something is lacking, there are enough developpers out there that have access to the tools and knowledge that is necessary to build or add whatever is missing to existing solutions.

Some of you might not know, but Linux exist since the early 90s, and the backbone of the internet was generally built up on software built on and for linux, so this history means that you can build out anything you can imagine using generally available software and languages without having to pay commercial companies any licensing fees.

The only issue with this model, is that there is no "support" in the traditional sense. Linux is BYOS, or "bring your own support"
This means you need to familizarize yourself with the software, how it's built, how it runs, and ways to work with in in order to acomplish anything using open source software. At best, you can communicate with software developeres and "maintainers" when you encounter issues with certain packages, but the entire software stack that powers Linux is a relatively complex suite of different pieces and if you encounter obscure issues, you might feel on your own when it comes to figuring out the issue.

Now, there are books, online courses, and even university courses that provide people sometimes in-depth explorations of Linux concepts, but it still takes years of familiarity with the way Linux runs to really get the grasp of it, and for most people, as light users of computers and technology, can seem overwhelming to try and understand.

This is why Windows and MacOs are still prevailent on the workstation level, as most people simply want to use a computer and not have to learn how it works. In that sense, Windows might make sense, but when you are building web infrastructure and have the ability to understand the technology, it makes more sense to build it out with DIY solutions and free software that is at the core of Linux and it is why it powers most of the world's internet infrastructure.

Seeing as Linux is now over 30 years old, it is better than ever and generally "just works" (for the most part), and at this point in time there are more open source alternatives to commercially available software than ever before.

So that being said, if you are even remotely technoloically curious or are serious about furtering your career or starting a cereer in tech, you should try it!

But oh boy, where to start?
Well, first, a few simple concepts about what makes Linux different than other operating systems...

First off, there is no single Linux "flavor"
Linux is more of a software philosophy, than a singlular software stack, because there are various "distributions", or branches of linux and although at the core they more or less work the same, there are different ways of doing things and they are generally built for different reasons and some like some flavors more than others.

There are literally thousands of linux distributions, but seeing as the software generally started from the same core, what happened is different distributions are based on different architectures and at the time there are a handful of what can be considered "base distributions", for example Red Hat Enterprise Linux (REHL), Ubuntu, Arch, and OpenSUSE.

So what differnciates these different branches of Linux? Mostly the people behind their development and the way they distribute and install software on these distributions. They also have different ways of releasing updates and depending on the environment or use case, some might be more advantaeous to work with than others.

For example, we have what are called "rolling release" distributions, like Arch, which does not have a planned release schedule. The operating system gets updates as they are made available and it is considered to be a "cutting edge" operating system as you are alway running the latest versions of system packages, generally always getting the latest updates before anyone else. But then there are Linux distributions with perhaps one or two major updates a year, like Ubuntu which has biannual releases, so system packages that are updated get some time to vet the updates before being released to most end users and delivered all at once in a major release instead of simply updating every little part as updates come out.

We can look at it from a difference in use cases, as for example on your personal computer running modern hardware used to play games or work with media, running the latest versions of software might provide an advantage, but on a webserver or service that is critical as they are under some of the world's most important websites and infrasctuture, making a major update once or twice a year might provide you with more stability and less chances of seeing breaking changes randomly throughout the year. This does make some upgrades more demanding in between major versions compared to simply alyways updating everything as it comes, but it also means there might be less maintainence and random issues throughout the year in case some minor updates break or change things that could affect the service uptime.

In Linux, "Everything is a file"

While windows registry entries to store system and software information, everything in linux is essentially a file. You better get used to the idea of managing software via config files because just about anything that you will be using and need to set up on Linux is done so using configuration files and although it is now second nature for me to dig into the documentation of open source software to figure out how to properly configure something, it might not be as intuitive for someone who's never had to do that.

Anyother major difference between windows and Linux, is the way files are structured. In Windows, your drivers are lettered ie C:/ and generally peaking all files live under the folder structure you are used to including Program Files and user files. Wheras in Linux, there is a root "/", and everything else is sorted into the followin structure based on their uses as follows:
/ (Root): The base of the entire file system hierarchy.
 It is the starting point for all file paths.
/bin: Contains essential command binaries (programs) that must be available in single-user mode for system maintenance and recovery, such as ls, cp, and bash.
/sbin: Holds essential system administration binaries, like fsck, init, and route, primarily used by the root user for system maintenance.
/etc: Stores host-specific system-wide configuration files, including settings for services, users, and network parameters.
/home: Contains individual home directories for each user, where personal files and user-specific settings are stored.
/lib: Contains essential shared libraries required by the binaries in /bin and /sbin.
/opt: Used for installing optional or third-party software packages.
/proc: A virtual filesystem that provides information about running processes and system kernel data as files.
/root: The home directory specifically for the root user.
/run: Stores run-time variable data about the system since the last boot, such as process IDs and information about logged-in users.
/srv: Contains data for services provided by the system, such as web server files or FTP data.
/tmp: A directory for temporary files created by programs, which are typically deleted upon system reboot.
/usr: A secondary hierarchy for read-only user data, containing the majority of user utilities, applications, libraries, and documentation.
 It includes subdirectories like /usr/bin for user programs, /usr/sbin for system administration programs, and /usr/lib for libraries.
/var: Holds variable data files that change during normal system operation, such as log files, spool files for printing and email, and runtime state information.
 Subdirectories include /var/log for logs and /var/lib for persistent state data.
/dev: Contains device files that represent hardware components and virtual devices, allowing software to interact with hardware.
/boot: Stores static boot files required to start the operating system, including the kernel image and the GRUB bootloader files.
/media: Used as a mount point for removable media like CDs, USB drives, and floppy disks.
/mnt: A temporary mount point for filesystems, often used by system administrators.

Now, it is nor super important to understand this to get started, but it offers an introductory look at the Linux file structure

The terminal

Before Graphical User Interfaces (GUIs), a computer was nothing but a terminal waiting for user input and providing an output based on the program run. It is a very infromation-in/information-out kind of situation, and although a GUI makes it easier to see and navigate information, under linux and the way it is built, everything can be done via terminal and it still remains one of the most common and easilest way to interact with your computer, especially when it comes to running services and making configuration changes and updates.

It may seem daunting at first, but one of the best programs I suggest you install in order to aid with using the terminal is Fish, as that provides a history and/or file based autocompletion to terminal commands, so that commands you use more often appear as suggestions when starting to type a command.
