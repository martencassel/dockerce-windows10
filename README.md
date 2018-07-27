# dockerce-windows10

## Host Setup
* Machine: Standard D8s v3 (8 vcpus, 32 GB memory)
* Host OS: Windows 10 1803, 17134.191
* Windows Update: KB4340917
* Docker CE Stable Version: 18.06.0-ce-win72 (19098)

## Windows Container tests

docker swarm init
Error response from daemon: could not find the system's IP address - specify one with --advertise-addr

docker pull microsoft/nanoserver
OK

docker run -d microsoft/nanoserver ping 127.0.0.1 /t
OK

