## Virtual bridge between Network namespaces
### 1. The [Network namespaces](https://en.wikipedia.org/wiki/Linux_namespaces#Network_(net)) are added
\# `ip netns add ns0`\
\# `ip netns add ns1`\
\# `ip netns add ns2`\
\# `ip netns add ns3`

>Each namespace will have a private set of IP addresses, its own routing table, socket listing, connection tracking table, firewall, and other network-related resources.

*To verify:*\
\# `ip netns list`\
<span style="font-family: monospace">
&ensp;ns3\
&ensp;ns2\
&ensp;ns1\
&ensp;ns0
</span>

---
### 2. Virtual bridge is added
\# `ip link add vbr0 type bridge`

*To verify:*\
\# `ip link show vbr0`\
<span style="font-family: monospace">
&ensp;4: vbr0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 82:a2:4b:2f:e3:96 brd ff:ff:ff:ff:ff:ff
</span>

---
### 3. Virtual interfaces are added
\# `ip link add veth0a type veth peer name veth0b`\
\# `ip link add veth1a type veth peer name veth1b`\
\# `ip link add veth2a type veth peer name veth2b`\
\# `ip link add veth3a type veth peer name veth3b`

*To verify:*\
\# `ip link show type veth`\
<span style="font-family: monospace">
&ensp;5: veth0b@veth0a: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 6e:43:02:72:f4:bb brd ff:ff:ff:ff:ff:ff\
&ensp;6: veth0a@veth0b: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 52:0c:87:cf:05:e9 brd ff:ff:ff:ff:ff:ff\
&ensp;7: veth1b@veth1a: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 72:7c:99:bc:12:89 brd ff:ff:ff:ff:ff:ff\
&ensp;8: veth1a@veth1b: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether c2:ec:73:be:13:db brd ff:ff:ff:ff:ff:ff\
&ensp;9: veth2b@veth2a: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 1a:b2:64:c9:fc:eb brd ff:ff:ff:ff:ff:ff\
&ensp;10: veth2a@veth2b: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether fa:51:e9:5e:a9:86 brd ff:ff:ff:ff:ff:ff\
&ensp;11: veth3b@veth3a: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether e2:1b:cb:9d:b0:94 brd ff:ff:ff:ff:ff:ff\
&ensp;12: veth3a@veth3b: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 32:71:95:23:69:48 brd ff:ff:ff:ff:ff:ff
</span>

---
### 4. Assign the interfaces

>Interfaces ending with **a** are assigned to the corresponding namespaces, their counterparts ending with **b** are assigned to the bridge.\
To add an interface to the bridge, its state must be **up**.

\# `ip link set vbr0 up`

\# `ip link set veth0a netns ns0`\
\# `ip link set veth1a netns ns1`\
\# `ip link set veth2a netns ns2`\
\# `ip link set veth3a netns ns3`

\# `ip link set veth0b master vbr0`\
\# `ip link set veth1b master vbr0`\
\# `ip link set veth2b master vbr0`\
\# `ip link set veth3b master vbr0`

*To verify:*\
\# `for type in bridge veth; do ip link show type $type; done`\
<span style="font-family: monospace">
&ensp;4: vbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 1a:b2:64:c9:fc:eb brd ff:ff:ff:ff:ff:ff\
&ensp;5: veth0b@if6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop master vbr0 state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 6e:43:02:72:f4:bb brd ff:ff:ff:ff:ff:ff link-netns ns0\
&ensp;7: veth1b@if8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop master vbr0 state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 72:7c:99:bc:12:89 brd ff:ff:ff:ff:ff:ff link-netns ns1\
&ensp;9: veth2b@if10: <BROADCAST,MULTICAST> mtu 1500 qdisc noop master vbr0 state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 1a:b2:64:c9:fc:eb brd ff:ff:ff:ff:ff:ff link-netns ns2\
&ensp;11: veth3b@if12: <BROADCAST,MULTICAST> mtu 1500 qdisc noop master vbr0 state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether e2:1b:cb:9d:b0:94 brd ff:ff:ff:ff:ff:ff link-netns ns3
</span>

\# `for i in {0..3}; do ip -n ns${i} link show type veth; done`\
<span style="font-family: monospace">
&ensp;6: veth0a@if5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 52:0c:87:cf:05:e9 brd ff:ff:ff:ff:ff:ff link-netnsid 0\
&ensp;8: veth1a@if7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether c2:ec:73:be:13:db brd ff:ff:ff:ff:ff:ff link-netnsid 0\
&ensp;10: veth2a@if9: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether fa:51:e9:5e:a9:86 brd ff:ff:ff:ff:ff:ff link-netnsid 0\
&ensp;12: veth3a@if11: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 32:71:95:23:69:48 brd ff:ff:ff:ff:ff:ff link-netnsid 0
</span>

> The peers of interfaces are now referencing each other by their dynamic numbering in the output, as they are in a different namespace.

---
### 5. Assign IP addresses to interfaces in the namespaces

\# `ip netns exec ns0 ip address add dev veth0a 192.168.60.0/24`\
\# `ip netns exec ns1 ip address add dev veth1a 192.168.60.1/24`\
\# `ip netns exec ns2 ip address add dev veth2a 192.168.60.2/24`\
\# `ip netns exec ns3 ip address add dev veth3a 192.168.60.3/24`

&emsp;OR

\# `for i in {0..3}; do ip -n ns${i} address add dev veth${i}a 192.168.60.${i}/24; done`

>Additionally, all interfaces must be set **up**.

\# `ip -n ns0 link set veth0a up`\
\# `ip -n ns1 link set veth1a up`\
\# `ip -n ns2 link set veth2a up`\
\# `ip -n ns3 link set veth3a up`

\# `ip link set veth0b up`\
\# `ip link set veth1b up`\
\# `ip link set veth2b up`\
\# `ip link set veth3b up`

&emsp;OR

\# `for i in {0..3}; do ip -n ns${i} link set veth${i}a up; ip link set veth${i}b up; done`

*To verify:*\
\# `for i in {0..3}; do ip -n ns${i} address show type veth; done`\
<span style="font-family: monospace">
&ensp;6: veth0a@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 52:0c:87:cf:05:e9 brd ff:ff:ff:ff:ff:ff link-netnsid 0\
&emsp;&emsp;&emsp;inet 192.168.60.0/24 scope global veth0a\
&emsp;&emsp;&emsp;&emsp;&emsp;valid_lft forever preferred_lft forever\
&emsp;&emsp;&emsp;inet6 fe80::500c:87ff:fecf:5e9/64 scope link\
&emsp;&emsp;&emsp;&emsp;&emsp;valid_lft forever preferred_lft forever\
&ensp;8: veth1a@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000\
&emsp;&emsp;&emsp;link/ether c2:ec:73:be:13:db brd ff:ff:ff:ff:ff:ff link-netnsid 0\
&emsp;&emsp;&emsp;inet 192.168.60.1/24 scope global veth1a\
&emsp;&emsp;&emsp;&emsp;&emsp;valid_lft forever preferred_lft forever\
&emsp;&emsp;&emsp;inet6 fe80::c0ec:73ff:febe:13db/64 scope link\
&emsp;&emsp;&emsp;&emsp;&emsp;valid_lft forever preferred_lft forever\
&ensp;10: veth2a@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000\
&emsp;&emsp;&emsp;link/ether fa:51:e9:5e:a9:86 brd ff:ff:ff:ff:ff:ff link-netnsid 0\
&emsp;&emsp;&emsp;inet 192.168.60.2/24 scope global veth2a\
&emsp;&emsp;&emsp;&emsp;&emsp;valid_lft forever preferred_lft forever\
&emsp;&emsp;&emsp;inet6 fe80::f851:e9ff:fe5e:a986/64 scope link\
&emsp;&emsp;&emsp;&emsp;&emsp;valid_lft forever preferred_lft forever\
&ensp;12: veth3a@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000\
&emsp;&emsp;&emsp;link/ether 32:71:95:23:69:48 brd ff:ff:ff:ff:ff:ff link-netnsid 0\
&emsp;&emsp;&emsp;inet 192.168.60.3/24 scope global veth3a\
&emsp;&emsp;&emsp;&emsp;&emsp;valid_lft forever preferred_lft forever\
&emsp;&emsp;&emsp;inet6 fe80::3071:95ff:fe23:6948/64 scope link\
&emsp;&emsp;&emsp;&emsp;&emsp;valid_lft forever preferred_lft forever
</span>

---
### 6. Final testing, [iptables](https://en.wikipedia.org/wiki/Iptables) settings and clean-up
>Analyze the network traffic that passes through the bridge:

\# `tcpdump -vvvvln -i vbr0`\
Then **ping** the interface in **ns1** (192.168.60.2) from **ns0** in a new session:\
\# `ip netns exec ns0 ping 192.168.60.1 -c 3`

>**iptables** settings: if the FORWARD chain's policy of the FILTER table is set to DROP, then the packages forwarded through the bridge are dropped. In that case the **tcpdump** output of the bridge shows no response from the target, and the **ping** fails.

<span style="font-family: monospace">
tcpdump: listening on vbr0, link-type EN10MB (Ethernet), capture size 262144 bytes<br>
&ensp;23:35:06.070008 ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 192.168.60.1 tell 192.168.60.0, length 28<br>
&ensp;23:35:06.070063 ARP, Ethernet (len 6), IPv4 (len 4), Reply 192.168.60.1 is-at c2:ec:73:be:13:db, length 28<br>
&ensp;23:35:06.070067 IP (tos 0x0, ttl 64, id 9155, offset 0, flags [DF], proto ICMP (1), length 84)<br>
&emsp;&emsp;&emsp;192.168.60.0 > 192.168.60.1: ICMP echo request, id 3181, seq 1, length 64<br>
&ensp;23:35:07.133560 IP (tos 0x0, ttl 64, id 9607, offset 0, flags [DF], proto ICMP (1), length 84)<br>
&emsp;&emsp;&emsp;192.168.60.0 > 192.168.60.1: ICMP echo request, id 3181, seq 2, length 64<br>
&ensp;23:35:08.157702 IP (tos 0x0, ttl 64, id 9692, offset 0, flags [DF], proto ICMP (1), length 84)<br>
&emsp;&emsp;&emsp;192.168.60.0 > 192.168.60.1: ICMP echo request, id 3181, seq 3, length 64<br>
</span><br>

>In order to enable the transient traffic on the bridge the FORWARD chain has to be appended temporarily with the following rule:

\# `iptables -A FORWARD -p all -i vbr0 -j ACCEPT`

>After the correction made the expected output from **tcpdump** is shown and the **ping** succeeds.

<span style="font-family: monospace">
&ensp;00:11:35.603123 IP (tos 0x0, ttl 64, id 35351, offset 0, flags [DF], proto ICMP (1), length 84)<br>
&emsp;&emsp;&emsp;192.168.60.0 > 192.168.60.1: ICMP echo request, id 3318, seq 1, length 64<br>
&ensp;00:11:35.603187 IP (tos 0x0, ttl 64, id 7661, offset 0, flags [none], proto ICMP (1), length 84)<br>
&emsp;&emsp;&emsp;192.168.60.1 > 192.168.60.0: ICMP echo reply, id 3318, seq 1, length 64<br>
&ensp;00:11:36.637667 IP (tos 0x0, ttl 64, id 35991, offset 0, flags [DF], proto ICMP (1), length 84)<br>
&emsp;&emsp;&emsp;192.168.60.0 > 192.168.60.1: ICMP echo request, id 3318, seq 2, length 64<br>
&ensp;00:11:36.637700 IP (tos 0x0, ttl 64, id 7682, offset 0, flags [none], proto ICMP (1), length 84)<br>
&emsp;&emsp;&emsp;192.168.60.1 > 192.168.60.0: ICMP echo reply, id 3318, seq 2, length 64<br>
&ensp;00:11:37.661693 IP (tos 0x0, ttl 64, id 36046, offset 0, flags [DF], proto ICMP (1), length 84)<br>
&emsp;&emsp;&emsp;192.168.60.0 > 192.168.60.1: ICMP echo request, id 3318, seq 3, length 64<br>
&ensp;00:11:37.661725 IP (tos 0x0, ttl 64, id 8600, offset 0, flags [none], proto ICMP (1), length 84)<br>
&emsp;&emsp;&emsp;192.168.60.1 > 192.168.60.0: ICMP echo reply, id 3318, seq 3, length 64<br>
&ensp;00:11:41.053694 ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 192.168.60.0 tell 192.168.60.1, length 28<br>
&ensp;00:11:41.053699 ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 192.168.60.1 tell 192.168.60.0, length 28<br>
&ensp;00:11:41.053707 ARP, Ethernet (len 6), IPv4 (len 4), Reply 192.168.60.0 is-at 52:0c:87:cf:05:e9, length 28<br>
&ensp;00:11:41.053723 ARP, Ethernet (len 6), IPv4 (len 4), Reply 192.168.60.1 is-at c2:ec:73:be:13:db, length 28<br>
<br>
&ensp;PING 192.168.60.1 (192.168.60.1) 56(84) bytes of data.<br>
&ensp;64 bytes from 192.168.60.1: icmp_seq=1 ttl=64 time=0.090 ms<br>
&ensp;64 bytes from 192.168.60.1: icmp_seq=2 ttl=64 time=0.066 ms<br>
&ensp;64 bytes from 192.168.60.1: icmp_seq=3 ttl=64 time=0.066 ms<br><br>

&ensp;--- 192.168.60.1 ping statistics ---<br>
&ensp;3 packets transmitted, 3 received, 0% packet loss, time 2059ms<br>
&ensp;rtt min/avg/max/mdev = 0.066/0.074/0.090/0.011 ms
</span>

>Run the following commands to clean-up:

\# `ip link set vbr0 down`\
\# `ip link delete vbr0`\
\# `for i in {0..3}; do ip netns delete ns${i}; done`