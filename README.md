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
