# Linux Basics

## Linux file system
### 1. /bin
Binary executable files: This directory contains essential user binaries (programs) that must be present for the system to boot and run.

### 2. /usr
User programs and data: It contains the majority of user utilities and applications, with additional subdirectories like /usr/bin for user binaries, /usr/sbin for system binaries not required for booting, /usr/local for locally compiled programs, and more.

### 3. /sbin
System binaries: Similar to /bin but contains binaries essential for booting, restoring, recovering, and/or repairing the system in addition to the binaries in /bin.

### 4. /etc
Configuration files: System-wide configuration files that affect the system's behavior for all users, such as /etc/passwd.

### 5. /dev
Device files: Device files that represent and provide access to hardware components.

### 6. /proc
Process information: A virtual filesystem providing process and kernel information as files. For example, /proc/{pid} directory contains information about the process with that particular pid.

### 7. /var
Variable data files: This includes files that are meant to change as the system runs. This includes things like logs (/var/log), packages and database storage, and cached data.

### 8. /tmp
Temporary files: Directory for storing temporary files used by system and applications.

### 9. /home
Home directories: This is a typical directory for storing user personal files and folders. Each user is typically given a directory in /home.

### 10. /boot
Boot loader files: Files needed to boot the operating system, including the Linux kernel itself and the GRUB bootloader.

### 11. /media
Removable media devices: Temporary mount directory for removable devices like USB drives.

### 12. /lib
System libraries: Shared libraries needed by applications in /bin and /sbin to boot the system.

### 13. /opt
Optional add-on applications: Additional third-party software that is not essential for the system to run.

### 14. /mnt
Mount directory for mounting filesystems: Temporarily mounted filesystems.

### 15. /srv
Service data: Directory for site-specific data which is served by the system.

### ~ : Home Directory
### / " Root Directory
### .. : Parent of current directory 
### . : Current directory

