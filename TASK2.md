# Redirect traffic from port 81 in one subnetwork to port 80 in another subnetwork (Using Centos/RHEL 6)
## Introduction
This guide will show you how to redirect traffic from port 81 in one subnetwork to port 80 in another subnetwork using NAT prerouting and postrouting on Centos/RHEL 6.

## Prerequisites 
- Two virtual machines with CentOS/RHEL 6
- Installed `iptables`, `curl`, and `httpd` packages

## A little bit of theory
Port forwarding allows you to forward incoming packets to another destination, such as another port or IP address. This can be useful for redirecting traffic to a specific server or device on your network.

To set up port forwarding, you need to configure your firewall to allow incoming connections on the port that you want to forward, and then create a rule to tell the firewall where to forward the traffic.

__Here is a simplified explanation of how port forwarding works:__

1. A client sends a packet to port 81 on server A.
2. Server A's firewall intercepts the packet and checks to see if there is a port forwarding rule for port 81.
3. If there is a port forwarding rule for port 81, the firewall modifies the packet's destination port to port 80 and forwards the packet to server B.
4. Server B receives the packet and processes it as if it had been sent directly to port 80.
5. Server B sends a response packet back to the client.
6. The firewall on server A intercepts the response packet and modifies the packet's source port to port 81 before forwarding the packet to the client.
7. The client receives the response packet and thinks that it came from server A on port 81.

To set up port forwarding, you need to configure your firewall and create a rule to tell it where to forward the traffic.

### Remark about iptables
In CentOS/RHEL 7 iptables have been replaced with firewalld. To use iptables instead of firewalld on CentOS/RHEL 7, you need to disable firewalld and enable iptables instead. In this example I am using CentOS 6, so I do not need to do that.

## IP Forwarding
```bash
[abohat@vm ~]$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
```
If the output is equal to 0, this means that IP forwarding is disabled. To enable IP forwarding use provided command:

```bash
[abohat@vm ~]$ sudo echo "net.ipv4.ip_forward = 1"|sudo tee /etc/sysctl.d/99-ipforward.conf
net.ipv4.ip_forward = 1
```

And to activate the changes:
```bash
[abohat@vm ~]$ sudo sysctl -p /etc/sysctl.d/99-ipforward.conf
net.ipv4.ip_forward = 1
```

## Iptables
Next, you need to check that iptables is running on your system. Iptables is running as a kernel module, so it can't be seen as one of the normal processes. You need to use the next command:
```bash
[abohat@vm ~]$ lsmod|grep iptable
iptable_nat            12928  0
nf_nat                 18242  1 iptable_nat
nf_conntrack_ipv4      14078  3 nf_nat,iptable_nat
nf_conntrack           52720  3 nf_conntrack_ipv4,nf_nat,iptable_nat
iptable_filter         12536  0
ip_tables              22042  2 iptable_filter,iptable_nat
x_tables               19118  3 ip_tables,iptable_filter,iptable_nat
```

If there is no output, it means that iptables is disabled.
To start iptables use provided command:
```bash
[abohat@vm ~]$ sudo systemctl start iptables
```

You can see that there are no rules configured and all traffic is allowed to and from the system.

INPUT, OUTPUT and FORWARD chains:

```bash
[abohat@vm ~]$ sudo iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

NAT-related chains:
```bash
[abohat@vm ~]$ sudo iptables -t nat -L -n
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
```

In case you have some rules configured, you can flush them, but __be careful about doing this in the production environment, it can be dangerous.__
```bash
[abohat@vm ~]$ sudo iptables -F
[abohat@vm ~]$ sudo iptables -t nat -F
```

## Enable Port Forwarding
We need to use PREROUTING and POSTROUTING rules for NAT iptables because they allow us to modify packets before and after they are routed. 
- __PREROUTING__ rules are applied to packets before they are routed. 
- __POSTROUTING__ rules are applied to packets after they are routed.

```bash
[abohat@vm ~]$ sudo iptables -t nat -A PREROUTING -p tcp --dport <forwarded port> -j DNAT --to-destination <IP of web server>:<port on web server>
[abohat@vm ~]$ sudo iptables -t nat -A POSTROUTING -p tcp -d <IP of web server> --dport <port on a web server> -j SNAT --to-source <forwarded IP>
```

Now if we check tables of rules we should see new rules:
```bash
[abohat@vm ~]$ sudo iptables -t nat -L -n
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:81 to:10.0.2.1:80

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
SNAT       tcp  --  0.0.0.0/0            10.0.2.1              tcp dpt:80 to:10.0.1.1
```

And finally to permanently save the rules, execute iptables-save:
```bash
[abohat@vm ~]$ sudo iptables-save|sudo tee /etc/sysconfig/iptables
# Generated by iptables-save v1.4.21 on Fri Nov  10 16:02:29 2023
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A PREROUTING -p tcp -m tcp --dport 81 -j DNAT --to-destination 10.0.2.1:80
-A POSTROUTING -d 10.0.2.1/24 -p tcp -m tcp --dport 80 -j SNAT --to-source 10.0.1.1
COMMIT
# Completed on Fri Nov  6 12:02:25 2014
# Generated by iptables-save v1.4.21 on Fri Nov  10 16:02:29 2023
*filter
:INPUT ACCEPT [862:76022]
:FORWARD ACCEPT [98:11497]
:OUTPUT ACCEPT [666:84508]
COMMIT
# Completed on Fri Nov  10 16:02:29 2023
```

## Conclusion
You have now successfully redirected traffic from port 81 in one subnetwork to port 80 in another subnetwork using NAT prerouting and postrouting on Centos/RHEL 6.

