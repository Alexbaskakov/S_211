# MikroTik RouterOS
## Table of contents:
#### Command Line Interface (CLI) Overview
#### Reset Configuration
#### Example Router Configuration
MikroTik is a manufacturer of networking equipment, including switches, routers, and wireless access points. MikroTik products run the MikroTik RouterOS operating system, which is based on the Linux kernel.
### Command Line Interface (CLI) Overview
RouterOS can be controlled from a command line interface (CLI). There are a variety of commands, each with their own syntax and possible options. The CLI can be accessed in multiple ways: via direct serial port connection, over the network via telnet or SSH, via a console screen within the GUI Winbox utility, or via your web browser. The console is also used for writing scripts. Since there are a lot of available commands, they are split into groups organized in a hierarchy. Fortunately, you do not have to remember every possible command, or even the complete list of commands. RouterOS has several features to help you in entering commands.

- **Abbreviated Commands**: For every command, you only need to type as many letters as are needed to make that command unique. For example, consider the command ``ping 10.0.0.1 count 3 size 100``. That same command could be entered as `pi 10.1 c 3 si 100`, which is much shorter! In addition, commands can be auto-completed through use of the `[TAB]` key. For example, `interface mt[TAB]` is completed as `interface monitor-traffic`, which is the full command.
- **HotLock Mode**: When HotLock mode is enabled, commands will be auto completed. To enter/exit HotLock mode press `[CTRL]+[V]`. The Double `>>` is an indication that HotLock mode is enabled. For example if you type `pi`, it will be auto completed to `ping`.

- **Online Help**: Whenever you are stuck, press the ? key, and the CLI will display the syntax and command structure for the command you have entered. For example, `ping ?` will show the syntax for the ping command.

- **Safe Mode**: When you first connect to the router and log in (default username: admin, no password) the command line will give you a prompt like this: [admin@MikroTik]>. It is sometimes possible to change the router configuration in a way that will make the router inaccessible over the network, and only reachable from the local console. If this kind of operator error occurs, there may be no way to undo the last change since the connection to the router is already cut. Safe mode can be used to minimize such risk. Safe mode is entered by pressing ``[CTRL]+[X]``. To save changes and quit safe mode, you should press ``[CTRL]+[X]`` again. To exit without saving changes, hit ``[CTRL]+[D]``.

- **To avoid configuration problems**, **it is recommended you always enter the safe mode at the beginning of each lab via ``[CTRL]+[X]`` and never exit this mode until the end of the lab, where you exit with ``[CTRL]+[D]``. The message Safe Mode taken** is displayed and the command prompt changes to reflect that the session is now in safe mode ``(<SAFE>)``. All configuration changes that are made while the router is in safe mode are automatically undone if safe mode session terminates abnormally, i.e., by ``[CTRL]+[D]``. Note that quit is considered as a normal termination, and thus will save the changes into the configuration.

## Reset Configuration
You want to start every lab with the default configuration and ensure that nothing has been saved in the router configuration from a previous lab:
```
system reset-configuration
```
``Tip``: Use the tab key to auto-complete this command!

The router will ask you to confirm that you want to reset the configuration. Type “y” as your response. The command sets the router back to its default configuration, including the default login name and password ('admin' and no password). IP addresses and other configuration is erased, and interfaces will be disabled. After the reset command, the router will reboot, and you will need to reconnect.

If you're curious about the default configuration, you can print it via:
```
system default-configuration print
```
## Example Router Configuration
This example shows how to configure a simple router with two interfaces connecting two subnets at the command line: Generic Two Subnet Router Network
![](https://cyberlab.pacific.edu/images/a/8/c/5/5/a8c5572f28de56807549ac2e4f5eff78c9fa34ba-two-subnet-config.png)
If you are not in safe mode, go to this mode by `[CTRL]+[X]`. The command prompt will change from `[admin@MikroTik] > to [admin@MikroTik] > <SAFE>`

Check the list of interfaces on your router:
```
<SAFE> interface print
Flags: D - dynamic, X - disabled, R - running, S - slave
 #     NAME                                TYPE       ACTUAL-MTU L2MTU  MAX-L2MTU MAC-ADDRESS
 0  R  ether1                              ether            1500                  0C:46:BE:AE:E1:00
 1  R  ether2                              ether            1500                  0C:46:BE:AE:E1:01
 2  R  ether3                              ether            1500                  0C:46:BE:AE:E1:02
 3  R  ether4                              ether            1500                  0C:46:BE:AE:E1:03
 ```
Check the list of IP addresses assigned to the interfaces:
```
<SAFE> ip address print
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 ```
Assign IP address 192.168.0.1 with subnet mask 255.255.255.0 to interface ether1:
```
<SAFE> ip address add address=192.168.0.1 netmask=255.255.255.0 interface=ether1
```
Assign IP address 192.168.1.1 with subnet mask 255.255.255.0 to interface ether2:
```
<SAFE> ip address add address=192.168.1.1 netmask=255.255.255.0 interface=ether4
```
Verify that the IP addresses have been assigned to the ether1 and ether2 interfaces:
```
<SAFE> ip address print
[admin@MikroTik] <SAFE> ip address print
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0   192.168.0.1/24     192.168.0.0     ether1
 1   192.168.1.1/24     192.168.1.0     ether2
```
_______________
`Tip:` if you make a mistake entering an IP address, you can remove it via the following process:

1) First, enter ``ip address print`` and note the line number (#) that the incorrect entry appears on
2) Next, enter `ip address remove numbers=X`, where `X` is the number of the entry you want to remove.
3) Confirm that the incorrect entry was removed by repeating `ip address print` again.
____________________
For this simple network, we don't need to define routes. Both subnets are directly attached to the router, and the routing table entries will be added automatically. But if we did want to add those static manually, the commands would look like this:

Route the traffic destined to the network 192.168.0.0/24 through interface ether1 and the traffic destined to the network 192.168.1.0/24 through interface ether2
```
<SAFE> ip route add dst-address=192.168.0.0/24 gateway=ether1
<SAFE> ip route add dst-address=192.168.1.0/24 gateway=ether2
```
Check the assigned static routes to verify network configuration:
```
<SAFE> ip route print
Flags: X - disabled, A - active, D - dynamic, C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme,
B - blackhole, U - unreachable, P - prohibit
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 ADC  192.168.0.0/24     192.168.0.1     ether1                    0
 1 ADC  192.168.1.0/24     192.168.1.1     ether2                    0
 ```
Finally, disable the DHCP Client on the router. Unless you ran Wireshark on a link, you wouldn't notice that a new MikroTik router defaults to running a DHCP client on Ether1, trying to auto-configure that network port so a new administrator can access the router over the network. It's not helpful for us in a simulated network.
```
ip dhcp-client print    # Observe the running DHCP client and its configuration number
ip dhcp-client remove numbers=0    #Remove it
```
Press `Control+[D]` to exit safe mode and leave the router configuration unchanged.

# Static routing

In GNS3, create a network topology that matches this one:
![Network Diagram (Note: Subnet labels and dashed borders are for informational use only)](https://cyberlab.pacific.edu/images/4/c/5/b/c/4c5bcfca4be82c83d1487802273c0ac5a0b5b560-network-03.png)

### This network should meet the following specifications:
```
There are two routers with hostnames left and right as viewed on the network diagram.
```
```
Subnet 1 - 172.16.10.0/24
- Contains one Ethernet Switch
- Contains VPC1 with IP address 172.16.10.1
- Contains the "Ether1" port of the "left" Router with IP address 172.16.10.254
```
```
Subnet 2 - 172.16.20.0/24
- Contains one Ethernet Switch
- Contains VPC2 with IP address 172.16.20.1
- Contains the "Ether2" port of the "left" Router with IP address 172.16.20.254
```
```
Subnet 3 - 192.168.0.0/24
- Contains the "Ether3" port of the "left" Router with IP address 192.168.0.254
- Contains the "Ether1" port of the "right" Router with IP address 192.168.0.253
```
```
Subnet 4 - 10.11.12.0/24
- Contains one Ethernet Switch
- Contains VPC3 with IP address 10.11.12.1
- Contains the "Ether2" port of the "right" Router with IP address 10.11.12.254
```
```
Subnet 5 - 10.11.13.0/24
- Contains one Ethernet Switch
- Contains VPC4 with IP address 10.11.13.1
- Contains the "Ether3" port of the "right" Router with IP address 10.11.13.254
```
Unlike in previous labs, here you are responsible for determining the correct commands to use when configuring your network devices. Refer to the previous labs if you need a refresher on how to configure IP addresses and hostnames for the routers and VPCs.

**Tips:**
- The process goes more smoothly if you configure the routers first, and then the PCs in each subnet.
- The specific port on a switch does not matter (until we get to a point in the semester where we are configuring switches)
- The specific port on a router does matter. The router configuration in software needs to be consistent with the way the cables are wired in hardware.
## Configuration Steps:

1) **Configure the hostnames of the "left" and "right" routers** to prevent confusion. Use the ``system identity set name=XXX`` command. Notice that the command prompt changes to reflect this. You should see `[admin@left] >` instead of `[admin@MikroTik] >`
2) **Configure IP addresses on all router interfaces that are connected to subnets**. Use the `ip address add address=a.b.c.d/n interface=XXX` command.
3) **Configure the IP address on each VPC**. Use the `ip a.b.c.d/n w.x.y.z` command. When determining the correct "ip" command to use, ask yourself:
-   1) What is the IP address I want the machine to have?
-   2) What is the subnet that IP address is in?
-   3) What is the IP address of the default gateway? The default gateway is the router that the VPC should send packets to when trying to communicate outside the subnet. Because routers have multiple IP addresses, you should choose the IP address of the interface that is within the subnet in question.
-   4) Save the configuration on the VPCs via the `save` command and exit safe mode on the router.
When finished, your left router should be configured like this:
```
[admin@left] > ip address print
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0   192.168.0.254/24   192.168.0.0     ether3
 1   172.16.20.254/24   172.16.20.0     ether2
 2   172.16.10.254/24   172.16.10.0     ether1
 ```
And your right router should be configured like this:
```
[admin@right] > ip address print
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0   192.168.0.253/24   192.168.0.0     ether1
 1   10.11.12.254/24    10.11.12.0      ether2
 2   10.11.13.254/24    10.11.13.0      ether3
```
And PC1 (as an example) should be configured like this:
```
 PC1> show ip

NAME        : PC1[1]
IP/MASK     : 172.16.10.1/24
GATEWAY     : 172.16.10.254
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 20018
RHOST:PORT  : 127.0.0.1:20019
MTU:        : 1500
```
## Test Network
How do we know if the network is working? Let's test it, but in a very **systematic** way, to ensure all the pieces are functional.

First, check to see that each individual host has its own IP address and subnet configured as desired, and has connectivity. Do this by pinging a neighbor in the same subnet.

1) From the left router, ping the right router (192.168.0.253). They are on the same subnet. This should work.
2) From the left router, ping PC1 (172.16.10.1) through the switch. They are on the same subnet. This should work.
3) From the left router, ping PC2 (172.16.20.1) through the switch. They are on the same subnet. This should work.
4) From the right router, ping PC3 (10.11.12.1) through the switch. They are on the same subnet. This should work.
5) From the right router, ping PC4 (10.11.13.1) through the switch. They are on the same subnet. This should work.

Next, let's check if we can send messages **between** subnets.

1) From PC1, ping PC2 (172.16.20.1) through the switch, router, and switch. This should work.
2) From PC1, ping the ether1 port of the right router (192.168.0.253). This should NOT work.

```Wait, why does this not work?? Everything is wired up! Stupid computers... Stupid Computers```

Let's think about what happens when the ping request message is sent:

1) PC1 wants to send a ping request message to the "right" router - 192.168.0.253.
2) PC1 determines that this destination is outside the local subnet
3) PC1 forwards the message to the "left" router
4) The "left" router determines that the destination is on a local subnet, subnet 3, and is directly attached
5) The "left" router forwards the message to the destination. Success!
![Network Diagram](https://cyberlab.pacific.edu/images/4/6/1/9/a/4619a0ac23e888d2dd23f604552c3591b3411b44-network-03-ping-request.png)

While the ping request is successfully received, the ping reply immediately runs into difficulties.

1) The "right" router wants to send a ping reply message to PC1 - 172.16.10.1.
2) The "right" router determines that the destination is not on any directly attached subnets
3) The "right" router never transmits the ping reply because it doesn't know where to send it. **Failure!**
![Network Diagram](https://cyberlab.pacific.edu/images/b/8/5/3/b/b853bbecbafed8da283e6fd114b1cc8df364d7ce-network-03-ping-reply.png)
```
What is needed is a way to tell each router about subnets that are not directly connected, but instead are further away. This is accomplished in the routing table.
```
Routing tables tell routers where to send packets whose destination network is not directly attached to the router. Routing table entires can be static or dynamic. Today, we look at static routes, which are entered manually. In a future lab we'll look at protocols that a router can use to create a routing table dynamically.

## Create Static Routes
On each router, you need to create static entries in the routing table specifying how to reach any subnets that are not directly connected to the router. (It learned about directly attached subnets when you configured the interfaces).

The syntax for this is: `ip route add dst-address=a.b.c.d/n gateway=w.x.y.z`. That specifies the subnet to be reached `(a.b.c.d/n)` and the IP of the next-hop router `(w.x.y.z)` which will move the packet closer to its destination. Note that this is the "next-hop" router - there may be others routers between here and the destination, but only the next hop is being entered in this particular routing table.

For the left router:
```
# Examine existing routing table with the entries created by the interfaces
ip route print

# Add new entries specifying how to reach distant subnets by way of the "right" router
# To reach subnet 4, the next hop is the "right" router:
ip route add dst-address=10.11.12.0/24 gateway=192.168.0.253
# To reach subnet 5, the next hop is the "right" router:
ip route add dst-address=10.11.13.0/24 gateway=192.168.0.253
```
When finished, the routing table for the left router should look like this:
```
[admin@left] > ip route print
Flags: X - disabled, A - active, D - dynamic,
C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme,
B - blackhole, U - unreachable, P - prohibit
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 A S  10.11.12.0/24                      192.168.0.253             1
 1 A S  10.11.13.0/24                      192.168.0.253             1
 2 ADC  172.16.10.0/24     172.16.10.254   ether1                    0
 3 ADC  172.16.20.0/24     172.16.20.254   ether2                    0
 4 ADC  192.168.0.0/24     192.168.0.254   ether3                    0
 ```
`After adding those two static routing table entries for the left router, switch to the right router and add two similar rules for its routing table.`

## Test Network (Again)
1) From PC1, ping the ether1 port of the right router (192.168.0.253). This should work.
2) From PC1, ping PC3 (10.11.12.1). This should work.
3) From PC1, ping PC4 (10.11.13.1). This should work.

### Lab Submission
Submit the following items to the assignment:

Screenshot(s) showing:

- Your GNS3 topology
- Successful pings from PC1 to PC3.
- Successful pings from PC1 to PC4
- List the two "ip route add" rules you manually entered into the right router to give it full knowledge of all the subnets in our network.
- List the IP address of every interface through which a ping packet passes on its way from the ping application in PC2 to the ping application in PC3.

- Explain in your own words: What is a subnet?

- Explain in your own words: How does a router determine whether a destination IP address is inside a particular subnet?

- When finished, press the Stop button and exit GNS3. The GNS3 VM (in VMware) should halt and exit automatically.

## Troubleshooting
Having trouble getting your network pings to work when they should? Before asking for assistance, verify that the IP addresses of all the PCs and routers in your network are correctly configured. Example output from a working lab network is shown below.

IP Address Configuration
![](https://cyberlab.pacific.edu/images/c/9/6/6/6/c966661e20e26c9d5100334e14d14dbb4c73ce16-ip-addr-troubleshooting.png)
