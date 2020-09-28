
######################### START HERE #######################

For this case, as a workaround, a TUNNEL can be performed when we make the SSH Connection from CLIENT A to SERVER A, which has the appropriate access to SERVER B on port 8000.

In practice, we will use SERVER A as a JUMP to access port 8000 running on SERVER B.

Below are the commands:

First open an SSH session between CLIENT A and SERVER A, on SERVER A, edit with the command: vi > /etc/ssh/sshd_config, navigate to the AllowTcpForwarding line, leaving it as follows: AllowTcpForwarding yes, then restart the sshd service > service sshd restart.

From CLIENT A, we must run the ssh command with the following parameters:

ssh -g -L 7777:SERVERB:8000 username@SERVERA

NOTE: I'm choosing the port 7777, because it has no service on any of the clients and servers involved.

FYI: The -g parameter on the coomand must be used to allow remote hosts such as CLIENT A to connect to the locally forwarded port.
FYI_2: The -L parameter in the SSH command, also known as local port forwarding (OpenSSH), will allow connections from the chosen port (7777) to be forwarded to port 8000 which is accessible on SERVER B, allowing the connection to the appropriate port .

If this SSH connection is successful, SERVER A will display its prompt asking for the password.

In order for CLIENT A to access this port (8000) on SERVER B, just open his browser and type http://localhost:7777.

######################### END HERE #######################


######################### START HERE #######################

Another way to solve the case would be to configure another network interface present on SERVER A and B, and then create a static route on CLIENT A, directing to the network configured on these interfaces, for example:

Suppose :

CLIENT A:
--
eth0

192.168.1.10/28

GW: 192.168.1.1

SERVER A:
--
eth0
192.168.1.11/28

GW: 192.168.1.1

(NETWORK TO COMMUNICATE WITH CLIENT A)


eth1
10.0.0.2/28 (NETWORK TO COMMUNICATE WITH SERVER B)

SERVER B: 
--
eth0

10.0.0.3/28

-----

In CLIENT A, a static route must be added with the following command:

# route add -net 10.0.0.0 netmask 255.255.255.240 gw 192.168.1.1

In SERVER A, the packet forwarding parameter must be enabled, so that SERVER A can redirect requests coming on port 8080 to SERVER B:

# vi /etc/sysctl.conf> line (net.ipv4.ip_forward = 1)

Still on server A, add the following rule to iptables:

# iptables -I INPUT -p tcp 10.0.0.3 --dport 8000 -j ACCEPT.

In this way, from CLIENT A, it will be possible to access port 8000 using the address > 192.168.1.11:8000, and also for SERVER B at address > 10.0.0.3:8000

-----

######################### END HERE #######################
