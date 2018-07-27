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
