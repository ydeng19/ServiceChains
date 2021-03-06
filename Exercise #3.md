# Exercise 3 - Malicious IoT Detection and Blocking

# Overview

A number of malicous IoT (Internet of Things) devices are running on your network! It's your job to analyze the traffic to determine how it is communicating and block the traffic.

Each IoT device is communicating with a "Command and Control" server at least every 60 seconds using either TDP or UDP. Using TCPDump and/or Snort, determine what remote IP(s) the device is communicating and block that IP. Utilize a second NetMon to verify that the traffic is being blocked.

No login credentials are provided for the IoT box. You should consider it a black box where you can only examine traffic in/out of the virtual machine.

# Goals

  * Detect outbound message by the rogue IoT device
  * Block and log the malicious messages
  * Utilize service chains to manipulate the packet flows

# Tools

  * NetMon image is equipped with Snort and TCPDump
  * TCPDump usage information: http://www.tcpdump.org/tcpdump_man.html
  * Snort usage information: http://www.manpagez.com/man/8/snort/

# Prereq
Install the `whois` client
```
# yum install whois
```

Networking Setup:
  * Leverage the network security groups to allow SSH and HTTP to the project from your laptop external network (done in Lab Setup)
  * Leverage the external, internal, security management & service networking (done in Lab Setup)
  
  * Use NetMon hairpin to enable initial pass-through of rogue traffic
  ```
  brctl hairpin br0 eth1 on
  ```
  * Clean up previous exercise `l2insertion` in `midonet-cli`
  ```
  # midonet-cli
  midonet-cli> list l2insertion
  midonet-cli> l2insertion l2insertionX delete
  ```

# Image login info

  * admin/openstack for the NetMon images
  * no login available for the IoT boxes - consider it a black box!

# Lab Setup

## Instances

Startup the following images or reuse the NetMon from previous exercises.

| Instance Name | Image         | Flavor   | Network           | Floating IP |
| ------------- |:-------------:| --------:|------------------:|------------:|
| IoT-1         | IoT           | m1.small | internal          | none        |
| IoT-2         | IoT           | m1.small | internal          | none        |
| NetMon        | NetMon        | m1.small | mgmt,service	     | assign	eth0 |


## Service Chaining

Use L2 service-insertion or port-mirroring to monitor rogue traffic on the network on the NetMon instance. Protect other networks by inserting a NetMon for traffic routing out of the network via the gateway.

Use `neutron-cli` or Horizon to determine the network gateway port UUID and MAC address and the NetMon port UUID.
```
# neutron port-list
# neutron port-show <rogue-network-gateway_UUID>
```

Use `midonet-cli` to create Service Chaining of routed traffic by protecting the network gateway.
  ```
  # midonet-cli
  midonet-cli> list l2insertion
  midonet-cli> l2insertion add port <rogue-network-gw_UUID> srv-port <NetMon_UUID> fail-open true mac <rogue-network-gw_MAC> 
 ```
 
 ## Network Monitoring
 Monitor traffic on the NetMon to determine the source and destination IP of the rogue traffic. 
 
 ```
# tcpdump -i eth1 not port 22
# whois <rogue-traffic-dst_IP>
```

Administratively, there are several options for dealing with rogue traffic: administratively shutting down the rogue VM instance, applying policy in MidoNet via Security Groups or MidoNet rules & chains, or using the service chaining technique to use 3rd-party software to apply policy. Treat traffic to block traffic via Security Groups or snort.

Optionally, if using a 3rd party software like snort on the NetMon instance, you can leverage service chaining to observe "post-policy" traffic to determine the polcies are implemented. As in the previous exercise, you can use `midonet-cli` to add to the existing service chain and insert another NetMon image.

Finally, log into the NetMon instance and observe that the policy is enforced.

From here feel free to use the tools in the workshop to experiment as you wish!
