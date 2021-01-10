# Setting up a local development server running the latest version of Debian

![debian](http://tinyimg.io/i/k9vc7m5.png)

In this article we will be setting up a headless (no desktop) debian webserver in Virtualbox or Parallels running the latest version of   Debia with shared folders and connecting via SSH to a static IP address.

## Choosing which Virtualizer to use. Virtualbox or Parallels on macOS

I've just switched from virtualbox to Parallels about 3 years after first writing my initial guide using virtual box. **Even though Parallels is paid software, I strongly recommend using it over Virtualbox**. I've always sided with using the free solution, however parallels on mac is not only more reliable extremely well polished, it cuts about 9 out of 10 of the steps.

With Virtualbox I spent many hours working with it to get SSH working, making sure the networking worked each time (sometimes it wouldn't for no apparent reason) and I actually never full once got it working exactly the way that I needed it to. Even with the shared folders set up I ran into permissions errors, and rightfully so. However with Parallels everything works out of the box as needed.

### Now the bad news about Parallels

I believe that you can only run a headless server with SSH on the upgraded version of parallels (pro or business edition). That said, if I had started with it in the beginning it would have saved me about 40 hours thus far.

**Parallels Desktop Pro Edition is $99.99/yr on a subscription** :/ or I believe $99 to out right buy that version without any free upgrades.

I'll leave it to you to decide if it's worth it, but if each of my hours was worth $3 I would have come out ahead. That said I did learn a lot about VM's and Linux by doing it with Virtual Box. But Again, never quite got the set up perfect. Just very close.

I'm going to start this guide with Parallels and cover virtualbox set up below it.

## Setting up a Folder for Virtual Machines

I use this structure to manage my VMs. Feel free to define your own.

```bash
#Top Level Folder to hold all my Virtual Machine stuff. Located in my user folder
~/ Virtual Machines
│
│   # Folder to hold out dated or no longer used virtual machines
├── - Archive
│   └── Linux Pop!_OS.pvm
│ 
│   # Folder to hold different logos and media for each virtual Machines
├── Logos
│   ├── Debian.icns
│   └── Mac.icns
│ 
│   # Folder to hold the ISOs to run the VMs off of
├── OS
│   ├── debian-10.6.0-amd64-netinst.iso
│   ├── elementaryos-5.1-stable.20200814.iso
│   ├── linuxmint-20-cinnamon-64bit.iso
│   └── pop-os_20.04_amd64_intel_13.iso
│ 
│   # Folder that will hold the a single shared folder for each virtual machine
├── Share
│   ├── Debian Server
│   └── MacOS
│
│   # Folder to Hold the Actual Virtual Machine Instances
└── VMs
    └── Debian Server.pvm
```

My `Virtual Machines` Folder is directly in my User Folder.

-   **Archive** Holds old virtual machines that I don't use anymore
-   **Logos** - Holds logos and pictures associated with each VM
-   **OS** - Holds the ISOs that I run the VMs off of
-   **Share** - Holds a general folder for each VM that will be used to share files between the host and the guest
-   **VMs** - Where the actual Virtual Machines will be installed to

Feel free to use my folder structure or make your own that makes sense to you.

## Choosing which Virtualizer to use. Virtualbox or Parallels on macOS

I've just switched from virtualbox to Parallels about 3 years after first writing my initial guide using virtual box. **Even though Parallels is paid software, I strongly recommend using it over Virtualbox**. I've always sided with using the free solution, however parallels on mac is not only more reliable extremely well polished, it cuts about 9 out of 10 of the steps.

With Virtualbox I spent many hours working with it to get SSH working, making sure the networking worked each time (sometimes it wouldn't for no apparent reason) and I actually never full once got it working exactly the way that I needed it to. Even with the shared folders set up I ran into permissions errors, and rightfully so. However with Parallels everything works out of the box as needed.

### Now the bad news about Parallels

I believe that you can only run a headless server with SSH on the upgraded version of parallels (pro or business edition). That said, if I had started with it in the beginning it would have saved me about 40 hours thus far.

**Parallels Desktop Pro Edition is $99.99/yr on a subscription** :/ or I believe $99 to out right buy that version without any free upgrades.

I'll leave it to you to decide if it's worth it, but if each of my hours was worth $3 I would have come out ahead. That said I did learn a lot about VM's and Linux by doing it with Virtual Box. But Again, never quite got the set up perfect. Just very close.

### A guide for whichever you decided to use

-   ### [Development server using Parallels](01.1a-local-development-server-with-parallels-on-mac.md)
-   ### [Development server using Virtual Box](01.1b-local-development-server-with-virtualbox.md)
