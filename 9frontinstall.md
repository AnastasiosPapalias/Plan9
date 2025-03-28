https://github.com/AnastasiosPapalias/Plan9/blob/main/Glenda%20Toy%20Packaging.png

# 9front Installation Guide

## Table of Contents

1. [Introduction to 9front](#introduction-to-9front)
2. [System Requirements](#system-requirements)
3. [Preparing for Installation](#preparing-for-installation)
4. [Installation Process](#installation-process)
5. [Post-Installation Configuration](#post-installation-configuration)
6. [The Rio Windowing System](#the-rio-windowing-system)
7. [The Acme Editor](#the-acme-editor)
8. [File Management](#file-management)
9. [The Plumber and Faces](#the-plumber-and-faces)
10. [Networking Configuration](#networking-configuration)
11. [Communicating with Other Systems](#communicating-with-other-systems)
12. [Documentation and Help Resources](#documentation-and-help-resources)
13. [Programming in 9front](#programming-in-9front)
14. [Installing Additional Software](#installing-additional-software)
15. [Troubleshooting](#troubleshooting)
16. [Command Reference](#command-reference)
17. [Why Use Plan 9?](#why-use-plan-9)

## Introduction to 9front

9front is a fork of the Plan 9 operating system, which was originally developed by Bell Labs as a research successor to UNIX. Like Plan 9, 9front embodies the Unix philosophy of simplicity, orthogonality, and minimalism, but takes these principles even further.

### What is 9front?

9front extends Plan 9 with additional drivers, bug fixes, and features while maintaining the core design philosophy. It provides a unique computing environment that is distinctly different from mainstream operating systems.

Key characteristics of 9front include:

1. A unified filesystem-centric approach where "everything is a file"
2. Network transparency with the 9P protocol
3. Unicode support (UTF-8) throughout the system
4. A minimalist graphical user interface
5. A distributed computing model

### Philosophy

9front, like Plan 9, follows these core principles:

1. **Simplicity:** The system prioritizes clean, simple solutions over complex ones
2. **Orthogonality:** Components are designed to be combined in various ways
3. **Transparency:** System resources are represented through the filesystem
4. **Minimalism:** Features are included only when necessary

As noted in the original Plan 9 documentation: "More UNIX than the UNIXes themselves" (Unicibus ipsis Unicior), indicating that 9front continues the early UNIX ideals of simplicity and orthogonality, often in ways that are closer to the original spirit of UNIX than modern UNIX-like systems.

## System Requirements

9front has modest hardware requirements compared to modern operating systems.

### Minimum Requirements

1. CPU: 486 or later x86 processor (32-bit)
2. RAM: 32 MB (64 MB recommended)
3. Storage: 500 MB free disk space
4. Display: VGA-compatible graphics
5. Input: PS/2 or USB keyboard and mouse (three-button mouse recommended)

### Recommended Hardware

1. CPU: Pentium II or better
2. RAM: 128 MB or more
3. Storage: 2 GB or more disk space
4. Network: Ethernet adapter (for network installation)
5. Display: SVGA-compatible graphics
6. Input: Three-button mouse (essential for the full interface experience)

### Supported Architectures

1. x86 (386/amd64)
2. ARM
3. PowerPC
4. RISC-V

### Hardware Compatibility Notes

1. 9front works best with hardware that has open documentation or was common during the early 2000s
2. Graphics support is limited to specific cards/chipsets
3. Wireless networking support is limited
4. A hardware compatibility list is available on the 9front wiki

## Preparing for Installation

Before installing 9front, you need to prepare your hardware and obtain the installation media.

### Obtaining 9front

1. Download the ISO image from [http://9front.org/download](http://9front.org/download)
2. Verify the checksums to ensure the download is intact
3. Burn the ISO to a CD/DVD or create a bootable USB drive

### Creating Installation Media

#### CD/DVD Method

1. Use your preferred disc burning software
2. Burn the ISO as an image, not as a file
3. Verify the disc after burning

#### USB Method

1. Format a USB drive with a FAT32 partition
2. Extract the ISO contents to the USB drive
3. Make the USB drive bootable using the appropriate tools

### Backing Up Existing Data

If you're installing on a system with existing data:

1. Back up all important files to external media
2. Consider creating disk images of existing partitions
3. Verify your backups before proceeding

### Planning Your Installation

1. Decide whether you want a single-boot or dual-boot setup
2. Determine your partition layout
3. Note your network configuration details if needed
4. Consider whether you need special drivers

## Installation Process

The 9front installation process is text-based and straightforward, though quite different from mainstream operating systems.

### Booting the Installation Media

1. Insert your installation media (CD/DVD or USB)
2. Configure your system to boot from this media (through BIOS/UEFI)
3. When prompted, press any key to boot from the media
4. The system will load the 9front kernel and start the installation process

### Installation Steps

1. At the terminal prompt, start the installation with:
   ```
   term% inst/start
   ```

2. Choose all the defaults when prompted

3. When asked for the disk, select your primary disk (usually sdC0):
   ```
   sdC0
   ```

4. When partitioning the disk, select mbr:
   ```
   mbr
   ```

5. After partitioning, write changes and quit:
   ```
   w
   q
   ```

6. Continue following the installer prompts for:
   - Installation type (local media or network)
   - Component selection (choose complete for a full installation)
   - User account creation and password setting
   - Network configuration (IP, gateway, DNS)

7. When installation is complete, remove the installation media and reboot

### First Boot

1. The system will boot to a login prompt
2. Enter your username and password
3. By default, the system will launch the rio windowing system
4. You're now ready to use 9front

## Post-Installation Configuration

After installing 9front, several configuration steps will help you customize your system.

### System Configuration Files

The main configuration files are located in /rc/bin/:

1. cpurc - Startup configuration for CPU servers
2. termrc - Startup configuration for terminals
3. bootrc - Basic boot configuration

User-specific configuration is in /usr/username/lib/:

1. profile - User profile settings and environment

### Setting Up User Environment

1. Edit your profile to customize your environment:
   ```
   sam $home/lib/profile
   ```

2. Add or modify environment variables

3. Change the font by modifying the "font=" line:
   ```
   font=lib/font/bit/fixed/unicode.9x15.font
   ```

4. Configure startup applications

Example profile modifications:
```
bind -a $home/bin/rc /bin
bind -a $home/bin/$cputype /bin
```

### Configuring Automatic Network Setup

To configure DHCP to run automatically at startup:
```
sam $home/lib/profile
```

Add the following line:
```
ip/ipconfig
```

### Customizing the Initial Rio Layout

Customize how windows are arranged when rio starts:
```
sam $home/bin/rc/riostart
```

Sample configuration:
```
window -r 0 0 161 117 stats -lmisce
window -r 161 0 725 117 faces -i
window -r 0 117 725 500 -scroll rc
```

### Configuring Audio

For VM installations, pick "Soundblaster 16" as the Audio Controller, then:
```
9fs 9fat
sam /n/9fat/plan9.ini
```

Add the following line:
```
audio8=type=sb16 port=0x220 irq=5 dma=5
```

### Customizing Plan9.ini

Access and edit the system configuration file:
```
9fs 9fat
sam $home/n/9fat/plan9.ini
```

## The Rio Windowing System

Rio is 9front's minimalist windowing system, offering a clean interface with minimal resource requirements.

### Starting Rio

Rio typically starts automatically after login, launched from your profile. If needed, start it manually by typing:
```
rio
```

### Basic Navigation

1. **Creating Windows:**
   - Right-click anywhere on the gray background
   - Select "New" from the menu
   - Right-click and drag to define the window size

2. **Window Management:**
   - Left-click a window to bring it to the front
   - Left-drag window edges to resize
   - Right-click the background, select "Delete", then right-click on a window to close it

3. **Scrolling:**
   - Use the scroll bar on the left side of windows
   - Left-click to scroll up, right-click to scroll down
   - Middle-drag to scroll to a specific position
   - Use the ↑ and ↓ keys for keyboard scrolling

### Text Selection and Editing

1. **Selecting Text:**
   - Click once to place the cursor
   - Double-click to select a word
   - Click and drag to select multiple characters
   - Double-click inside quotes or parentheses to select content within them

2. **Clipboard Operations:**
   - Middle-click and select "cut" to cut text
   - Middle-click and select "paste" to paste text
   - Middle-click and select "snarf" to copy text

3. **Hold Mode:**
   - Press Esc to enter/exit hold mode
   - In hold mode, text turns blue and is held for editing
   - Press Esc again to exit hold mode

### Special Keys

1. Delete: Interrupts execution (like Ctrl+C)
2. Backspace: Deletes text
3. Ctrl+A: Move to beginning of line
4. Ctrl+E: Move to end of line
5. Ctrl+U: Delete from cursor to start of line
6. Ctrl+W: Delete the previous word

### Customizing Rio Appearance

You can install alternative themes for Rio, such as rio.mono:
```
hget http://plan9.stanleylieber.com/src/rio.mono.tgz > $home/theme/rio.mono.tgz
cd theme
tar zxf rio.mono.tgz
cd rio.mono
mk install
```

To use the theme:
```
rio.mono
```

To make it permanent, edit your profile:
```
acme $home/lib/profile
```

Change the line:
```
rio -r riostart
```

To:
```
rio.mono -r riostart
```

## The Acme Editor

Acme is 9front's unique text editor and development environment, inspired by the Oberon system.

### Starting Acme

Launch Acme by typing:
```
acme
```

Or to open a specific file:
```
acme filename.txt
```

### Interface Overview

Acme divides the screen into columns of panels (windows):
1. **Main Menu:** At the top of the left column
2. **Column Menu:** Below the main menu
3. **Panel Menu:** At the top of each panel

### Mouse Button Usage

Acme heavily uses the three mouse buttons:
1. **Button 1 (Left):** Sets pointer position
2. **Button 2 (Middle):** Executes commands
3. **Button 3 (Right):** Selects and searches text

### Key Menu Items

1. **Main Menu:**
   - Newcol: Create a new column
   - Putall: Save all open files
   - Exit: Close Acme

2. **Column Menu:**
   - New: Create a new panel
   - Cut/Paste/Snarf: Text operations
   - Zerox: Duplicate the current panel
   - Delcol: Delete the column

3. **Panel Menu:**
   - Del: Close the panel
   - Put: Save the file
   - Undo: Revert changes
   - Look: Search function

### Text Editing

1. **Cursor Placement:**
   - In Acme, you don't need to click to set focus
   - The cursor follows the mouse pointer
   - Type anywhere your pointer is positioned

2. **Text Selection:**
   - Works the same as in Rio
   - Left-click and drag to select text
   - Double-click for word selection

3. **Executing Commands:**
   - Middle-click on a command name to execute it
   - Type a command after the bar (|) and middle-click to run it
   - Right-click on a word to search for it

4. **File Navigation:**
   - Right-click on directories to browse them
   - Right-click on filenames to open them

### Search and Replace

To search:
1. Type "Look" followed by a search term
2. Middle-click on "Look" and the term

For search and replace:
1. Type `Edit s/oldtext/newtext/g`
2. Middle-click on the command

For global search and replace:
1. Type `Edit ,s/oldtext/newtext/g`
2. Middle-click on the command

## File Management

9front's file management model is unique, centered around the concept that "everything is a file."

### File System Structure

The main directories include:
1. / - Root directory
2. /bin - Executable programs
3. /dev - Device files
4. /env - Environment variables
5. /lib - Libraries and shared data
6. /mnt - Mount points
7. /net - Network interfaces
8. /proc - Process information
9. /srv - Service connection points
10. /sys - System files and source code
11. /tmp - Temporary files
12. /usr - User home directories

### Basic File Operations

1. **Listing Files:**
   ```
   ls -l directory
   lc directory # Columnated listing
   ```

2. **Copying Files:**
   ```
   cp source destination
   ```

3. **Moving/Renaming Files:**
   ```
   mv source destination
   ```

4. **Removing Files:**
   ```
   rm filename
   ```

5. **Creating Directories:**
   ```
   mkdir directoryname
   ```

### File Permissions

9front's permission system is simpler than Unix:
1. Use chmod to change permissions
2. Basic permissions are +rwx (read, write, execute)

Example:
```
chmod +rw filename
```

### Mount Points and Namespaces

One of 9front's distinguishing features is its namespace management:

1. **Binding Directories:**
   ```
   bind source target
   ```

2. **Mounting Filesystems:**
   ```
   mount /srv/fsname /mnt/mountpoint
   ```

3. **Creating Union Directories:**
   ```
   bind -a source target
   ```

## The Plumber and Faces

The Plumber is 9front's inter-process communication system, while Faces monitors email and network activity.

### Understanding the Plumber

The Plumber routes messages between programs based on content:
1. Files open in appropriate applications
2. URLs open in web browsers
3. Email addresses open mail clients

### Configuring the Plumber

1. The plumber configuration file is located at:
   ```
   /usr/yourusername/lib/plumbing
   ```

2. Edit this file to add custom rules:
   ```
   sam /usr/yourusername/lib/plumbing
   ```

3. Restart the plumber after making changes:
   ```
   cat /usr/yourusername/lib/plumbing >/mnt/plumb/rules
   ```

### Example Plumb Rules

Example rule to open PDF files with page:
```
type is text
data matches '([a-zA-Z0-9_\-./]+)\.pdf'
plumb to postscript
plumb client page
arg '/usr/yourusername/$0'
```

### Using Faces

Faces displays active network connections and monitors email:

1. Start Faces:
   ```
   faces -i
   ```

2. The `-i` flag shows all network connections

3. Click on a face to interact with that connection

### Customizing Faces

1. Create a custom faces directory:
   ```
   mkdir -p $home/lib/face
   ```

2. Add your own face images (48x48 pixels) with appropriate names
   
3. Set your personal face:
   ```
   cp yourface.gif $home/lib/face/48x48x8
   ```

4. Create domain-specific faces in subdirectories:
   ```
   mkdir -p $home/lib/face/domain.com
   cp domainface.gif $home/lib/face/domain.com/48x48x8
   ```

### Integrating Plumber and Faces

1. Click on a face in the Faces display to plumb that entity

2. Configure plumb rules for handling specific network resources

## Networking Configuration

Setting up and managing network connections in 9front.

### Basic Network Configuration

1. For DHCP configuration, simply run:
   ```
   ip/ipconfig
   ```

2. Test connectivity:
   ```
   ip/ping 8.8.8.8
   ```

3. For static IP configuration, edit `/lib/ndb/local`:
   ```
   sam /lib/ndb/local
   ```

4. Add configuration like:
   ```
   ip=192.168.1.2 ipmask=255.255.255.0 ipgw=192.168.1.1
   dns=8.8.8.8
   ```

### Network Commands

1. **Viewing Network Status:**
   ```
   ip/ipconfig
   cat /net/ipifc/0/status
   ```

2. **Testing Connectivity:**
   ```
   ip/ping destination
   ```

3. **DNS Lookup:**
   ```
   ndb/dnsquery hostname
   ```

### Setting Up Services

1. **Web Server:** Configure and start with:
   ```
   ip/httpd/httpd
   ```

2. **SSH-like Service (drawterm):** Configure authentication in `/lib/ndb/auth`

### File Sharing with 9P

9front uses the 9P protocol for file sharing:

1. Export a filesystem:
   ```
   srv -n servicename
   ```

2. Import a remote filesystem:
   ```
   9fs remotesystem
   mount /srv/remotesystem /mnt/mountpoint
   ```

## Communicating with Other Systems

Methods for interacting between 9front and other operating systems.

### Using Drawterm

Drawterm is a terminal emulator for accessing 9front from other systems:

1. **Installation:**
   - Download drawterm for your platform from [http://drawterm.9front.org](http://drawterm.9front.org/)
   - Install according to platform instructions

2. **Connecting:**
   ```
   drawterm -c cpuserver -a authserver
   ```

3. **File Transfer:**
   - Local PC's files appear in /mnt/term
   - Copy files directly using regular file operations

### Web Browsing

9front includes browsers for accessing the web:

1. **Mothra:** Simple graphical browser
   ```
   mothra url
   ```

2. **Abaco:** Another browser option
   ```
   abaco url
   ```

Sample URLs to explore:
- https://www.youtube.com/watch?v=2XgXDyZa8Jw
- https://www.youtube.com/watch?v=Jd3JDzTa5oA (Plan 9 Games)

### Email

Configure and use the built-in mail client:

1. Edit `/mail/lib/mailname` to set your email address
2. Use `mail` to read and send messages
3. Configure SMTP in `/mail/lib/remotemail`

### Printing

Set up printing to external printers:

1. Configure `/sys/lib/lp/devices`
2. Use `lp` command to print files

## Documentation and Help Resources

How to access documentation and get help with 9front.

### Built-in Documentation

1. **Man Pages:**
   ```
   man topic
   ```

2. **Formatted Documentation:**
   ```
   man -P topic # View as PostScript
   ```

3. **Viewing Documents:**
   ```
   page document.pdf # View PDF files
   ```

### Online Resources

1. Official 9front website: [http://9front.org](http://9front.org/)
2. 9front FAQ: [http://fqa.9front.org](http://fqa.9front.org/)
3. Wiki: [http://wiki.9front.org](http://wiki.9front.org/)
4. Mailing lists and IRC channels

### Creating Documentation

9front includes troff for document creation:

1. Create a .ms file with troff markup
2. Process with:
   ```
   troff -ms -mpictures myfile.ms | dpost -f | ps2pdf > myfile.pdf
   ```

## Programming in 9front

Development tools and programming in the 9front environment.

### The C Programming Environment

9front includes its own C compiler and development tools:

1. **Compiler Names:**
   - 8c - 386 compiler
   - 6c - AMD64 compiler
   - 5c - ARM compiler
   - 8l, 6l, 5l - Corresponding linkers

2. **Sample Program:**
   ```c
   #include <u.h>
   #include <libc.h>
   
   void
   main(void)
   {
       print("Hello, 9front!\n");
       exits(0);
   }
   ```

3. **Compilation Process:**
   ```
   8c myprog.c # Compile to object file
   8l -o myprog myprog.8 # Link to executable
   myprog # Run the program
   ```

### Other Programming Languages

9front supports several programming languages:

1. **rc** - The shell scripting language
2. **mk** - Build tool similar to make
3. **awk** - Text processing language
4. **Go** - Available through additional installation

### Using Version Control

The built-in version control system:
```
cd /sys/src
con
hg pull -u
```

### Debugging

Tools for debugging programs:
```
acid program # Debugger
```

## Installing Additional Software

### Installing Go

1. Clone the repository:
   ```
   cd $home
   hg clone -r release https://code.google.com/p/go
   ```

2. Edit your profile to add Go to your path:
   ```
   sam $home/lib/profile
   ```

3. Add the following line after the binds:
   ```
   bind -a $home/go/bin /bin
   ```

### Building & Installing the AMD64 Kernel

1. Prepare the system:
   ```
   cd /
   rc /sys/lib/rootstub
   cd /sys/src
   objtype=amd64 mk install
   cd /sys/src/9/pc64
   mk install
   ```

2. Install the new bootloader:
   ```
   9fs 9fat
   rm /n/9fat/9bootfat # Required, cannot copy over existing bootloader
   cp /386/9bootfat /n/9fat/
   chmod +al /n/9fat/9bootfat # defrag magic
   ```

3. Copy the kernel:
   ```
   cp /amd64/9pc64 /n/9fat/
   ```

4. Edit the plan9.ini configuration:
   ```
   sam /n/9fat/plan9.ini
   ```

5. Set the bootfile variable to use the 9pc64 kernel:
   ```
   bootfile=9pc64
   ```

6. Reboot the system.

## Troubleshooting

### Boot Problems

1. **Cannot Boot:**
   - Check boot configuration in 9fat partition
   - Try booting with the installation media
   - Use bootfs to repair the boot configuration

2. **Kernel Panics:**
   - Note the error message
   - Try booting with different kernel parameters
   - Check hardware compatibility

### System Crashes

1. **Rio Crashes:**
   - Switch to a text console with Alt+[n]
   - Kill processes with `kill pid`
   - Restart rio with `rio`

2. **Application Crashes:** 
   - Use the Delete key to interrupt stuck processes

### Network Issues

1. **No Network Connectivity:**
   - Check physical connections
   - Verify `/lib/ndb/local` configuration
   - Test with `ip/ping`

2. **DNS Problems:**
   - Check DNS server configuration
   - Restart the DNS resolver with `svc/dns -r`
   - Test with `ndb/dnsquery`

### Filesystem Problems

1. **Cannot Access Files:**
   - Check permissions
   - Verify mount points
   - Use `fs/check` to check filesystem integrity

### Getting Help

When all else fails:
1. Check the documentation
2. Visit the 9front website
3. Join the mailing list or IRC channel
4. Boot with the installation media and attempt repairs

## Command Reference

Common commands in 9front and their mainstream OS equivalents:

| 9front Command | Linux Equivalent | Description |
|----------------|------------------|-------------|
| ip/ping        | ping             | Test network connectivity |
| ip/ipconfig    | ipconfig/ifconfig| Configure network interfaces |
| DEL (key)      | Ctrl+C           | Interrupt running programs |
| fshalt -r      | reboot           | Restart the system |
| hget           | wget             | Download files from web |
| faces          | N/A              | Monitor network connections |
| acme           | N/A              | Text editor & development environment |
| sam            | vim/emacs        | Text editor |
| rio            | N/A              | Window manager |
| 9fs            | mount            | Mount remote filesystems |

## Why Use Plan 9?

Plan 9 offers only a few advantages to single workstation users running in isolation. Its advantages grow rapidly as the number of networked Plan 9 workstations increase. If you are developing a large distributed computing application, using Plan 9 makes a lot of sense.

If, for instance, you are performing large-scale scientific computing that needs to run across a large number of computers you are faced with a variety of difficult challenges. A particular problem in large node computing is that the failure of a single node can bring your whole computing cluster to a halt. This problem is increasingly likely as the number of processors increase.

Consider that a computer node with a mean time between failure of 10,000 hours (or about 1.15 years), when used in a cluster of 10,000 nodes will fail on average of once an hour. In other words, your large, expensive super-computer will crash once an hour.

Plan 9 provides the basis for writing processes that can be mirrored or replicated in more efficient ways and can become fault tolerant. Without increased fault tolerance, large scale computing just doesn't scale well.
