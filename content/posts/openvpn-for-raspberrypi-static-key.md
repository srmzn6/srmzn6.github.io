---
title: "Openvpn for Raspberrypi Static Key"
date: 2012-08-21 07:28:16.000000000 -04:00
draft: false
tags:
    - Raspberry Pi
keywords:
    - Raspberry Pi
    - OpenVPN
---
You all must be familiar with the amazing things that your RaspberryPi can do ! One of the things that I wanted to get on RaspberryPi was OpenVPN so that I could securely browse the internet while on free WiFi networks like Starbucks. Without the fear of being snooped on.

Before beginning, please note that this is an OpenVPN static key setup and following are the advantages and disadvantages of this setup.
**Advantages:**
1. Simple setup.
2. No X509 PKI (Public Key Infrastructure) to maintain.

**Disadvantages:**
1. Limited Scalibility.
2. If the static key is compromised then all the previous sessions can be disclosed.
3. Secret Key must exist in plaintext form on each VPN peer.
4. Secret Key must be exchanged using pre-existing secure channel.


For me, I am ready to live with the disadvantages (for now).

Following are the steps to get and configure OpenVPN Server on your RaspberryPi. Assuming that you have debian, I don't expect it to be much different for other linux distributions.

**Get OpenVPN:**
`$ sudo apt-get install openvpn`

*Get bridge-utils* ?? (can be skipped)
`$ sudo apt-get install bridge-utils`

**Generate the Static Key**
`$ openvpn --genkey --secret static.key`

Please, make sure that static.key is not compromised.
	
**Create Server Configuration File:**
create a file called server.conf with the following contents:

```
dev tun
ifconfig 10.8.0.1 10.8.0.2
secret static.key
```

**Start the OpenVPN server:**
start OpenVPN server with the following command:
`$ sudo openvpn --config (path to server.conf)/server.conf --log (path to save the log)/server.log`

**Configure the client:**
1. Copy the static.key to client and server over a secure channel.
2. create a file on the client machine client.conf with following contents:


```
remote myremote.mydomain
dev tun
ifconfig 10.8.0.2 10.8.0.1
secret static.key
```

On the client you can use the OpenVPN client (for Mac/Win/Linux) (download from [OpenVPN](http://openvpn.net/) and all it needs is the client.conf file and the static.key.

### VPN Speed Test Results
**Without VPN**:   Download: 2.90 Mb/s      Upload: 2.69 Mb/s      Latency: 25 ms 

**With VPN**:        Download: 2.71 Mb/s      Upload: 2.03 Mb/s      Latency: 27 ms

