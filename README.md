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


# 注意以下两者的区别
--delete-service</br>
--remove-service

# 创建ipset集合--失败的方法
最初的想法是，使用firewall-cmd原生命令创建大量的ipset集合备用。
但是下述脚本，执行了2天多，才执行了2/3。此方法淘汰。
```
#!/bin/bash
directory=/tmp/country_block_$(date +"%Y%m%d%H%M%S")
mkdir ${directory} && cd ${directory}
wget -c https://www.ipdeny.com/ipblocks/data/countries/all-zones.tar.gz
tar -zxf all-zones.tar.gz

for name in `ls *.zone`
do
    firewall-cmd --permanent --new-ipset=${name} --type=hash:net
    cat ${name} | while read line
    do
        firewall-cmd --permanent --ipset=${name} --add-entry=${line}
    done
done
```
# 改良版本
但是上面的失败也是有价值的。</br>
发现在/etc/firewalld/ipsets目录下，生成的cn.zone.xml的文件都是有固定结构。</br>
```
<?xml version="1.0" encoding="utf-8"?>
<ipset type="hash:net">
  <entry>1.0.1.0/24</entry>
  <entry>91.234.36.0/24</entry>
</ipset>
```
所以，换个思路，直接用脚本生成xml文件，然后放在该目录下。</br>
进一步分析，也可以发现之前的脚本为什么这么慢。因为每次执行命令，都会生成一次xml.old文件。这个文件是原来的备份。</br>
如果可以选择不生成这个文件，我估计效率应该会快很多。

```
#!/bin/bash
directory=/tmp/country_block_$(date +"%Y%m%d%H%M%S")
mkdir ${directory} && cd ${directory}
wget -c https://www.ipdeny.com/ipblocks/data/countries/all-zones.tar.gz
tar -zxf all-zones.tar.gz

for i in `ls *.zone`
do
    echo '<?xml version="1.0" encoding="utf-8"?>' >> ${i}.xml
    echo '<ipset type="hash:net">' >> ${i}.xml
    cat ${i} | while read line
    do
        echo "  <entry>${line}</entry>" >> ${i}.xml
    done
    echo '</ipset>' >> ${i}.xml
done
```

# 实战演练<只允许中国的IP访问>
假设当前的zone是默认的public，没有改动过
拷贝cn.zone.xml文件到/etc/firewalld/ipsets目录
```
firewall-cmd --permanent --add-rich-rule 'rule family="ipv4" source ipset="cn.zone" port port=22 protocol=tcp accept'
firewall-cmd --reload
firewall-cmd --remove-service=ssh
```
换个IP重新登录(如果之前是--complete-reload就不需要更换IP)。如果可以正常登录，那就没问题，可以改为永久生效。
（上面的操作是为了不把自己锁在外面）
```
firewall-cmd --permanent --remove-service=ssh
```

**操作后验证**
查看系统日志，可以发现原来有各个国家的IP一直在尝试非法访问，现在应该只剩国内的IP了。
tail -f /var/log/secure | egrep -o '(([0-1]?[0-9]{0,2}|([2]([0-4][0-9]|[5][0-5])))\.){3}([0-1]?[0-9]{0,2}|([2]([0-4][0-9]|[5][0-5])))'
