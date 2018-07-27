# dockerce-windows10


## Host Setup
* Machine: Standard D8s v3 (8 vcpus, 32 GB memory)
* Host OS: Windows 10 1803, 17134.191
* Windows Update: KB4340917
* Docker CE Stable Version: 18.06.0-ce-win72 (19098)

## Installation

* Run windows update to install latest windows updates KB4340917
* Disable cortana: https://gallery.technet.microsoft.com/scriptcenter/How-to-disable-Cortana-on-b44924a4
* Install Docker CE. restarts computer after installation.
* Ask to enable windows containers, does so, ask to enable hyper-v and container support.
* Computer reboots and docker service starts successfully.

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

```
docker swarm init --advertise-addr 10.0.1.4

- RDP disconnect
- Relogin, ok.

PS C:\Users\marten> docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 1
Server Version: 18.06.0-ce
Storage Driver: windowsfilter
 Windows:
Logging Driver: json-file
Plugins:
 Volume: local
 Network: ics l2bridge l2tunnel nat null overlay transparent
 Log: awslogs etwlogs fluentd gelf json-file logentries splunk syslog
Swarm: active
 NodeID: v6z6qt51q4zdf399z28njytyj
 Is Manager: true
 ClusterID: 636ln7x991fgud2msg4vjaoui
 Managers: 1
 Nodes: 1
 Orchestration:
  Task History Retention Limit: 5
 Raft:
  Snapshot Interval: 10000
  Number of Old Snapshots to Retain: 0
  Heartbeat Tick: 1
  Election Tick: 10
 Dispatcher:
  Heartbeat Period: 5 seconds
 CA Configuration:
  Expiry Duration: 3 months
  Force Rotate: 0
 Autolock Managers: false
 Root Rotation In Progress: false
 Node Address: 10.0.1.4
 Manager Addresses:
  10.0.1.4:2377
Default Isolation: hyperv
Kernel Version: 10.0 17134 (17134.1.amd64fre.rs4_release.180410-1804)
Operating System: Windows 10 Pro N Version 1803 (OS Build 17134.191)
OSType: windows
Architecture: x86_64
CPUs: 8
Total Memory: 32GiB
Name: win10
ID: 7ZZH:G2CK:WBB6:B37G:OBTX:DJGB:HIIF:ZDKM:NZ57:7JWG:NYBR:WGNN
Docker Root Dir: C:\ProgramData\Docker
Debug Mode (client): false
Debug Mode (server): true
 File Descriptors: -1
 Goroutines: 140
 System Time: 2018-07-27T11:31:02.597615Z
 EventsListeners: 1
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
```

```
PS C:\Users\marten> docker service create --name=test microsoft/nanoserver ping 127.0.0.1 /t
bf04o3dpg9xpwqdpwq81spxa1
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged

PS C:\Users\marten> docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                         PORTS
bf04o3dpg9xp        test                replicated          1/1                 microsoft/nanoserver:latest
PS C:\Users\marten> docker service ps test
ID                  NAME                IMAGE                         NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
24q827gwpki4        test.1              microsoft/nanoserver:latest   win10               Running             Running about a minute ago

```

## DNS Resolution tests

### Docker Compose

```
version: '3.2'

services:
  frontend:
    image: microsoft/nanoserver
    command: ping backend /t 
  backend:
    image: microsoft/nanoserver
    command: ping frontend /t
```

```
PS C:\Users\marten> docker-compose up
WARNING: The Docker Engine you're using is running in swarm mode.

Compose does not use swarm mode to deploy services to multiple nodes in a swarm. All containers will be scheduled on the current node.

To deploy your application across the swarm, use `docker stack deploy`.

Creating network "marten_default" with the default driver
Creating marten_backend_1  ... done
Creating marten_frontend_1 ... done
Attaching to marten_frontend_1, marten_backend_1
frontend_1  |
frontend_1  | Pinging backend [172.20.186.66] with 32 bytes of data:
frontend_1  | Reply from 172.20.186.66: bytes=32 time=2ms TTL=128
backend_1   |
backend_1   | Pinging frontend [172.20.187.254] with 32 bytes of data:
backend_1   | Reply from 172.20.187.254: bytes=32 time<1ms TTL=128
frontend_1  | Reply from 172.20.186.66: bytes=32 time=1ms TTL=128
backend_1   | Reply from 172.20.187.254: bytes=32 time=2ms TTL=128
frontend_1  | Reply from 172.20.186.66: bytes=32 time=1ms TTL=128
backend_1   | Reply from 172.20.187.254: bytes=32 time=3ms TTL=128
backend_1   | Reply from 172.20.187.254: bytes=32 time<1ms TTL=128
frontend_1  | Reply from 172.20.186.66: bytes=32 time<1ms TTL=128
frontend_1  | Reply from 172.20.186.66: bytes=32 time=1ms TTL=128
backend_1   | Reply from 172.20.187.254: bytes=32 time=1ms TTL=128
frontend_1  | Reply from 172.20.186.66: bytes=32 time=11ms TTL=128
backend_1   | Reply from 172.20.187.254: bytes=32 time<1ms TTL=128
frontend_1  | Reply from 172.20.186.66: bytes=32 time=21ms TTL=128
backend_1   | Reply from 172.20.187.254: bytes=32 time=1ms TTL=128
Gracefully stopping... (press Ctrl+C again to force)
```


### Docker Swarm

```
version: '3.2'

services:
  frontend:
    image: microsoft/nanoserver
    command: ping backend /t 
  backend:
    image: microsoft/nanoserver
    command: ping frontend /t
```

```
PS C:\Users\marten> docker stack deploy -c .\docker-compose.yml test
Creating network test_default
Creating service test_backend
Creating service test_frontend
PS C:\Users\marten>
```

```
PS C:\Users\marten> docker service logs -f test_backend
test_backend.1.wzdrqabitj6e@win10    | Ping request could not find host frontend. Please check the name and try again.
test_backend.1.v3t2k6rxy7t9@win10    | Ping request could not find host frontend. Please check the name and try again.
test_backend.1.uyp8hyv21ln2@win10    | Ping request could not find host frontend. Please check the name and try again.
test_backend.1.uvakv6zbwfo4@win10    | Ping request could not find host frontend. Please check the name and try again.
PS C:\Users\marten> docker service logs -f test_frontend
test_frontend.1.oamkvlq8pwrc@win10    | Ping request could not find host backend. Please check the name and try again.
test_frontend.1.xb5ny2amdcrn@win10    | Ping request could not find host backend. Please check the name and try again.
test_frontend.1.w348m7fpb8iz@win10    | Ping request could not find host backend. Please check the name and try again.
test_frontend.1.yxxhbrkdppfp@win10    | Ping request could not find host backend. Please check the name and try again.
test_frontend.1.vlfg64xorttp@win10    | Ping request could not find host backend. Please check the name and try again.

```

## Setup test swarm DNS resolution

```
C:\Users\marten>type docker-compose.yml
version: '3.2'

services:
  frontend:
    image: microsoft/nanoserver
    command: ping 127.0.0.1 /t
  backend:
    image: microsoft/nanoserver
    command: ping 127.0.0.1 /t
```

```
PS C:\Users\marten> docker ps
CONTAINER ID        IMAGE                         COMMAND               CREATED             STATUS              PORTS               NAMES
ed2e69fedac1        microsoft/nanoserver:latest   "ping 127.0.0.1 /t"   3 minutes ago       Up 3 minutes                            test2_backend.1.703mxsc2b62vgycs0mssr8msh
10bdabd7a286        microsoft/nanoserver:latest   "ping 127.0.0.1 /t"   3 minutes ago       Up 3 minutes                            test2_frontend.1.jnffccepv2nq49ew435lmobpo
```

ed2e69fedac1:
```
Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : ctw2ceifly2ehh5cn5mbuouule.ax.internal.cloudapp.net
   Link-local IPv6 Address . . . . . : fe80::dda:1cee:5e1e:8b81%4
   IPv4 Address. . . . . . . . . . . : 10.0.3.6
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.0.3.1
```

10bdabd7a286
```
Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : ctw2ceifly2ehh5cn5mbuouule.ax.internal.cloudapp.net
   Link-local IPv6 Address . . . . . : fe80::6cc1:8770:cd7e:d3bd%4
   IPv4 Address. . . . . . . . . . . : 10.0.3.4
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.0.3.1
```

```
PS C:\Users\marten> docker inspect test2_default
[
    {
        "Name": "test2_default",
        "Id": "iukewht4k0kotgdj31op5i9sk",
        "Created": "2018-07-27T12:10:12.5367641Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.0.3.0/24",
                    "Gateway": "10.0.3.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "10bdabd7a286336364cf0b58cd9660dd70958d2ecdc14c2887c95f8a4d73402d": {
                "Name": "test2_frontend.1.jnffccepv2nq49ew435lmobpo",
                "EndpointID": "70009230fb6d74039a50a12d760c08e03c31f4cc099f704559951788f379a66d",
                "MacAddress": "00:15:5d:b2:e8:0f",
                "IPv4Address": "10.0.3.4/24",
                "IPv6Address": ""
            },
            "ed2e69fedac128e4e25a936901d0f5dc52e86723a090da45cce1afb884dd8c8d": {
                "Name": "test2_backend.1.703mxsc2b62vgycs0mssr8msh",
                "EndpointID": "1cedab666ce750e926019f3793efc81281cef9d3f5cf3b26d380b467999179a0",
                "MacAddress": "00:15:5d:b2:e6:97",
                "IPv4Address": "10.0.3.6/24",
                "IPv6Address": ""
            },
            "lb-test2_default": {
                "Name": "test2_default-endpoint",
                "EndpointID": "7ae7921b515490ebe57b6792258690537240cb7cd50bea99f9422fcf6553f06f",
                "MacAddress": "00:15:5d:b2:e8:0e",
                "IPv4Address": "10.0.3.2/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4100",
            "com.docker.network.windowsshim.hnsid": "986608f7-0e25-4851-9ac3-47014bc7dac0"
        },
        "Labels": {
            "com.docker.stack.namespace": "test2"
        },
        "Peers": [
            {
                "Name": "65efd0fcb3a1",
                "IP": "10.0.1.4"
            }
        ]
    }
]
```

```
Ping connective works.
```


## Expose Port from container

```
docker pull stefanscherer/whoami
docker run -d -p 8080:8080 --name whoami -t stefanscherer/whoami
(iwr http://$(docker inspect -f '{{ .NetworkSettings.Networks.nat.IPAddress }}' whoami):8080 -UseBasicParsing).Content

PS C:\Users\marten> (iwr http://$(docker inspect -f '{{ .NetworkSettings.Networks.nat.IPAddress }}' whoami):8080 -UseBasicParsing).Content
I'm 26afd92eca30 running on windows/amd64
```

## External port publishing from service

```
PS C:\Users\marten> docker service create --name=whoami --publish 8080:8080 stefanscherer/whoami
6q427z7ray9cakbdrvwq348m6
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```

```
PS C:\Users\marten> (iwr http://137.117.196.184:8080 -UseBasicParsing).Content
I'm de6fb837b003 running on windows/amd64
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

## Docker Info

```
PS C:\Users\marten> docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 1
Server Version: 18.06.0-ce
Storage Driver: windowsfilter
 Windows:
Logging Driver: json-file
Plugins:
 Volume: local
 Network: ics l2bridge l2tunnel nat null overlay transparent
 Log: awslogs etwlogs fluentd gelf json-file logentries splunk syslog
Swarm: inactive
Default Isolation: hyperv
Kernel Version: 10.0 17134 (17134.1.amd64fre.rs4_release.180410-1804)
Operating System: Windows 10 Pro N Version 1803 (OS Build 17134.191)
OSType: windows
Architecture: x86_64
CPUs: 8
Total Memory: 32GiB
Name: win10
ID: 7ZZH:G2CK:WBB6:B37G:OBTX:DJGB:HIIF:ZDKM:NZ57:7JWG:NYBR:WGNN
Docker Root Dir: C:\ProgramData\Docker
Debug Mode (client): false
Debug Mode (server): true
 File Descriptors: -1
 Goroutines: 26
 System Time: 2018-07-27T11:28:32.2424744Z
 EventsListeners: 1
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
```

## Compute Processes are HyperV

```
PS C:\Users\marten>  get-computeprocess | ? { $_.Owner -eq "docker" }


Id                : 6114e0099b396cc82ec6fc393dbf16114ec8710f2654872eb7cff4b30af12c62
Type              : Container
Isolation         : HyperV
IsTemplate        : False
RuntimeId         : dd5721ea-0684-49d2-9e48-c0d645d29e36
RuntimeTemplateId : 6BDDD4E3-D0E4-44DF-BE4E-A318B4A0FB82
RuntimeImagePath  : C:\ProgramData\Docker\windowsfilter\53e736d2f48446814c1c2d22409eceacd380cb4fe5960564304ee967a83e3384\UtilityVM
Owner             : docker

Id                : c442f20f2c977aef0346372e48445a1de1ac7efd71de58976768a490977be1a2
Type              : Container
Isolation         : HyperV
IsTemplate        : False
RuntimeId         : 7f5e0477-810e-482a-8315-5f60e695661d
RuntimeTemplateId : 6BDDD4E3-D0E4-44DF-BE4E-A318B4A0FB82
RuntimeImagePath  : C:\ProgramData\Docker\windowsfilter\53e736d2f48446814c1c2d22409eceacd380cb4fe5960564304ee967a83e3384\UtilityVM
Owner             : docker



PS C:\Users\marten>


```

### KBS

```
PS C:\Users\marten> Get-WmiObject -query 'select * from win32_quickfixengineering' | foreach {$_.hotfixid}
KB4230204
KB4338832
KB4343669
KB4340917
```

## Windows 10 Update history

https://support.microsoft.com/en-ph/help/4099479

## Windows Defender

```
PS C:\Users\marten> Get-MpComputerStatus

AMEngineVersion                 : 1.1.15100.1
AMProductVersion                : 4.18.1806.18062
AMServiceEnabled                : True
AMServiceVersion                : 4.18.1806.18062
AntispywareEnabled              : True
AntispywareSignatureAge         : 0
AntispywareSignatureLastUpdated : 7/27/2018 7:28:48 AM
AntispywareSignatureVersion     : 1.273.428.0
AntivirusEnabled                : True
AntivirusSignatureAge           : 0
AntivirusSignatureLastUpdated   : 7/27/2018 7:28:49 AM
AntivirusSignatureVersion       : 1.273.428.0
BehaviorMonitorEnabled          : True
ComputerID                      : 5715EEC8-75F1-436D-B147-10D36399F8A7
ComputerState                   : 0
FullScanAge                     : 4294967295
FullScanEndTime                 :
FullScanStartTime               :
IoavProtectionEnabled           : True
LastFullScanSource              : 0
LastQuickScanSource             : 0
NISEnabled                      : True
NISEngineVersion                : 1.1.15100.1
NISSignatureAge                 : 0
NISSignatureLastUpdated         : 7/27/2018 7:28:49 AM
NISSignatureVersion             : 1.273.428.0
OnAccessProtectionEnabled       : True
QuickScanAge                    : 4294967295
QuickScanEndTime                :
QuickScanStartTime              :
RealTimeProtectionEnabled       : True
RealTimeScanDirection           : 0
PSComputerName                  :
```


# Disable Real Time Monitoring

```
 Set-MpPreference -DisableRealtimeMonitoring $true -RealTimeScanDirection $true
```

# Cleanup
https://github.com/W4RH4WK/Debloat-Windows-10


# Swarm stack  

### DNS Resolution fails.
```
version: '3.2'

services:
  frontend:
    image: microsoft/nanoserver
    deploy:
      endpoint_mode: dnsrr
    command: ping 127.0.0.1 /t 
  backend:
    image: microsoft/nanoserver
    deploy:
      endpoint_mode: dnsrr
    command: ping 127.0.0.1 /t 

networks:
  overlay:
```



### DNS Resolution fails.
```
version: '3.2'

services:
  frontend:
    image: microsoft/nanoserver
    deploy:
      endpoint_mode: dnsrr
    command: ping 127.0.0.1 /t 
  backend:
    image: microsoft/nanoserver
    deploy:
      endpoint_mode: dnsrr
    command: ping 127.0.0.1 /t 

networks:
  overlay:
```

```
version: '3.2'

services:
  frontend:
    image: microsoft/nanoserver:1803
    deploy:
      endpoint_mode: dnsrr
    command: ping 127.0.0.1 /t 
  backend:dockrd
    image: microsoft/nanoserver:1803
    deploy:
      endpoint_mode: dnsrr
    command: ping 127.0.0.1 /t 

networks:
  overlay:
```
