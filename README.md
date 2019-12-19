# [firewalld Rich Language](https://fedoraproject.org/wiki/Features/FirewalldRichLanguage)

Example 1: Enable new IPv4 and IPv6 connections for protocol 'ah'
```
firewall-cmd --add-rich-rule='rule protocol value="ah" accept'
```
Example 2: Allow new IPv4 and IPv6 connections for service ftp and log 1 per minute using audit
```
firewall-cmd --add-rich-rule='rule service name="ftp" audit limit value="1/m" accept'
```
Example 3: Allow new IPv4 connections from address 192.168.0.0/24 for service tftp and log 1 per minutes using syslog
```
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.0.0/24" service name="tftp" log prefix="tftp" level="info" limit value="1/m" accept'
```
Example 4: New IPv6 connections from 1:2:3:4:6:: to service radius are all rejected and logged at a rate of 3 per minute. New IPv6 connections from other sources are accepted.
```
firewall-cmd --add-rich-rule='rule family="ipv6" source address="1:2:3:4:6::" service name="radius" log prefix="dns" level="info" limit value="3/m" reject rule family="ipv6" service name="radius" accept'
```
Example 5: Forward IPv6 port/packets receiving from 1:2:3:4:6:: on port 4011 with protocol tcp to 1::2:3:4:7 on port 4012
```
firewall-cmd --add-rich-rule='rule family="ipv6" source address="1:2:3:4:6::" forward-port to-addr="1::2:3:4:7" to-port="4012" protocol="tcp" port="4011"'
```
Example 6: White-list source address to allow all connections from 192.168.2.2
```
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.2.2" accept'
```
Example 7: Black-list source address to reject all connections from 192.168.2.3
```
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.2.3" reject'
```
Example 8: Black-list source address to drop all connections from 192.168.2.4
```
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.2.4" drop'
```


# 注意以下两者的区别：
--delete-service
--remove-service
