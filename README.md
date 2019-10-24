# Linux Privelege Escalation

## Basics

In a CTF challenge in which you manage to obtain a shell / arbitary command execution on a linux machine, but the action you want to perform requires higher priveleges. What do you do?

First the absolute basics:

The best way to get good at linux privesc is to understand linux, know where to look and what to look for, and you will find interesting vulnerabilities for yourself. Try to get a good understanding of what's normal for a linux device to look like and what's not. 

Here is some starting info about the linux filesystem:

```
/bin: Where Linux core commands reside like ls, mv.

/boot: Where boot loader and boot files are located.

/dev: Where all physical drives are mounted like USBs DVDs.

/etc: Contains configurations for the installed packages. (basically all programs keep their config files here)

/home: Where every user will have a personal folder to put his folders with his name like /home/user

/lib: libraries, not very interesting for hacking

/media: Here is the external devices like DVDs and USB sticks that are mounted and you can access their files from here. not often seen in CTFs.

/mnt: Where you mount other things Network locations and some distros you may find your mounted USB or DVD. Worth checking if any interesting filesystems this device has access to.

/opt: Some optional packages are located here and this is managed by the package manager. worth looking for interesting binaries

/proc: Because everything on Linux is a file, this folder for processes running on the system, not useful in most CTF chals but very interesting from an OS point of view.

/root: The home folder for the root user. needs root access.

/sbin: Like /bin, but binaries here are for root user only.

/tmp: Contains the temporary files. (memory backed, will dissapear when you reboot box, useful for saving temporary hacking / privesc scripts to)

/usr: Where the utilities and files shared between users on Linux.

/var: Contains system logs and other variable data. Also stores backups of password files in /var/backup!
```

make sure you understand how priveleges work on a linux device, more info can be found here:

Overview of linux permissions;
http://www.penguintutor.com/linux/file-permissions-reference

What sudo is:
https://www.linux.com/tutorials/linux-101-introduction-sudo/

## Interesting files/folders


**/etc/passwd**: all users and group info  
**/etc/shadow**: all hashed user passwords (needs root)  
**~/.bash_history**: command history  
**/var/log**: all logs  
**/var/backups**: backups  
**~/.ssh**: ssh keys / info  

Often CTF flags have flag in the filename, no harm in running a `locate flag`

## Stages of privesc:

### Who are you and where are you? 

`whoami`, `pwd`, `groups`

### What does this device look like?

What machine / operating system am I looking at?
`uname -a`

What other users exist?
`who -a`
`ls /home`

### What do I have access to? 

Do I have a home folder?  
`ls ~`

Do I own any files?  
```find / -user `whoami` 2> /dev/null | grep -v proc```

What does my network look like?  
`ip a; route;`

Do I have a command history?  
`ls ~/*_history`

Do I have any SSH keys?  
`ls ~/.ssh`  

### Where can I go from here?

Do I have sudo rights? `sudo -l`

Trivially exploitable files?
```
find / -perm -u=s -type f 2>/dev/null
find / -perm -g=s -type f 2>/dev/null
```

Now start using tools like GTFObins / linenum (discussed in next section)

## g0tm1lk

This is a great resource, contains a bunch of commands that are invaluable when performing linux privelege escalation, the best bit to check out CTF wise is the SUID/GUID section, but is probably the best generic collection of commands out there.
https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/

## GTFObins

GTFObins (GTFO binaries) is a project dedicated to providing a repository of programs that can be abused. They will be in the format: if you have this type of access to this executable / config file you can perform this type of priveleged action.

When you have access to a linux (for windows check out https://lolbas-project.github.io/#) machine ask:

"what programs / files are present that don't come normally installed on this type of machine?"

then look that piece of software up on GTFObins

https://gtfobins.github.io/

you might find something interesting e.g if you have sudo rights to a program you can abuse it to get a priveleged file read

## linenum

linenum is a quick tool for running the techniques I've described and much more, all you do is download the shell script, run it and examine the output. It will have loads of false positives from stuff that isn't a security risk, but it's a really good way to get loads of info at once.

https://github.com/rebootuser/LinEnum

## Other useful commands

get all users that have logged in (ignore service users)
`lastlog | grep -v Never`

If you're super desperate check out a tool called sucrack, its a local sudo password brute forcer, it's very blunt but arguably effective

## Tips and Tricks

Use piping to grep with an inverse (-v) arg to filter your outputs, e.g

```find / -user `whoami` 2> /dev/null | grep -v proc```

means find any file belonging to my user that DOES NOT have 'proc' in it (ie discard running processes). This will help you narrow down your search by filtering known false positives.

If you're looking for a certain file name no harm is doing `locate filename`, since you might find a copy of it in an interesting directory

If you know your flag starts with a prefix e.g `WTCTF{flag_here}` and you can't find it might as well `grep / WTCTF > results.txt &` and let it run

Remove bullshit from linux shadow files:
`grep -v "$6$" /etc/shadow`

## practice resources

Overthewire is great for getting to grips with linux and basic linux hacking, don't dismiss the basics, there will always be a time you need to find an interesting way to get something off a machine.

hackthebox boxes are the best place to exercise your linux privesc skills, provided you can reach that stage of each challenge, if this an area you're interested in check it out.