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
