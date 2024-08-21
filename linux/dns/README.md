## The issue with superset
### Some links
https://github.com/apache/superset/blob/master/superset/config.py
https://github.com/python/cpython/blob/main/Lib/ssl.py#L1027C22-L1027C33 

Insert a line with sed:
```sed -i '8i This is Line 8' FILE```

Insert line with leading spaces to a specific line:
```sed -i "${line} i \ \ ${text}" $file```

Delete a line with sed: ```sed '5d' input.txt```

How to configure DNS caching server with dnsmasq in RHEL: https://access.redhat.com/solutions/2189401 
How to find a domain's authoritative nameservers: https://jvns.ca/blog/2022/01/11/how-to-find-a-domain-s-authoritative-nameserver/