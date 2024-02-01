### Managing firewalld
```
firewall-cmd --state     # Display whether service is running
firewall-cmd --reload    # To reload the permanent rules without interrupting existing persistent connections
```

### list details of zones 
```
firewall-cmd --get-zones 
firewall-cmd --get-default-zone
firewall-cmd --get-active-zones
firewall-cmd --list-all --zone=public
firewall-cmd --permanent --zone=public --get-target
```

### add/remove interfaces to zones

To add interface “eth1” to “public” zone.
```
firewall-cmd --zone=public --change-interface=eth1
```

### list/add/remove services to zones
To add “samba and samba-client” service to a specific zone. You may include, “permanent” flag to make this permanent change.
```
firewall-cmd --zone=public --add-service=samba --add-service=samba-client --permanent
firewall-cmd --permanent --zone=public --remove-service=ssh
firewall-cmd --zone=public --add-service=ssh --timeout=5m

# a rich rule:
firewall-cmd --permanent --zone=internal --add-rich-rule='rule family=ipv4 source address="1.1.1.0/8" service name="samba" accept'
```

### list and Add ports to firewall
```
firewall-cmd --list-ports
firewall-cmd --zone=public --add-port=5000/tcp

```
## change the public zone's target to DROP
```
firewall-cmd --permanent --zone=public --set-target=DROP
# don't forget to allow ping:
firewall-cmd --permanent --zone=internal --add-rich-rule='rule protocol value="icmp" accept'

firewall-cmd --reload
```