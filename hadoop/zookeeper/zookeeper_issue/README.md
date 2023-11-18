During my first attempt to instal HDP on lxc1 using Vagrant, I bumped into this issue. Tried the following:

1. checked /etc/zookeeper/3.1.4.0-315/0/zoo.cfg in all nodes, checked that /var/lib/zookeeper/myid corresponds to names in configs, changed owners of /var/lib/zookeeper. Restarted nodes 1 by 1 from Ambari. No success
2. ```/data1/hdp/3.1.4.0-315/zookeeper/bin/zkCli.sh -server 127.0.0.1:2181``` didnt work
3. `echo stat | nc localhost 2181` reported that zookeeper didn't serve requests. `echo 'ruok' | nc localhost 2181; echo` responded that zookeeper is ok (`imok`). `telnet hadoopcm1-li-lxc1-${fqdn} 2181` also worked
4. Stopped all zookeeper nodes, cleaned logs, then started the nodes. It was clear at that point that there was a problem with leader election. After some reading, found that there were no processes listening on port 3888, which is used for leader election. Running `telnet 192.168.10.21 3888` from 192.168.10.20 failed. 
5. Actually, Zookeeper was listening on 3888 port, but only on 127.0.1.1 (local loop). It turned out that FQDN's were bind to 127.0.1.1 (entries were in /etc/hosts ). After commenting them out, and restarted Zookeeper, everything worked.

It is Vagrant that adds these `127.0.1.1` lines in /etc/hosts. Note the discussion on https://github.com/hashicorp/vagrant/issues/7263. The suggestion is: "the modification of /etc/hosts only occurs when the user customizes the hostname using config.vm.hostname = "...". A potentially easier solution is to omit that configuration and set the hostname via a custom shell script instead. '. In the case of HDP repo, it'll have another side effect, namely not having FQDN bind to static IP address. Probably, /etc/hosts should be changed during ansible provisioning.

More about 127.0.1.1
The IP address 127.0.1.1 in the second line of this example may not be found on some other Unix-like systems. The Debian Installer creates this entry for a system without a permanent IP address as a workaround for some software (e.g., GNOME) as documented in the bug #719621.

The <host_name> matches the hostname defined in the “/etc/hostname”.

For a system with a permanent IP address, that permanent IP address should be used here instead of 127.0.1.1.

For a system with a permanent IP address and a fully qualified domain name (FQDN) provided by the Domain Name System (DNS), that canonical <host_name>.<domain_name> should be used instead of just <host_name>. 

(https://www.digitalocean.com/community/questions/removed-127-0-1-1-hostname-hostname-from-etc-hosts-after-installing-sendmail-ubuntu)

https://www.debian.org/doc/manuals/debian-reference/ch05.en.html#_the_hostname_resolution

https://askubuntu.com/questions/754213/what-is-difference-between-localhost-address-127-0-0-1-and-127-0-1-1

https://serverfault.com/questions/363095/why-does-my-hostname-appear-with-the-address-127-0-1-1-rather-than-127-0-0-1-in
