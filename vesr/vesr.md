# Настраиваем vESR

Дефолтные креды: **admin/password**

Сразу после логина надо поменять

```text
password P@ssw0rd
commit
confirm
save
```

## Базовая настройка

Одинаковая на RTR1 и RTR2

Ставим хостнеймы и внешние адреса в сторону испа

```text
hostname RTR1
domain name company.prof
int g 1/0/0
ip address 10.10.10.2/30
ip firewall disable
exit
ip route 0.0.0.0/0 10.10.10.1
end
commit
confirm
save
```

Подинтерфейсы настраиваем следующим образом

```text
configure
int g 1/0/1.100
ip firewall disable
ip address 10.0.10.1/27
exit
int g 1/0/1.200
ip firewall disable
ip address 10.0.10.33/27
exit
int g 1/0/1.300
ip firewall disable
ip address 10.0.10.65/27
end
commit
confirm
save
```

На RTR2 настраивается аналогично, меняются подсети

## DHCP

```text
ip dhcp-server
ip dhcp-server pool vlan100
network 10.0.10.0/27
address-range 10.0.10.10-10.0.10.20
default-router 10.0.10.1
domain-name company.prof
end
commit
confirm
save
```

Для остальных vlan и RTR2 настройка аналогичная

## Логи

```text
configure
syslog host SRV1
remote-address 10.0.10.10
transport udp
severity debug
port 514
end
commit
confirm
save
```

## NAT

```text
configure
object-group network LOCAL
ip prefix 10.0.10.0/27
exit
object-group network WAN
ip address-range 10.10.10.2
exit
nat source
pool WAN
ip address-range 10.10.10.2
ruleset MASQUERADE
to interface g 1/0/1
rule 1
match source-address LOCAL
action source-nat pool WAN
enable
end
commit
confirm
save
```

## IPSEC

Настройки айписека должны быть абсолютно одинаковые на обоих роутерах

```text

object-group service ISAKMP
 port-range 500
 exit

security ike proposal ike_prop1
 dh-group 2
 authentication algorithm md5
 encryption algorithm aes128
 exit

security ike policy ike_pol1
 pre-shared-key hexadecimal 123FFF
 proposal ike_prop1
 exit

security ike gateway ike_gw1
 ike-policy ike_pol1
 mode route-based
 bind-interface vti 1
 version v2-only
 exit

security ipsec proposal ipsec_prop1
 authentication algorithm md5
 encryption algorithm aes128
 exit

security ipsec policy ipsec_pol1
 proposal ipsec_prop1
 exit

security ipsec vpn ipsec1
 ike establish-tunnel immediate
 ike gateway ike_gw1
 ike ipsec-policy ipsec_pol1
 enable
 exit
exit

commit
confirm
```

## OSPF

```text

# RTR1:
router ospf log-adjacency-changes
router ospf 1
 area 1.1.1.1
  network 10.0.10.0/28
  network 10.0.10.32/28
  network 10.0.10.128/27
  enable
  exit
 enable
 exit

# RTR2
router ospf log-adjacency-changes
router ospf 1
 area 1.1.1.1
  network 10.0.20.0/28
  network 10.0.20.32/28
  network 10.0.20.128/27
  enable
  exit
 enable
 exit

# RTR1
tunnel vti 1
 ip firewall disable
 local address 10.10.10.2
 remote address 10.10.20.2
 ip address 100.0.10.1/30
 ip ospf instance 1
 ip ospf area 0.0.0.0
 ip ospf
 enable
 exit

#RTR2
tunnel vti 1
 ip firewall disable
 local address 10.10.20.2
 remote address 10.10.10.2
 ip address 100.0.10.2/30
 ip ospf instance 1
 ip ospf area 1.1.1.1
 ip ospf
 enable
 exit
```
