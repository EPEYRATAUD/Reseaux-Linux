# I. ARP
## 1. Echange ARP

### Ping entre les deux machines 

```
[targa@node1 ~]$ ping 10.2.1.12
PING 10.2.1.12 (10.2.1.12) 56(84) bytes of data.
64 bytes from 10.2.1.12: icmp_seq=1 ttl=64 time=0.674 ms## Node1
64 bytes from 10.2.1.12: icmp_seq=2 ttl=64 time=0.748 ms
64 bytes from 10.2.1.12: icmp_seq=3 ttl=64 time=0.688 ms
64 bytes from 10.2.1.12: icmp_seq=4 ttl=64 time=0.812 ms
^C
--- 10.2.1.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3080ms
rtt min/avg/max/mdev = 0.674/0.730/0.812/0.060 ms
```


```
[targa@node2 ~]$ [targa@node2 ~]$ ping 10.2.1.11
PING 10.2.1.11 (10.2.1.11) 56(84) bytes of data.
64 bytes from 10.2.1.11: icmp_seq=1 ttl=64 time=0.414 ms
64 bytes from 10.2.1.11: icmp_seq=2 ttl=64 time=0.696 ms
64 bytes from 10.2.1.11: icmp_seq=3 ttl=64 time=0.787 ms
^C
--- 10.2.1.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2081ms
rtt min/avg/max/mdev = 0.414/0.632/0.787/0.160 ms
```

### Table ARP

```
[targa@node1 ~]$ ip n s
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:60 DELAY
10.2.1.12 dev enp0s8 lladdr 08:00:27:fc:83:9a STALE
```

```
[targa@node2 ~]$ ip n s
10.2.1.11 dev enp0s8 lladdr 08:00:27:df:3b:1f STALE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:60 DELAY
```

* Pour vérifier les adresses MAC, on effectue un ``` ip a ``` aux deux machines : 

#### Node1
```
[targa@node1 ~]$ ip a
[...]
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:df:3b:1f brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.11/24 brd 10.2.1.255 scope global noprefixroute enp0s8
    [...]

```
#### Node2

```
[targa@node2 ~]$ ip a
[...]
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:fc:83:9a brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.12/24 brd 10.2.1.255 scope global noprefixroute enp0s8
    [...]
```
* Les adresses MAC ```08:00:27:df:3b:1f``` et ```08:00:27:fc:83:9a ``` correspondent bien aux tables ARP.

## 2. Analyse de trames


* tcpdump
```
[targa@node1 ~]$ sudo tcpdump -i enp0s8 -c 10 -w mon_fichier.pcap not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```
* ping de node2 vers node1

```
[targa@node2 ~]$ ping 10.2.1.11
PING 10.2.1.11 (10.2.1.11) 56(84) bytes of data.
64 bytes from 10.2.1.11: icmp_seq=1 ttl=64 time=0.526 ms
64 bytes from 10.2.1.11: icmp_seq=2 ttl=64 time=0.772 ms
64 bytes from 10.2.1.11: icmp_seq=3 ttl=64 time=0.795 ms
64 bytes from 10.2.1.11: icmp_seq=4 ttl=64 time=0.844 ms
64 bytes from 10.2.1.11: icmp_seq=5 ttl=64 time=0.667 ms
64 bytes from 10.2.1.11: icmp_seq=6 ttl=64 time=0.400 ms
64 bytes from 10.2.1.11: icmp_seq=7 ttl=64 time=0.512 ms
64 bytes from 10.2.1.11: icmp_seq=8 ttl=64 time=0.777 ms
64 bytes from 10.2.1.11: icmp_seq=9 ttl=64 time=0.792 ms
^C
--- 10.2.1.11 ping statistics ---
9 packets transmitted, 9 received, 0% packet loss, time 8181ms
rtt min/avg/max/mdev = 0.400/0.676/0.844/0.149 ms
```
* récuperation du fichier pcap en lançant un serveur sur le port 8888

```
[targa@node1 ~]$ python3 -m http.server 8888
Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
10.2.1.1 - - [20/Sep/2021 17:43:03] "GET / HTTP/1.1" 200 -
10.2.1.1 - - [20/Sep/2021 17:43:07] "GET /mon_fichier.pcap HTTP/1.1" 200 -
```
![](https://i.imgur.com/qKWLGMi.png)

| ordre | type trame  | source                    | destination                 |
|-------|-------------|-------------------------- |---------------------------- |
| 1     | Requête ARP |`node1` `08:00:27:df:3b:1f`| Broadcast `FF:FF:FF:FF:FF`  |
| 2     | Réponse ARP |`node2` `08:00:27:fc:83:9a`| `node1` `08:00:27:df:3b:1f` |
|       |             |                           |                             |



# II. Routage

## 1. Mise en place du routage
 * On active le routage sur la machine ```router```

 ```
 [targa@router ~]$ sudo firewall-cmd --get-active-zone
public
  interfaces: enp0s3 enp0s8 enp0s9
[targa@router ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
success
 ```

* Création du fichier ``` route-enp0s8``` dans ``` /etc/sysconfig/network-scripts/```

* Pour chaque machine, on ajoute le réseau de la machine distant ainsi que le gateway de la machine de base.

* Première machine
```
10.2.2.0/24 via 10.2.1.254 dev enp0s8
```
* Deuxième machine

```
10.2.1.0/24 via 10.2.2.254 dev enp0s8
```

* Test de ping des deux côtés
```
PING 10.2.2.12 (10.2.2.12) 56(84) bytes of data.
64 bytes from 10.2.2.12: icmp_seq=1 ttl=63 time=0.735 ms
64 bytes from 10.2.2.12: icmp_seq=2 ttl=63 time=1.21 ms
64 bytes from 10.2.2.12: icmp_seq=3 ttl=63 time=1.66 ms
64 bytes from 10.2.2.12: icmp_seq=4 ttl=63 time=1.78 ms
^C
--- 10.2.2.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3023ms
rtt min/avg/max/mdev = 0.735/1.343/1.775/0.412 ms
```

```
[targa@node1 network-scripts]$ ping 10.2.2.12
PING 10.2.2.12 (10.2.2.12) 56(84) bytes of data.
64 bytes from 10.2.2.12: icmp_seq=1 ttl=63 time=0.927 ms
64 bytes from 10.2.2.12: icmp_seq=2 ttl=63 time=1.55 ms
64 bytes from 10.2.2.12: icmp_seq=3 ttl=63 time=1.57 ms
64 bytes from 10.2.2.12: icmp_seq=4 ttl=63 time=1.63 ms
^C
--- 10.2.2.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3067ms
rtt min/avg/max/mdev = 0.927/1.419/1.634/0.285 ms
```

## 2. Analyse de trames

* Vider la table ARP des trois machines.

```
[targa@router network-scripts]$ sudo ip neigh flush all
[targa@node1 network-scripts]$ sudo ip neigh flush all
[targa@marcel network-scripts]$ sudo ip neigh flush all
```
* ping de node1 vers marcel 

```
[targa@node1 network-scripts]$ ping 10.2.2.12
PING 10.2.2.12 (10.2.2.12) 56(84) bytes of data.
64 bytes from 10.2.2.12: icmp_seq=1 ttl=63 time=0.863 ms
64 bytes from 10.2.2.12: icmp_seq=2 ttl=63 time=0.905 ms
64 bytes from 10.2.2.12: icmp_seq=3 ttl=63 time=1.65 ms
64 bytes from 10.2.2.12: icmp_seq=4 ttl=63 time=1.56 ms
^C
--- 10.2.2.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3049ms
rtt min/avg/max/mdev = 0.863/1.243/1.645/0.360 ms
```
* Table ARP des trois machines

```
[targa@node1 network-scripts]$ ip neigh show
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:60 REACHABLE
10.2.1.254 dev enp0s8 lladdr 08:00:27:83:39:16 STALE
```
* Le ping passe par la passerelle de node1 : ```10.2.1.254```
```
[targa@router network-scripts]$ ip neigh show
10.2.2.12 dev enp0s9 lladdr 08:00:27:6f:64:53 DELAY
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 DELAY
10.2.1.11 dev enp0s8 lladdr 08:00:27:df:3b:1f REACHABLE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:60 DELAY
```
* A expliquer plus tard

```
[targa@marcel network-scripts]$ ip neigh show
10.2.2.1 dev enp0s8 lladdr 0a:00:27:00:00:66 DELAY
10.2.2.254 dev enp0s8 lladdr 08:00:27:cb:46:86 REACHABLE
```
* Le pong passe par la passerelle de marcel : ```10.2.2.254 ```

| ordre | type trame  | IP source | MAC source                | IP destination | MAC destination            |
|-------|-------------|-----------|---------------------------|----------------|----------------------------|
| 1     | Requête ARP | x         | `node1` `08:00:27:df:3b:1f`  | x              | Broadcast `08:00:27:83:39:16` |
| 2     | Réponse ARP | x         | `marcel` `08:00:27:6f:64:53` | x              | `node1` `08:00:27:df:3b:1f`   |
| ...   | ...         | ...       | ...                       |                |                            |
| 1     | Ping        |`Broadcast` `10.2.1.254`         | `Broadcast` `08:00:27:83:39:16`                         | `node1` `10.2.1.11`              | `node1` `08:00:27:df:3b:1f`                          |
| 2     | Pong        | `node1` `10.2.1.11`           | `node1` `08:00:27:df:3b:1f`                         | `Broadcast` `10.2.1.254`              |`Broadcast` `08:00:27:83:39:16`                           |
| 1     | Ping        | `node1` `10.2.1.11`           | `node1` `08:00:27:df:3b:1f`                         | `Broadcast` `10.2.1.254`              |`Broadcast` `08:00:27:83:39:16`                           |
| 2     | Pong        |`Broadcast` `10.2.1.254`         | `Broadcast` `08:00:27:83:39:16`                         | `node1` `10.2.1.11`              | `node1` `08:00:27:df:3b:1f`                          |
| 1     | Ping        |`marcel` `10.2.2.12`         |`marcel` `08:00:27:6f:64:53`                        | `Broadcast` `10.2.2.254`               | `Broadcast` `08:00:27:cb:46:86`                          |
| 2     | Pong        |  `Broadcast` `10.2.2.254`         | `Broadcast` `08:00:27:cb:46:86`                         | `marcel` `10.2.2.12`               | `marcel` `08:00:27:6f:64:53`                          |


## 3. Accès internet

* Modification du fichier ```network ``` dans ```/etc/sysconfig/ ``` en ajoutant la ligne suivante :

#### node1
```
GATEWAY=10.2.1.254
```
#### marcel
```
GATEWAY=10.2.2.254
```



* Ping vers ```google.com ``` par l'adresse ```8.8.8.8 ```

#### node1 
```
[targa@node1 sysconfig]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=18.9 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=23.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=20.1 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 18.889/20.796/23.364/1.889 ms
```


#### marcel

```
[targa@marcel sysconfig]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=22.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=22.7 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=21.8 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 21.835/22.420/22.754/0.432 ms
```

* Modification du fichier ```/etc/resolv.conf ```

* Ajout de la ligne :


```
nameserver 1.1.1.1
```

* curl avec node1 vers ```google.com ```
```
</BODY></HTML>
[targa@node1 ~]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

* dig avec node1 vers ```google.com```

```
[targa@node1 etc]$ dig google.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14239
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             212     IN      A       172.217.19.238

;; Query time: 25 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Sat Sep 25 10:37:06 CEST 2021
;; MSG SIZE  rcvd: 55

```

* curl avec marcel vers ```google.com ```

```
[targa@marcel ~]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```


* dig avec marcel vers ```google.com```

```
[targa@marcel ~]$ dig google.com
; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 261
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             61      IN      A       216.58.209.238

;; Query time: 26 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Sat Sep 25 10:39:52 CEST 2021
;; MSG SIZE  rcvd: 55
```

* ping de node1 vers ```google.com```

```
[targa@node1 etc]$ ping google.com
PING google.com (216.58.209.238) 56(84) bytes of data.
64 bytes from par10s29-in-f14.1e100.net (216.58.209.238): icmp_seq=1 ttl=112 time=23.1 ms
64 bytes from par10s29-in-f14.1e100.net (216.58.209.238): icmp_seq=2 ttl=112 time=23.2 ms
64 bytes from par10s29-in-f14.1e100.net (216.58.209.238): icmp_seq=3 ttl=112 time=23.2 ms
64 bytes from par10s29-in-f14.1e100.net (216.58.209.238): icmp_seq=4 ttl=112 time=23.1 ms
^C
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 23.074/23.133/23.181/0.158 ms
```

* ping de marcel vers ```google.com```

```
[targa@marcel ~]$ ping google.com
PING google.com (216.58.198.206) 56(84) bytes of data.
64 bytes from par10s27-in-f206.1e100.net (216.58.198.206): icmp_seq=1 ttl=113 time=22.9 ms
64 bytes from par10s27-in-f206.1e100.net (216.58.198.206): icmp_seq=2 ttl=113 time=22.7 ms
64 bytes from par10s27-in-f206.1e100.net (216.58.198.206): icmp_seq=3 ttl=113 time=21.8 ms
^C
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 21.814/22.470/22.926/0.506 ms
```

### Analyse de trames

* ping de node1 vers ```8.8.8.8 ```

```
[targa@node1 etc]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=24.0 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=22.6 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=24.7 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=113 time=24.7 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=113 time=24.10 ms
[...]
```

* Capture avec tcpdump

```
[targa@node1 ~]$ sudo tcpdump -i enp0s8 -c 100 -w tp2_routage_internet.pcap
100 packets captured
100 packets received by filter
0 packets dropped by kernel
```

* Récupération du fichier 

```
[targa@node1 ~]$ python3 -m http.server 8888
Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
10.2.1.1 - - [25/Sep/2021 11:08:52] "GET / HTTP/1.1" 200 -
10.2.1.1 - - [25/Sep/2021 11:08:52] code 404, message File not found
10.2.1.1 - - [25/Sep/2021 11:08:52] "GET /favicon.ico HTTP/1.1" 404 -
10.2.1.1 - - [25/Sep/2021 11:08:58] "GET /mon_fichier.pcap HTTP/1.1" 200 -

```


| ordre | type trame | IP source           | MAC source               | IP destination | MAC destination |     |
|-------|------------|---------------------|--------------------------|----------------|-----------------|-----|
| 1     | ping       | `node1` `10.2.1.12` | `node1` `08:00:27:df:3b:1f` | `google.com` `8.8.8.8`      | `google.com` `08:00:27:83:39:16`               |     |
| 2     | pong       | `google.com` `8.8.8.8`                |`google.com` `08:00:27:83:39:16`                      | `node1` `10.2.1.12`         | `node1` `08:00:27:df:3b:1f`             | ... |


# III. DHCP

## 1. Mise en place du serveur DHCP

* création d'un serveur DHCP sur node1.

#### update de la machine
```
[targa@node1 network-scripts]$ sudo dnf update
```

#### installation du serveur DHCP
```
[targa@node1 network-scripts]$ sudo dnf install dhcp-server
```

#### création d'un backup du fichier de configuration ``` dhcpd.conf```

```
[targa@node1 ~]$ sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.confl.bak
```


#### modification du fichier de configuration ```dhcpd.conf``` dans ```/etc/dhcp/ ``` 
```
sudo nano /etc/dhcp/dhcpd.conf
```

![](https://i.imgur.com/PMedv5K.png)


#### lancement du serveur dhcp
```
[targa@node1 etc]$ sudo systemctl start dhcpd
[targa@node1 etc]$ sudo systemctl enable dhcpd
```

 #### vérification de l'état du serveur 
```
[targa@node1 etc]$ sudo systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2021-09-26 11:26:13 CEST; 20s ago
     Docs: man:dhcpd(8)
           man:dhcpd.conf(5)
 Main PID: 38691 (dhcpd)
   Status: "Dispatching packets..."
    Tasks: 1 (limit: 18054)
   Memory: 5.6M
   CGroup: /system.slice/dhcpd.service
           └─38691 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Sep 26 11:26:13 node1.net1.tp2 dhcpd[38691]: Database file: /var/lib/dhcpd/dhcpd.leases
Sep 26 11:26:13 node1.net1.tp2 dhcpd[38691]: PID file: /var/run/dhcpd.pid
Sep 26 11:26:13 node1.net1.tp2 dhcpd[38691]: Source compiled to use binary-leases
Sep 26 11:26:13 node1.net1.tp2 dhcpd[38691]: Wrote 0 leases to leases file.
Sep 26 11:26:13 node1.net1.tp2 dhcpd[38691]: Listening on LPF/enp0s8/08:00:27:df:3b:1f/10.2.1.0/24
Sep 26 11:26:13 node1.net1.tp2 dhcpd[38691]: Sending on   LPF/enp0s8/08:00:27:df:3b:1f/10.2.1.0/24
Sep 26 11:26:13 node1.net1.tp2 dhcpd[38691]: Sending on   Socket/fallback/fallback-net
Sep 26 11:26:13 node1.net1.tp2 dhcpd[38691]: Server starting service.
Sep 26 11:26:13 node1.net1.tp2 systemd[1]: Started DHCPv4 Server Daemon.
Sep 26 11:26:33 node1.net1.tp2 dhcpd[38691]: DHCPDISCOVER from 08:00:27:fc:83:9a via enp0s8
```

#### ouverture du port udp 67 pour le serveur DHCP
```
[targa@node1 etc]$ sudo firewall-cmd --add-port=67/udp --permanent
[sudo] password for targa:
success
```

## Node2 client

#### Update de la machine

```
[targa@node2 network-scripts]$ sudo dnf update
```

#### Modification du fichier fichier de configuration de la carte enp0s8 ```ifcfg-enp0s8 ``` en prenant le chemin ```cd /etc/sysconfig/network-scripts/```


![](https://i.imgur.com/fKm72cu.png)


#### Demande d'adresse ip avec la machine cliente (node2).
```
[targa@node2 ~]$ sudo dhclient epn0s8
```

```
[targa@node2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:fc:83:9a brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.12/24 brd 10.2.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 159sec preferred_lft 159sec
    inet6 fe80::a00:27ff:fefc:839a/64 scope link
       valid_lft forever preferred_lft forever
```


### côté node1

```
[targa@node1 etc]$ sudo systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2021-09-26 11:26:13 CEST; 4h 54min ago
     Docs: man:dhcpd(8)
           man:dhcpd.conf(5)
 Main PID: 38691 (dhcpd)
   Status: "Dispatching packets..."
    Tasks: 1 (limit: 18054)
   Memory: 5.6M
   CGroup: /system.slice/dhcpd.service
           └─38691 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Sep 26 16:05:00 node1.net1.tp2 dhcpd[38691]: DHCPREQUEST for 10.2.1.12 from 08:00:27:fc:83:9a (node2) via enp0s8
Sep 26 16:05:00 node1.net1.tp2 dhcpd[38691]: DHCPACK on 10.2.1.12 to 08:00:27:fc:83:9a via enp0s8
Sep 26 16:06:13 node1.net1.tp2 dhcpd[38691]: DHCPREQUEST for 10.2.1.12 from 08:00:27:fc:83:9a via enp0s8
Sep 26 16:06:13 node1.net1.tp2 dhcpd[38691]: DHCPACK on 10.2.1.12 to 08:00:27:fc:83:9a (node2) via enp0s8
Sep 26 16:12:11 node1.net1.tp2 dhcpd[38691]: DHCPREQUEST for 10.2.1.12 from 08:00:27:fc:83:9a (node2) via enp0s8
Sep 26 16:12:11 node1.net1.tp2 dhcpd[38691]: DHCPACK on 10.2.1.12 to 08:00:27:fc:83:9a via enp0s8
Sep 26 16:13:43 node1.net1.tp2 dhcpd[38691]: DHCPREQUEST for 10.2.1.12 from 08:00:27:fc:83:9a via enp0s8
Sep 26 16:13:43 node1.net1.tp2 dhcpd[38691]: DHCPACK on 10.2.1.12 to 08:00:27:fc:83:9a (node2) via enp0s8
Sep 26 16:18:12 node1.net1.tp2 dhcpd[38691]: DHCPREQUEST for 10.2.1.12 from 08:00:27:fc:83:9a (node2) via enp0s8
Sep 26 16:18:12 node1.net1.tp2 dhcpd[38691]: DHCPACK on 10.2.1.12 to 08:00:27:fc:83:9a via enp0s8
```

* le serveur dhcp adresse bien une adresse ip dynamiquement à node2.


#### ping de node2 vers la passerelle ( route par défaut )

```
[targa@node2 ~]$ ping 10.2.1.254
PING 10.2.1.254 (10.2.1.254) 56(84) bytes of data.
64 bytes from 10.2.1.254: icmp_seq=1 ttl=64 time=0.839 ms
64 bytes from 10.2.1.254: icmp_seq=2 ttl=64 time=1.15 ms
64 bytes from 10.2.1.254: icmp_seq=3 ttl=64 time=0.641 ms
64 bytes from 10.2.1.254: icmp_seq=4 ttl=64 time=0.702 ms
^C
--- 10.2.1.254 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3063ms
rtt min/avg/max/mdev = 0.641/0.833/1.151/0.198 ms
```
### vérification de la route par défaut

```
[targa@node2 ~]$ ip r s
default via 10.2.1.254 dev enp0s8 proto dhcp metric 100
10.2.1.0/24 dev enp0s8 proto kernel scope link src 10.2.1.12 metric 100
```
#### vérification du serveur DNS avec la commande ``` dig ```

```
[targa@node2 ~]$ dig google.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35256
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             153     IN      A       216.58.198.206

;; Query time: 25 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Sun Sep 26 12:21:57 CEST 2021
;; MSG SIZE  rcvd: 55
```

#### ping vers un nom de domaine

```
[targa@node2 ~]$ ping google.com
PING google.com (216.58.209.238) 56(84) bytes of data.
64 bytes from par10s29-in-f238.1e100.net (216.58.209.238): icmp_seq=1 ttl=112 time=22.1 ms
64 bytes from par10s29-in-f238.1e100.net (216.58.209.238): icmp_seq=2 ttl=112 time=22.2 ms
64 bytes from par10s29-in-f238.1e100.net (216.58.209.238): icmp_seq=3 ttl=112 time=22.5 ms
64 bytes from par10s29-in-f238.1e100.net (216.58.209.238): icmp_seq=4 ttl=112 time=22.8 ms
^C
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 22.056/22.404/22.798/0.324 ms
```
### 2. Analyse de trames

* vidage des tables ARP de node1 et node2.

```
[targa@node1 etc]$ sudo ip neigh flush all
[targa@node2 ~]$ [targa@node2 ~]$ ip neigh show
```

* rupture du bail dhcp de node 2 pour qu'il puisse demander une nouvelle IP

```
[targa@node2 ~]$ sudo dhclient -r
[sudo] password for targa:
Killed old client process
```

![](https://i.imgur.com/C6a2w2A.png)


* Capture avec tcpdump

```
[targa@node1 ~]$ sudo tcpdump -i enp0s8 -c 200 -w tp2_dhcp.pcap
[sudo] password for targa:
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
200 packets captured
223 packets received by filter
0 packets dropped by kernel
```
![](https://i.imgur.com/1RRvSdp.png)
![](https://i.imgur.com/T47HwCK.png)

| ordre | type trame  | IP source | MAC source                | IP destination | MAC destination            |
|-------|-------------|-----------|---------------------------|----------------|----------------------------|
| 1     |  DHCP | `0.0.0.0`         | `node2` `08:00:27:fc:83:9a`   | `255.255.255.255`              | Broadcast `FF:FF:FF:FF:FF` |
| 2     |  ARP | x         | `node1` `08:00:27:df:3b:1f` | x              | Broadcast `FF:FF:FF:FF:FF`   |
|                 |                            |
| 1     |DHCP       |`node1` `10.2.1.11`         | `node1` `08:00:27:df:3b:1f` | `node2` `10.2.1.12`              | `node2` `08:00:27:fc:83:9a`                          |
| 2     | DHCP       | `0.0.0.0`           | `node2` `08:00:27:fc:83:9a`  | `255.255.255.255`  |Broadcast `FF:FF:FF:FF:FF` |
| 1     | DHCP        | `node1` `10.2.1.11`           | `node1` `08:00:27:df:3b:1f` | `node2` `10.2.1.12`              | `node2` `08:00:27:fc:83:9a` |
| 2     |  ARP | x         | `node2` `08:00:27:fc:83:9a` | x              | Broadcast `FF:FF:FF:FF:FF`   |                         |
| 2     |  ARP | x         | `node1` `08:00:27:df:3b:1f` | x              | Broadcast `FF:FF:FF:FF:FF`   |
| 2     |  ARP | x         | `node2` `08:00:27:fc:83:9a` | x              | Broadcast `FF:FF:FF:FF:FF`   |  
| 2     |  ARP | x         | `node1` `08:00:27:df:3b:1f` | x              | Broadcast `FF:FF:FF:FF:FF`   |
| 2     |  ARP | x         | `node2` `08:00:27:fc:83:9a` | x              | Broadcast `FF:FF:FF:FF:FF`   |
| 2     |  ARP | x         | `node1` `08:00:27:df:3b:1f` | x              | `node1` `08:00:27:fc:83:9a`|
| 1     | DHCP      |`node2` `10.2.1.12`|`node2` `08:00:27:fc:83:9a`| `node1` `10.2.1.11` | `node1` `08:00:27:df:3b:1f`   |
| 1     |DHCP       |`node1` `10.2.1.11`         | `node1` `08:00:27:df:3b:1f` | `node2` `10.2.1.12`              | `node2` `08:00:27:fc:83:9a`                          |
| 2     |  ARP | x         | `node1` `08:00:27:df:3b:1f`| x              | `node2` `08:00:27:fc:83:9a`|
| 2     |  ARP | x         | `node2` `08:00:27:fc:83:9a` | x              |`node1` `08:00:27:df:3b:1f`   |
| 2     |  ARP | x         | `node2` `08:00:27:fc:83:9a` | x              |  Broadcast `FF:FF:FF:FF:FF`  |