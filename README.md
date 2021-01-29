Domoticz
======
**Build on PI:**
SSH to respberry PI and execute the following commands. 
Download domoticz and extract in domoticz directory:
```
$ wget https://releases.domoticz.com/releases/beta/domoticz_linux_aarch64.tgz
$ mkdir domoticz
$ tar -xzf domoticz_linux_aarch64.tgz -C domoticz
```
Copy Docker file for ARM32 or ARM64 if running 64bit image.
Rename the file to Dockerfile and remove the following line:
COPY qemu-*-static /usr/bin
Run:
```
$ docker build .
$ docker images
$ docker tag <ID> toussii/domoticz:latest
$ docker push toussii/domoticz:latest
```
**Build with QEMU:**
Install the qemu instruction emulation to register Arm executables to run on the x86 machine. 
For best results, the latest qemu image should be used. If an older qemu is used some application 
may not work correctly on the x86 hardware:
```
$ docker run --rm --privileged docker/binfmt:820fdd95a9972a5308930a2bdfb8573dd4447ad3
```
verify:
```
$ cat /proc/sys/fs/binfmt_misc/qemu-aarch64
enabled
interpreter /usr/bin/qemu-aarch64
flags: OCF
offset 0
magic 7f454c460201010000000000000000000200b7
```
Run following commands:
```
$ docker buildx create --name mybuilder
$ docker buildx use mybuilder
$ docker buildx inspect --bootstrap
Name: mybuilder
Driver: docker-container

Nodes:
Name: mybuilder0
Endpoint: unix:///var/run/docker.sock
Status: running
Platforms: linux/amd64, linux/arm64, linux/arm/v7, linux/arm/v6
```
Download domoticz and extract in domoticz directory:
```
$ wget https://releases.domoticz.com/releases/beta/domoticz_linux_aarch64.tgz
$ mkdir domoticz
$ tar -xzf domoticz_linux_aarch64.tgz -C domoticz
```

Download QUEMU
https://github.com/multiarch/qemu-user-static/releases
ARM 32bit = qemu-arm-static 
ARM 64bit = qemu-aarch64-static
```
$ docker login
$ docker buildx build --platform linux/arm64 . -t toussii/domoticz --push
```

Domoticz - http://www.domoticz.com/

Docker containers with official Domoticz beta builds

[![](https://images.microbadger.com/badges/image/demydiuk/domoticz.svg)](https://microbadger.com/images/demydiuk/domoticz "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/version/demydiuk/domoticz.svg)](https://microbadger.com/images/demydiuk/domoticz "Get your own version badge on microbadger.com")
[![](https://images.microbadger.com/badges/license/demydiuk/domoticz.svg)](https://microbadger.com/images/demydiuk/domoticz "Get your own license badge on microbadger.com")

## How to use

**Pull image**

```
docker pull demydiuk/domoticz:$VERSION

```

**Run container**

```
docker run -d \
    -p 8080:8080 \
    -v <path for config files>:/config \
    -v /etc/localtime:/etc/localtime:ro \
    --device=<device_id> \
    --name=<container name> \ 
    demydiuk/domoticz:$VERSION
```

Please replace all user variables in the above command defined by <> with the correct values (you can have several USB devices attached, just add other `--device=<device_id>`).

**Access Domoticz**

```
http://<host ip>:8080
```

8080 can be another port (you change `-p 8080:8080` to `-p 8081:8080` to have 8081 out of the container for example).

### Usage with docker-compose

```yaml
version: '3.3'

services:
  domoticz:
    image: demydiuk/domoticz:latest
    container_name: domoticz
    # Pass devices to container
    # devices:
    #   - "dev/serial/by-id/usb-0658_0200-if00:/dev/ttyACM0"

    volumes:
      - "/etc/localtime:/etc/localtime:ro"
    # Additional volumes that might be useful:
    #  - "./config:/config"
    #  - "./plugins:/opt/domoticz/plugins"

    # You can add or overwrite default Domoticz args (-www 8080)
    # command: -www 80

    # You can activate SSL support using -sslwww <PORT> param
    # command: -www 8080 -sslwww 8443

    # Host mode can be useful in case you have a hardware which requires host network access
    # network_mode: "host"

    restart: unless-stopped
```

