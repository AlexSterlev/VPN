# VPN
- Между двумя виртуалками поднять vpn в режимах
tun и tap Прочуствовать разницу.
- Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на виртуалку.
## TUN и TAB
TUN и TAP — виртуальные сетевые драйверы ядра системы. Они представляют собой программные сетевые устройства, которые отличаются от обычных аппаратных сетевых карт.
TAP эмулирует Ethernet устройство и работает на канальном уровне модели OSI, оперируя кадрами Ethernet. TUN (сетевой туннель) работает на сетевом уровне модели OSI, оперируя IP пакетами. TAP используется для создания сетевого моста, тогда как TUN для маршрутизации.

Устанавливаем сервер:
````
[root@server ~]# yum -y install epel-release
[root@server ~]# yum install -y openvpn iperf3 policycoreutils-python
````
Генерируем ключ:
````
[root@server ~]# openvpn --genkey --secret /etc/openvpn/static.key
````
Конфиг сервера:
````
dev tap
proto udp

ifconfig 10.10.10.1 255.255.255.0
topology subnet

secret /etc/openvpn/static.key

status /var/log/openvpn-status.log
log /var/log/openvpn.log

verb 3
````
Запускаем сервер:
````
[root@server ~]# systemctl enable --now openvpn@server

[root@server ~]# ip -c a show tap0
5: tap0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
    link/ether 5e:70:c6:f1:48:75 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 brd 10.10.10.255 scope global tap0
       valid_lft forever preferred_lft forever
    inet6 fe80::5c70:c6ff:fef1:4875/64 scope link 
       valid_lft forever preferred_lft forever
````
Подготовка клиента:
````
[root@client ~]# yum -y install epel-release
[root@client ~]# yum install -y openvpn iperf3 policycoreutils-python
````
Конфиг клиента:
````
dev tap
proto udp
remote 192.168.10.10

ifconfig 10.10.10.2 255.255.255.0
topology subnet
route 192.168.10.0 255.255.255.252

secret /etc/openvpn/static.key

status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3
````
Запускаем клиента:
````
[root@client ~]# systemctl enable --now openvpn@server
````
Проверяем подключение:
````
[root@client ~]# cat /var/log/openvpn-status.log 
OpenVPN STATISTICS
Updated,Wed Jul 21 08:24:44 2021
TUN/TAP read bytes,0
TUN/TAP write bytes,0
TCP/UDP read bytes,0
TCP/UDP write bytes,0
Auth read bytes,0
END

[root@client ~]# ip r | grep tap
10.10.10.0/24 dev tap0 proto kernel scope link src 10.10.10.2 
````
Замер скорости:
````
[root@server ~]# iperf3 -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 10.10.10.2, port 38196
[  5] local 10.10.10.1 port 5201 connected to 10.10.10.2 port 38198
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  18.5 MBytes   155 Mbits/sec                  
[  5]   1.00-2.01   sec  20.0 MBytes   167 Mbits/sec                  
[  5]   2.01-3.00   sec  19.5 MBytes   165 Mbits/sec                  
[  5]   3.00-4.01   sec  20.1 MBytes   168 Mbits/sec                  
[  5]   4.01-5.01   sec  20.1 MBytes   168 Mbits/sec                  
[  5]   5.01-6.00   sec  19.6 MBytes   165 Mbits/sec                  
[  5]   6.00-7.00   sec  19.6 MBytes   164 Mbits/sec                  
[  5]   7.00-8.00   sec  19.7 MBytes   165 Mbits/sec                  
[  5]   8.00-9.01   sec  20.3 MBytes   169 Mbits/sec                  
[  5]   9.01-10.01  sec  19.8 MBytes   167 Mbits/sec                  
[  5]  10.01-11.00  sec  19.5 MBytes   164 Mbits/sec                  
[  5]  11.00-12.00  sec  20.2 MBytes   169 Mbits/sec                  
[  5]  12.00-13.00  sec  19.5 MBytes   163 Mbits/sec                  
[  5]  13.00-14.00  sec  19.8 MBytes   167 Mbits/sec                  
[  5]  14.00-15.00  sec  20.3 MBytes   170 Mbits/sec                  
[  5]  15.00-16.00  sec  19.9 MBytes   168 Mbits/sec                  
[  5]  16.00-17.00  sec  20.0 MBytes   167 Mbits/sec                  
[  5]  17.00-18.00  sec  19.6 MBytes   164 Mbits/sec                  
[  5]  18.00-19.01  sec  19.9 MBytes   166 Mbits/sec                  
[  5]  19.01-20.00  sec  19.0 MBytes   160 Mbits/sec                  
[  5]  20.00-21.01  sec  20.1 MBytes   168 Mbits/sec                  
[  5]  21.01-22.00  sec  19.4 MBytes   163 Mbits/sec                  
[  5]  22.00-23.00  sec  19.1 MBytes   160 Mbits/sec                  
[  5]  23.00-24.01  sec  19.5 MBytes   163 Mbits/sec                  
[  5]  24.01-25.00  sec  19.5 MBytes   164 Mbits/sec                  
[  5]  25.00-26.01  sec  19.4 MBytes   162 Mbits/sec                  
[  5]  26.01-27.00  sec  20.0 MBytes   168 Mbits/sec                  
[  5]  27.00-28.00  sec  20.1 MBytes   169 Mbits/sec                  
[  5]  28.00-29.01  sec  19.4 MBytes   162 Mbits/sec                  
[  5]  29.01-30.00  sec  20.3 MBytes   171 Mbits/sec                  
[  5]  30.00-31.00  sec  20.2 MBytes   169 Mbits/sec                  
[  5]  31.00-32.00  sec  20.4 MBytes   171 Mbits/sec                  
[  5]  32.00-33.00  sec  19.7 MBytes   165 Mbits/sec                  
[  5]  33.00-34.00  sec  20.2 MBytes   170 Mbits/sec                  
[  5]  34.00-35.01  sec  19.2 MBytes   160 Mbits/sec                  
[  5]  35.01-36.01  sec  20.5 MBytes   172 Mbits/sec                  
[  5]  36.01-37.00  sec  19.3 MBytes   162 Mbits/sec                  
[  5]  37.00-38.00  sec  19.7 MBytes   165 Mbits/sec                  
[  5]  38.00-39.01  sec  20.0 MBytes   167 Mbits/sec                  
[  5]  39.01-40.01  sec  20.3 MBytes   170 Mbits/sec                  
[  5]  40.01-40.07  sec  1021 KBytes   144 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-40.07  sec  0.00 Bytes  0.00 bits/sec                  sender
[  5]   0.00-40.07  sec   792 MBytes   166 Mbits/sec                  receiver



[root@client ~]# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  4] local 10.10.10.2 port 38198 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec  99.8 MBytes   167 Mbits/sec  122    327 KBytes       
[  4]   5.00-10.00  sec  99.1 MBytes   166 Mbits/sec  138    216 KBytes       
[  4]  10.00-15.01  sec  99.8 MBytes   167 Mbits/sec   67    267 KBytes       
[  4]  15.01-20.01  sec  98.0 MBytes   164 Mbits/sec  138    152 KBytes       
[  4]  20.01-25.00  sec  97.5 MBytes   164 Mbits/sec  195    191 KBytes       
[  4]  25.00-30.00  sec  99.0 MBytes   166 Mbits/sec   14    324 KBytes       
[  4]  30.00-35.00  sec   100 MBytes   168 Mbits/sec   51    172 KBytes       
[  4]  35.00-40.00  sec  99.6 MBytes   167 Mbits/sec  137    262 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-40.00  sec   793 MBytes   166 Mbits/sec  862             sender
[  4]   0.00-40.00  sec   792 MBytes   166 Mbits/sec                  receiver

iperf Done.
````
Изменим режим работы и посмотри что из этого вышло:
````
[root@client ~]# ip -c a s dev tun0
7: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
    link/none 
    inet 10.10.10.2/24 brd 10.10.10.255 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::6da2:c0c0:7eaf:26fe/64 scope link flags 800 
       valid_lft forever preferred_lft forever


[root@server ~]# iperf3 -s                       
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 10.10.10.2, port 38200
[  5] local 10.10.10.1 port 5201 connected to 10.10.10.2 port 38202
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  18.8 MBytes   158 Mbits/sec                  
[  5]   1.00-2.00   sec  20.5 MBytes   172 Mbits/sec                  
[  5]   2.00-3.00   sec  20.3 MBytes   171 Mbits/sec                  
[  5]   3.00-4.01   sec  20.6 MBytes   172 Mbits/sec                  
[  5]   4.01-5.00   sec  19.9 MBytes   168 Mbits/sec                  
[  5]   5.00-6.00   sec  20.0 MBytes   168 Mbits/sec                  
[  5]   6.00-7.00   sec  20.3 MBytes   170 Mbits/sec                  
[  5]   7.00-8.00   sec  20.0 MBytes   168 Mbits/sec                  
[  5]   8.00-9.01   sec  20.2 MBytes   168 Mbits/sec                  
[  5]   9.01-10.01  sec  20.3 MBytes   170 Mbits/sec                  
[  5]  10.01-11.00  sec  20.0 MBytes   169 Mbits/sec                  
[  5]  11.00-12.00  sec  17.1 MBytes   144 Mbits/sec                  
[  5]  12.00-13.00  sec  19.8 MBytes   166 Mbits/sec                  
[  5]  13.00-14.00  sec  19.5 MBytes   164 Mbits/sec                  
[  5]  14.00-15.00  sec  19.7 MBytes   165 Mbits/sec                  
[  5]  15.00-16.01  sec  20.4 MBytes   171 Mbits/sec                  
[  5]  16.01-17.01  sec  20.1 MBytes   169 Mbits/sec                  
[  5]  17.01-18.01  sec  20.6 MBytes   173 Mbits/sec                  
[  5]  18.01-19.00  sec  19.9 MBytes   168 Mbits/sec                  
[  5]  19.00-20.01  sec  20.6 MBytes   171 Mbits/sec                  
[  5]  20.01-21.01  sec  20.1 MBytes   169 Mbits/sec                  
[  5]  21.01-22.00  sec  19.8 MBytes   168 Mbits/sec                  
[  5]  22.00-23.01  sec  20.1 MBytes   167 Mbits/sec                  
[  5]  23.01-24.00  sec  20.3 MBytes   171 Mbits/sec                  
[  5]  24.00-25.00  sec  19.9 MBytes   167 Mbits/sec                  
[  5]  25.00-26.00  sec  20.1 MBytes   168 Mbits/sec                  
[  5]  26.00-27.00  sec  19.9 MBytes   167 Mbits/sec                  
[  5]  27.00-28.00  sec  19.6 MBytes   165 Mbits/sec                  
[  5]  28.00-29.00  sec  19.5 MBytes   164 Mbits/sec                  
[  5]  29.00-30.00  sec  20.0 MBytes   168 Mbits/sec                  
[  5]  30.00-31.00  sec  20.0 MBytes   168 Mbits/sec                  
[  5]  31.00-32.01  sec  20.0 MBytes   167 Mbits/sec                  
[  5]  32.01-33.01  sec  19.6 MBytes   165 Mbits/sec                  
[  5]  33.01-34.00  sec  19.9 MBytes   168 Mbits/sec                  
[  5]  34.00-35.00  sec  19.4 MBytes   163 Mbits/sec                  
[  5]  35.00-36.01  sec  19.9 MBytes   166 Mbits/sec                  
[  5]  36.01-37.01  sec  19.2 MBytes   162 Mbits/sec                  
[  5]  37.01-38.01  sec  19.9 MBytes   167 Mbits/sec                  
[  5]  38.01-39.01  sec  19.9 MBytes   167 Mbits/sec                  
[  5]  39.01-40.01  sec  20.4 MBytes   171 Mbits/sec                  
[  5]  40.01-40.06  sec  1.09 MBytes   165 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-40.06  sec  0.00 Bytes  0.00 bits/sec                  sender
[  5]   0.00-40.06  sec   798 MBytes   167 Mbits/sec                  receiver




[root@client ~]# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  4] local 10.10.10.2 port 38202 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.01   sec   102 MBytes   172 Mbits/sec  224    311 KBytes       
[  4]   5.01-10.00  sec   101 MBytes   170 Mbits/sec  243    275 KBytes       
[  4]  10.00-15.01  sec  96.2 MBytes   161 Mbits/sec  286    184 KBytes       
[  4]  15.01-20.01  sec   101 MBytes   170 Mbits/sec    1    336 KBytes       
[  4]  20.01-25.01  sec   100 MBytes   169 Mbits/sec  324    278 KBytes       
[  4]  25.01-30.01  sec  99.2 MBytes   166 Mbits/sec  142    312 KBytes       
[  4]  30.01-35.00  sec  98.9 MBytes   166 Mbits/sec  145    179 KBytes       
[  4]  35.00-40.00  sec  98.9 MBytes   166 Mbits/sec   18    353 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-40.00  sec   798 MBytes   167 Mbits/sec  1383             sender
[  4]   0.00-40.00  sec   798 MBytes   167 Mbits/sec                  receiver

iperf Done.
````
Как видно из результатов замера скорости в данном случае разница минимальна, но если мы воспользуемся утилитой arping разница в природе TUN и TAP будет более очевидна.
### TAP
````
[root@client ~]# arping  10.10.10.1 -I tap0
ARPING 10.10.10.1 from 10.10.10.2 tap0
Unicast reply from 10.10.10.1 [4A:78:5F:5C:61:6E]  3.447ms
Unicast reply from 10.10.10.1 [4A:78:5F:5C:61:6E]  2.791ms
Unicast reply from 10.10.10.1 [4A:78:5F:5C:61:6E]  5.123ms
Unicast reply from 10.10.10.1 [4A:78:5F:5C:61:6E]  3.083ms
Unicast reply from 10.10.10.1 [4A:78:5F:5C:61:6E]  2.911ms
Unicast reply from 10.10.10.1 [4A:78:5F:5C:61:6E]  2.969ms
Unicast reply from 10.10.10.1 [4A:78:5F:5C:61:6E]  2.957ms
Unicast reply from 10.10.10.1 [4A:78:5F:5C:61:6E]  3.085ms
Unicast reply from 10.10.10.1 [4A:78:5F:5C:61:6E]  3.293ms
````
### TUN
Интерфейс tun0 поднят и работает.
````
[root@client ~]# ip -c a s dev tun0
3: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
    link/none
    inet 10.10.10.2/24 brd 10.10.10.255 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::f337:2724:1e84:4669/64 scope link flags 800
       valid_lft forever preferred_lft forever
 ````
 Но утилита arping ведет себя следующим образом:
 ````
 [root@client ~]# sudo arping  10.10.10.1 -I tun0
arping: Device tun0 not available.
````
Обьяснение тому что TUN  передаёт только IP-пакеты уровня 3 и широковещательный трафик обычно не передается, а ARP это уровень 2.
## TAP vs TUN
TAP:
- ведет себя как настоящий сетевой адаптер (за исключением виртуального сетевого адаптера)
- может передавать любые сетевые протоколы (IPv4, IPv6, Netalk, IPX и т. д.)
- Работает на уровне 2, то есть кадры Ethernet передаются через туннель VPN
- Может использоваться в мостовых соединениях
- требует гораздо больше накладных расходов на широковещательную передачу в туннеле VPN
- добавляет накладные расходы заголовков Ethernet на все пакеты, транспортируемые через туннель VPN
- плохо масштабируется

TUN:
- меньшие накладные расходы трафика, передаёт только трафик, предназначенный для VPN-клиента
- передаёт только IP-пакеты уровня 3
- широковещательный трафик обычно не передается
- может передавать только IPv4 (OpenVPN 2.3 добавляет потдержку IPv6)
- не может использоваться в мостовых соединениях.

## RAS
Устанавливаем сервер:
````
[root@server ~]# yum install -y epel-release
[root@server ~]# yum install -y openvpn easy-rsa policycoreutils-python
````
Инициализация PKI:
````
[root@server ~]# cd /etc/openvpn/
[root@server openvpn]# /usr/share/easy-rsa/3.0.6/easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/pki
````
Генерируем ключи для сервера:
````
[root@server openvpn]# /usr/share/easy-rsa/3.0.6/easyrsa build-ca nopass
[root@server openvpn]# /usr/share/easy-rsa/3.0.6/easyrsa gen-dh
[root@server openvpn]# /usr/share/easy-rsa/3.0.6/easyrsa build-server-full server nopass
[root@server openvpn]# openvpn --genkey --secret ta.key

[root@server openvpn]# mkdir keys
[root@server openvpn]# cp ta.key keys/
[root@server openvpn]# cp pki/ca.crt keys/
[root@server openvpn]# cp pki/dh.pem keys/
[root@server openvpn]# cp pki/issued/server.crt keys/
[root@server openvpn]# cp pki/private/server.key keys/
````
Генерируем сертификат для клиента:
````
[root@server openvpn]# /usr/share/easy-rsa/3.0.6/easyrsa build-client-full client nopass
````
Конфиг сервера:
````
proto udp
dev tun

ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/server.crt
key /etc/openvpn/keys/server.key 
dh /etc/openvpn/keys/dh.pem

server 10.10.10.0 255.255.255.0
route 192.168.10.0 255.255.255.0
push "route 192.168.10.0 255.255.255.0"
ifconfig-pool-persist ipp.txt
client-to-client
client-config-dir /etc/openvpn/client

keepalive 10 120
tls-auth /etc/openvpn/keys/ta.key 0
key-direction 0
cipher AES-256-CBC

persist-key
persist-tun

status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3
````
Запускаем openvpn сервер:
````
[root@server ~]# systemctl enable --now openvpn@server
````
Конфиг клиента:
````
dev tun
proto udp
remote 192.168.10.10
client
resolv-retry infinite

ca ./ca.crt
cert ./client.crt
key ./client.key

route 192.168.10.0 255.255.255.0

persist-key
persist-tun

verb 3
````
Подключаемся к серверу:
````
┌─[✗]─[san4ez@edukation]─[~]
└──╼ $sudo openvpn --config client.conf
Wed Jul  21 12:35:26 2021 OpenVPN 2.4.8 [git:makepkg/3976acda9bf10b5e+] x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Oct 30 2021
Wed Jul  21 12:35:26 2021 library versions: OpenSSL 1.1.1d  10 Sep 2021, LZO 2.10
Wed Jul  21 12:35:26 2021 Outgoing Control Channel Authentication: Using 160 bit message hash 'SHA1' for HMAC authentication
Wed Jul  21 12:35:26 2021 Incoming Control Channel Authentication: Using 160 bit message hash 'SHA1' for HMAC authentication
Wed Jul  21 12:35:26 2021 TCP/UDP: Preserving recently used remote address: [AF_INET]192.168.10.10:1194
Wed Jul  21 12:35:26 2021 Socket Buffers: R=[212992->212992] S=[212992->212992]
Wed Jul  21 12:35:26 2021 UDP link local (bound): [AF_INET][undef]:1194
Wed Jul  21 12:35:26 2021 UDP link remote: [AF_INET]192.168.10.10:1194
Wed Jul  21 12:35:26 2021 TLS: Initial packet from [AF_INET]192.168.10.10:1194, sid=0311008a 6342c53d
Wed Jul  21 12:35:26 2021 VERIFY OK: depth=1, CN=rasvpn
Wed Jul  21 12:35:26 2021 VERIFY KU OK
Wed Jul  21 12:35:26 2021 Validating certificate extended key usage
Wed Jul  21 12:35:26 2021 ++ Certificate has EKU (str) TLS Web Server Authentication, expects TLS Web Server Authentication
Wed Jul  21 12:35:26 2021 VERIFY EKU OK
Wed Jul  21 12:35:26 2021 VERIFY OK: depth=0, CN=server
Wed Jul  21 12:35:26 2021 [server] Peer Connection Initiated with [AF_INET]192.168.10.10:1194
Wed Jul  21 12:35:48 2021 PUSH: Received control message: 'PUSH_REPLY,route 192.168.10.0 255.255.255.0,route 10.10.10.0 255.255.255.0,topology net30,ping 10,ping-restart 120,ifconfig 10.10.10.6 10.10.10.5,peer-id 0,cipher AES-256-GCM'
...
Wed Jul  21 12:35:48 2021 Initialization Sequence Completed
````
Проверяем:
````
┌─[san4ez@edukation]─[~]
└──╼ $ip -c a s dev tun0
5: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 100
    link/none 
    inet 10.10.10.6 peer 10.10.10.5/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::8869:d771:c935:18d5/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever

┌─[san4ez@edukation]─[~]
└──╼ $ip r | grep tun
10.10.10.0/24 via 10.10.10.5 dev tun0 
10.10.10.5 dev tun0 proto kernel scope link src 10.10.10.6 

┌─[san4ez@education]─[~]
└──╼ $ping -c 4 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
▁▁▁▁
 0/  4 ( 0%) lost;    0/   0/   0ms; last:    0ms
 0/  4 ( 0%) lost;    0/   0/   0/   0ms (last 4)
--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3026ms
rtt min/avg/max/mdev = 0.477/0.564/0.708/0.088 ms
````
