# dnsmasq-ipsets-shift
* shift the ipsets support from dest IP to src IP
* this project's purpose is to identify and mark the dns requester's IP in ipset.

# install
```
make && make install
```

# dependence
```
apt-get install ipset
```

# usage
* first you have to create an ipset:
```
[ $(ipset list listname 2>/dev/null| wc -l) -eq 0 ] && { ipset -N listname iphash; }
```
* add ipset in conf file like this:
```
ipset=/domain.com/listname
```
* then a query for domain.com from a.b.c.d, The IP a.b.c.d will be added to the ipset listname
```
ipset list listname
```

# IP unique and more examples
* a request can be add to one ipset only
```
[ $(ipset list internal_list 2>/dev/null| wc -l) -eq 0 ] && { ipset -N internal_list iphash; }
[ $(ipset list international_list 2>/dev/null| wc -l) -eq 0 ] && { ipset -N international_list iphash; }
[ $(ipset list gaming_list1 2>/dev/null| wc -l) -eq 0 ] && { ipset -N gaming_list1 iphash; }
[ $(ipset list gaming_list2 2>/dev/null| wc -l) -eq 0 ] && { ipset -N gaming_list2 iphash; }

cat >> /etc/dnsmasq.conf <<EOT
ipset=/baidu.com/internal_list
ipset=/google.com/international_list
ipset=/xbox.ea.com/gaming_list1
ipset=/pspc.ea.com/gaming_list2
EOT

iptables -t nat -PREROUTING -i eth0 -p tcp -m tcp --dports 80,443 -m set --match-set internal_list -j RETURN
iptables -t nat -PREROUTING -i eth0 -p tcp -m tcp --dports 80,443 -m set --match-set international_list -j DNAT --to-destination 192.168.1.1:1080
iptables -t nat -PREROUTING -i eth0 -p tcp -m tcp --dport 10001:10099 -m set --match-set gaming_list1 -j DNAT --to-destination 162.13.234.1
iptables -t nat -PREROUTING -i eth0 -p tcp -m tcp --dport 10001:10099 -m set --match-set gaming_list2 -j DNAT --to-destination 162.13.234.2
```
* if IP a.b.c.d queried xbox.ea.com, in the dnsmasq log you will see
```
ipset del internal_list a.b.c.d xbox.ea.com
ipset del internation_list a.b.c.d xbox.ea.com
ipset add game_list1 a.b.c.d xbox.ea.com
ipset del game_list2 a.b.c.d xbox.ea.com
```
* you will see IP a.b.c.d in game_list1 if you type `ipset list game_list1`
```
Name: game_list1
Type: hash:ip
Header: family inet hashsize 1024 maxelem 65536 
Size in memory: 16536
References: 0
Members:
a.b.c.d
```
