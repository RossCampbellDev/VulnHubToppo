# Toppo vulnhub box
created new VM using the virtual hard disk image

start kali box, start toppo box

## Flag 1
Begin with netdiscover/nmap after checking host-only connection is set up right.

`netdiscover -i eth1 -r 192.168.56.101/24` - to find what hosts are on the network

also began wireshark, listening on eth1 (the host only network)

ran nmap:  `nmap -v 192.168.56.101 -p 1-1000`
22 SSH
80 HTTP
111 rpcbind

Try UDP too:  `nmap -sU -sT -p 1-1000 192.168.56.101 -oA nmap_output`
output result to a file
68 DHCP
111 rpcbind
979 unknown!

`nc -u 192.168.56.101 979` - connected… nothing happens
try nc again on TCP port 80.  find out it’s a Debian server on 127.0.1.1

ran curl on port 80, looks like there’s a web page - some kind of blog.  tried loading a few other ports in browser, nothing.

test the SSH por with a dictionary attack using hydra

`hydra -l root -P /usr/share/worldlists/rockyou.txt 192.168.56.101 -t 8 ssh`

-l for username, -P for password list, -t thread count

Hydra took forver to find nothing.

Dirbuster:
found an ‘admin’ directory.  Inside this directory is ‘notes.txt’.  In this file was a note saying “I need to change my password ‘12345ted123’ because its outdated”.  thanks!

tried it as both ‘root’ and ‘admin’ on SSH but doesn’t work.  permission denied (publickey,password)??

trying some other directories.  in the /mail/ directory there is a PHP file that apparently asks for arguments - gonna check that!

never mind - had to figure out the old fashioned way that the username for SSH was ted

got SSH access!

not found anything.  Perhaps send etc/shadow/ to the kali machine and use John to crack?

could not ‘cat’ shadow so I used python file open and socket to send the contents to my kali machine.

`john --wordlist=/usr/share/wordlists/rockyou.txt --fork=4 shadow`

test123?

YES!! got flag 1! as well as root access

0wnedlab{p4ssi0n_c0me_with_pract1ce}

## Flag 2
not sure what to do next… looked at .bash_history in the root dir.  Found a cat, nano then another cat command.  followed by reboot.  Interesting

‘etc/issue’ says:  “IP address : \4” is this something to do with CIDR notation?

nope.  etc/issue is simply the banner that is supplied to SSH logins

pwd shows that we are in /home/ted.  we can't move to root directory

### following a guide
the person runs `find / -perm -4000 2>/dev/null`
and then upon seeing `usr/bin/mawk` goes to GTFO bins and looks up an exploit for this which grants sudo use
running this (`mawk 'BEGIN {system("/bin/sh")}'`) then changes the user to root (check with `whoami`)

change to /root, find flag

#vulnhub #exploitation #hacking #pentesting
