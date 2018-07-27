# dockerce-windows10

## Host Setup
* Machine: Standard D8s v3 (8 vcpus, 32 GB memory)
* Host OS: Windows 10 1803, 17134.191
* Windows Update: KB4340917
* Docker CE Stable Version: 18.06.0-ce-win72 (19098)

## Windows Container tests

``` 
docker swarm init
Error response from daemon: could not find the system's IP address - specify one with --advertise-addr
```

```
docker pull microsoft/nanoserver
OK
```

```
docker run -d microsoft/nanoserver ping 127.0.0.1 /t
OK
```

## Network setup

```
PS C:\Users\marten> ipconfig

Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : xxx.internal.cloudapp.net
   Link-local IPv6 Address . . . . . : fe80::f1ea:b879:51b:837f%8
   IPv4 Address. . . . . . . . . . . : 10.0.1.4
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.0.1.1

Ethernet adapter vEthernet (Default Switch):

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::9e1:a82e:b4e3:1996%5
   IPv4 Address. . . . . . . . . . . : 172.31.38.65
   Subnet Mask . . . . . . . . . . . : 255.255.255.240
   Default Gateway . . . . . . . . . :

Ethernet adapter vEthernet (nat):

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::d056:9466:e8e8:5005%18
   IPv4 Address. . . . . . . . . . . : 172.18.16.1
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . :
```
