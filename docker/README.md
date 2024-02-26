## To run netstat in any docker container

1. Get the PID of your Docker container:
```
docker inspect -f '{{.State.Pid}}' container_name_or_id
```

2. Use the PID as the argument to the target (-t) option of nsenter. For example, to run netstat inside the container network namespace:

```
sudo nsenter -t 15652 -n netstat -ntulp
```

## A awk script to run a "self-made netstat" inside a container

```
awk 'function hextodec(str,ret,n,i,k,c){
    ret = 0
    n = length(str)
    for (i = 1; i <= n; i++) {
        c = tolower(substr(str, i, 1))
        k = index("123456789abcdef", c)
        ret = ret * 16 + k
    }
    return ret
}
function getIP(str,ret){
    ret=hextodec(substr(str,index(str,":")-2,2));
    for (i=5; i>0; i-=2) {
        ret = ret"."hextodec(substr(str,i,2))
    }
    ret = ret":"hextodec(substr(str,index(str,":")+1,4))
    return ret
}
NR > 1 {{if(NR==2)print "Local - Remote";local=getIP($2);remote=getIP($3)}{print local" - "remote}}' /proc/net/tcp
```
From https://staaldraad.github.io/2017/12/20/netstat-without-netstat/

or

```
grep -v "rem_address" /proc/net/tcp | awk 'function hextonum(str, ret, n, i, k, c) {if (str ~ /^0[xX][0-9a-FA-F]+$/) {str = substr(str, 3);n = length(str);ret = 0;for (i = 1; i <= n; i++) {c = substr(str, i, 1);c = tolower(c);k = index("123456789abcdef", c);ret = ret * 16 + k}} else ret = "NOT-A-NUMBER";return ret} {y=hextonum("0x"substr($2,index($2,":")-2,2));x=hextonum("0x"substr($3,index($3,":")-2,2));for (i=5; i>0; i-=2) {x = x"."hextonum("0x"substr($3,i,2));y = y"."hextonum("0x"substr($2,i,2));} print y":"hextonum("0x"substr($2,index($2,":")+1,4))" "x":"hextonum("0x"substr($3,index($3,":")+1,4));}'
```

## Proc as an alternative to the ps 
```
for proc in /proc/*/cmdline; do echo $(basename $(dirname $proc)) $(cat $proc | tr "\0" " "); done
# or
for proc in /proc/[0-9]*/cmdline; do echo $(basename $(dirname $proc)) $(cat $proc | tr "\0" " "); done
```