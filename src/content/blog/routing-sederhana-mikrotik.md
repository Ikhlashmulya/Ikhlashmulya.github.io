---
title: Routing sederhana mikrotik
date: 01-09-2025
slug:  routing-sederhana-mikrotik
---

Simulasi dilakukan dengan gns3 dengan skema NAT-mikrotik-Client(VPCS) seperti gambar berikut
![routing sederhana 1](/images/routing-mikrotik1.png)

Pada simulasi di atas, konfigurasi minimal router mikrotik sebagai berikut

- cek apakah sudah dapat ip dari NAT dengan
```
ip dhcp-client print
```
jika belum tambahkan
```
/ip dhcp-client add interface=ether1 disabled=no
/ip dhcp-client enable 0
```

- tambahkan ip untuk ether2
```
/ip address add address=192.168.1.1/24 interface=ether2
```

cek apakah sudah ditambahkan dengan command berikut

```
/ip address print
```

- tambahkan masquerade agar bisa konek internet
```
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
```

cek apakah sudah ditambahkan dengan command berikut

```
/ip firewall nat print
```

- tambahkan dns nya juga
```
/ip dns set servers=192.168.122.1 allow-remote-requests=yes
```
dnsnya bisa dari IP yang didapat dari NAT

cek apakah sudah ditambahkan dengan command berikut
```
/ip dns print
```

- selanjutnya tinggal set di clientnya (VPCS), dengan command berikut
```
ip 192.168.1.2 192.168.1.1
ip dns 192.168.122.1
```

- test ping

ke gateway
```
ping 192.168.1.1
```

ke internet
```
ping google.com
```

## Bridge

Bridge di MikroTik adalah fitur yang memungkinkan perangkat di beberapa interface jaringan (seperti Ethernet, WLAN, atau VLAN) menjadi satu jaringan atau segmentasi yang sama. Dengan kata lain, bridge menggabungkan beberapa interface fisik atau virtual sehingga perangkat yang terhubung ke interface-interface tersebut dapat berkomunikasi seolah-olah mereka berada dalam jaringan yang sama.

contoh simulasi di gns3 sebagai berikut
![routing sederhana 2](/images/routing-mikrotik2.png)

di gambar simulasi di atas client pc1 pc2 pc3 akan dianggap sebagai satu interface jaringan yang sama. Berikut adalah konfigurasinya

- membuat bridge baru dengan nama bridge1
```
/interface bridge add name=bridge1
```

- menambahkan beberapa interface ke dalam bridge1
```
/interface brigde port
add bridge=bridge1 interface=ether2
add bridge=bridge1 interface=ether3
add bridge=bridge1 interface=ether4
```

- selanjutnya edit IP 192.168.1.1/24 yang sudah ditambahkan ke ether2 jadi ke brigde1
```
/ip address edit 0 interface
```

ubah ether2 ke bridge1 kemudian ctrl+o

jika ingin menambahkan IP baru bisa dengan command berikut

```
/ip address add interface=bridge1 address=192.168.1.1/24
```

- selanjutnya set ip di client

pc1
```
ip 192.168.1.2 192.168.1.1
ip dns 192.168.122.1
```

pc2
```
ip 192.168.1.3 192.168.1.1
ip dns 192.168.122.1
```

pc3
```
ip 192.168.1.4 192.168.1.1
ip dns 192.168.122.1
```

## DHCP Server

DHCP Server di MikroTik adalah fitur yang memungkinkan router MikroTik untuk menyediakan alamat IP secara otomatis kepada perangkat yang terhubung ke jaringan. Dengan adanya DHCP Server, perangkat dalam jaringan tidak perlu dikonfigurasi secara manual untuk mendapatkan alamat IP, melainkan akan menerima konfigurasi IP (seperti alamat IP, subnet mask, gateway, dan DNS) secara otomatis dari server.

untuk menambahkan dhcp secara otomatis bisa menggunakan command
```
/ip dhcp-server setup
```

chat saat konfigurasi
```
Select interface to run DHCP server on 

dhcp server interface: bridge1 -- ketik nama bridge/interface
Select network for DHCP addresses 

dhcp address space: 192.168.1.0/24 -- sesuai IP di brigde/interface
Select gateway for given network 

gateway for dhcp network: 192.168.1.1 -- sesuai IP gateway di brigde/interface
Select pool of ip addresses given out by DHCP server 

addresses to give out: 192.168.1.2-192.168.1.254 -- range IP
Select DNS servers 

dns servers: 192.168.122.1 -- bisa 8.8.8.8 atau jika pake NAT bisa dengan IP yang didapat dari NAT
Select lease time 

lease time: 10m
```

atau jika ingin setup secara manual
```
/ip pool add name=pool8 ranges=YYY.YYY.YYY.100-YYY.YYY.YYY.120
/ip dhcp-server network add address=YYY.YYY.YYY.0/24 gateway=YYY.YYY.YYY.1
/ip dhcp-server add interface=ether8 address-pool=pool8
/ip dhcp-server network add dns-server=8.8.8.8
/ip dhcp-server enable
```

di client-nya (VPCS) cukup dengan command
```
ip dhcp
```

setelah menambahkan ip dhcp di client tidak perlu repot2 menset ip, gateway dan juga dns nya karena sudah otomatis ditambahkan dari dhcp server.

### referensi

- [onnocenter.or.id/wiki/index.php/Mikrotik](https://onnocenter.or.id/wiki/index.php/Mikrotik)