## Linux 

To find when a system was rebooted:
```
$ last
$ last -x
$ last -x reboot
$ last reboot | head -1
$ last -x shutdown

sudo last -f /var/log/wtmp-20231201| grep boot -A 5 -B 5


$ who -b
```
https://www.cyberciti.biz/tips/linux-last-reboot-time-and-date-find-out.html

To compare all files in two dirs:
```
diff -bur folder1/ folder2/
```

## Filesystem


How to view the contents of a directory when a separate drive is mounted on top of it.
```
# Let's say /home is mounted on a different drive than the / partition is mounted to. 
# To view the contents of the old underlying /home directory contents hidden by the mount, do the following.
mkdir /mnt/root
mount --bind / /mnt/root
ls /mnt/root/home/foo/.ssh
# As long as you use --bind (as opposed to --rbind), you get a clone of the mount without the stuff mounted on top of it.
```

how to get size of a directory:
```
sudo du -sh /var

#only for the first-level subdirectories
sudo du -h --max-depth=1 /var
```

```
du -x 
du -shx /*
```
will not traverse any mount points it encounters. But if it is told to start at a mount point then it will do as requested.



How to find multiple files
```
find . -type f \( -name "*.sh" -o -name "*.txt" -o -name "*.c" \)

#how to find all files based on user/group:
find /home -user tecmint
find /home -group developer

#Find Last 50-100 Days Modified Files
find / -mtime +50 –mtime -100

#Find Modified Files in Last 1 Hour
find / -mmin -60
#Find Accessed Files in Last 1 Hour
find / -amin -60
#Find Changed Files in Last 1 Hour ( e.g. changed permissions)
find / -cmin -60

#Find Size between 50MB – 100MB
find / -size +50M -size -100M

#find files older than 15 days and remove them:
sudo find /var/log/kafka/logs/ -type f -mtime +15 -exec rm -f {} \;
```


### Find files with a text, recursively. Return only filenames.
```
grep -lr "target_centos_major_minor" *
```

```
grep -rnw '/path/to/somewhere/' -e 'pattern'
```
- -r or -R is recursive,
- -n is line number, and
- -w stands for match the whole word.
- -l (lower-case L) can be added to just give the file name of matching files.
- -e is the pattern used during the search
Along with these, --exclude, --include, --exclude-dir flags could be used for efficient searching

### rsync

For dry run:
```
rsync -anv dir1/ dir2
```

```
rsync -ah --progress source-file destination-file
```
you can reduce the network transfer by adding compression with the -z option:
```
rsync -az source destination
```


The -P flag is very helpful. It combines the flags --progress and --partial. The first of these gives you a progress bar for the transfers and the second allows you to resume interrupted transfers:

```
rsync -azP source destination
```
More [here](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories)


Example of tee + sudo
```
echo vm.max_map_count=262144 | sudo tee -a /etc/sysctl.conf
```

Delete a service:
```
service=YOUR_SERVICE_NAME; systemctl stop $service && systemctl disable $service && rm /etc/systemd/system/$service &&  systemctl daemon-reload && systemctl reset-failed
```

Revert the given unit to its vendor configuration
```
systemctl revert $service
```

To see a tree of processes:
```
ps -aef --forest
```

To find processes killed by OOM Killer
```
dmesg | egrep -i 'killed process'
```

To find biggest files:
```
sudo du -a / 2>/dev/null | sort -n -r | head -n 20

sudo find / -type f -printf "%s\t%p\n" | sort -n | tail -20

sudo find / -type f -not -path "/data*" -printf "%s\t%p\n" | sort -n | tail -20

sudo bash -c 'cat /dev/null > /var/log/lastlog'
```



To find deleted  files that are still opened:
```
find /proc/*/fd -ls | grep  '(deleted)'
lsof -nP | grep '(deleted)'

```

Trick with sudo bash -c 
```
sudo bash -c 'cat /dev/null > /var/log/lastlog'
```




Curl with Authentication: 
```
curl --user daniel:secret http://example.com/
```
Curl docs: https://everything.curl.dev/http/auth

Other
systemd's log level

Get current level
```
$ sudo busctl get-property org.freedesktop.systemd1 \
    /org/freedesktop/systemd1 org.freedesktop.systemd1.Manager LogLevel
s "info"

```
Override level
```
$ sudo systemd-analyze set-log-level debug
$ sudo busctl get-property org.freedesktop.systemd1 \
    /org/freedesktop/systemd1 org.freedesktop.systemd1.Manager LogLevel
s "debug"
```

Create a public SSH key from the private key
```
ssh-keygen -f ~/.ssh/id_rsa -y > ~/.ssh/id_rsa.pub
```

tcpdump -n port 7860


Vagrant

halt all VMs:

```
for i in `vagrant global-status | grep running|awk '{print $1 }'`; do vagrant halt -f $i; done
```

Reload VM with provisioning:
```
vagrant reload --provision
```

```
vagrant up --provider=vmware_desktop
```


Postgres

```
psql -U postgres -c 'show config_file'
```

Docker

docker logs $container_id 
docker inspect $container_id 

Get a Docekrfile from an image: 
```
docker history --no-trunc $imagename | tac | tr -s ' ' | cut -d " " -f 5- | sed 's,^/bin/sh -c #(nop) ,,g' | sed 's,^/bin/sh -c,RUN,g' | sed 's, && ,\n  & ,g' | sed 's,\s*[0-9]*[\.]*[0-9]*\s*[kMG]*B\s*$,,g' | head -n -1
```

tac                                                                                          reverse the file
tr -s ' '                                                                                   trim multiple whitespaces into 1
cut -d " " -f 5-                                                                    remove the first fields (until X months/years ago)
sed 's,^/bin/sh -c #(nop) ,,g'                                      remove /bin/sh calls for ENV,LABEL...
sed 's,^/bin/sh -c,RUN,g'                                            remove /bin/sh calls for RUN
sed 's, && ,\n  & ,g'                                                       pretty print multi command lines following Docker best practices
sed 's,\s*[0-9]*[\.]*[0-9]*\s*[kMG]*B\s*$,,g'      remove layer size information
head -n -1                                                                          remove last line ("SIZE COMMENT" in this case)




TAR, ZIP
add directory recursively with password:
```
$zip –r -e filename.zip directory_name
```

List all files in an archive:
```
tar -tvf archive.tar
tar -jtvf archive.tar.bz2 #-j for bz2
tar -ztvf archive.tar.gz #-z for tar.gz
```

Extract files into specific directory:
```
tar -xf file.name.tar -C /path/to/directory
```

Create an archive that is split in 10 M parts: 
```
tar -cvj ~/path_to_original_file | split -b 10m -d - "name.tar.bz."
```

https://docs.oracle.com/javase/tutorial/deployment/jar/basicsindex.html
Common JAR file operations
Operation
Command
To create a JAR file
jar cf jar-file input-file(s)
To view the contents of a JAR file
jar tf jar-file
To extract the contents of a JAR file
jar xf jar-file
To extract specific files from a JAR file
jar xf jar-file archived-file(s)
To run an application packaged as a JAR file (requires the Main-class manifest header)
java -jar app.jar
To invoke an applet packaged as a JAR file


To extract rpm file:
```
rpm2cpio php-5.1.4-1.esp1.x86_64.rpm | cpio -idmv
```

To see list a downloaded package's dependencies/requirements:
```
$ rpm -qp mypackage.rpm --provides
$ rpm -qp mypackage.rpm --requires
```
( the same works for installed packages with `rpm -qi  mypackage.rpm --provides`)

SAR
https://www.thegeekstuff.com/2011/03/sar-examples/

CPU Usage of ALL CPUs
```
sar -u
```

CPU Usage of Individual CPU or Core
```
sar -P 1 
```

Memory usage:
```
sar -r

#historical data from files:
sar -r -f /var/log/sa/sa10

#SWAP usage:
sar -S
```

grub2, grubby
https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/kernel-module-driver-configuration/Working_with_the_GRUB_2_Boot_Loader/
```
grubby --default-kernel
grubby --default-index
#see man grubby

yum list installed "kernel-*"
sudo yum-complete-transaction
sudo yum reinstall kernel.x86_64
```
also see dracut (https://access.redhat.com/solutions/1958) and depmod (https://access.redhat.com/solutions/228453)


"Command not found" when using sudo

```
sudo -V | grep 'Value to override'
Value to override user's $PATH with: /sbin:/bin:/usr/sbin:/usr/bin

```
If $PATH is being overridden use visudo and edit /etc/sudoers
```
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
```

Setting environmental variables: https://help.ubuntu.com/community/EnvironmentVariables#A.2BAH4-.2F.pam_environment
https://askubuntu.com/questions/866161/setting-path-variable-in-etc-environment-vs-profile , https://stackoverflow.com/questions/8633461/how-to-keep-environment-variables-when-using-sudo/33183620#33183620

## tcpdump

```
 sudo tcpdump -i  eth0 "tcp port 8443 and dst 10.x6x.x8x.1" -XX 
 
```

## awk

To get a sum:
```
awk -F',' '{sum+=$57;} END{print sum;}' file.txt
```

Print all but the first column:

```
awk '{$1=""; print $0}' somefile
```

## scp to multiple nodes and avoid entering password
```
for i in {11..20}; do sshpass -f /path/to/file/with/password  scp -r ~/directory_to_copy username@node${i}-fqdn:/home/username ; done
```

## Check network connections
Display TCP/UDP sockets

```
#for TCP:
ss -t -a
#or
netstat -nat 

#for UDP:
ss -u -a
netstat -nau
```

iftop listens to network traffic and displays a table of current bandwidth usage by pairs of hosts
```
iftop -i eth1
iftop -F 192.168.1.0/24

```

View established connections only
```
netstat -natu | grep 'ESTABLISHED'
```

tcptrack monitors their state and displays information such as state, source/destination addresses and bandwidth usage
```
tcptrack -i eth0
```

### nmap

Scan all ports on all hosts in the network: `nmap 192.168.0.0/24`

A quick scan to find live host: `nmap -sn 192.168.0.0/24`. In older versions, instead of -sn, nmap accepted -Sp.
For more grapable format, use -oG

Scanning specific ports, regardless of their state: open, closed, filtered, etc: `nmap -sV -p 22,443 192.168.0.0/24`

Packet-tracing with nmap: `nmap -vv -n -sn -PE -T4 --packet-trace 192.168.2.3`

#### NSE scripts
```
nmap --script="name_of_script" --script-args="argument=arg" target
```
Scripts are available in /usr/share/nmap/scripts/

### List cron jobs for all users
```
for user in $(cut -f1 -d: /etc/passwd); do crontab -u $user -l; done
```

