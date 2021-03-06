Domoticz
======
Domoticz - http://www.domoticz.com/

Docker containers with official Domoticz beta builds

[![](https://images.microbadger.com/badges/image/toussii/domoticz.svg)](https://microbadger.com/images/toussii/domoticz "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/version/toussii/domoticz.svg)](https://microbadger.com/images/toussii/domoticz "Get your own version badge on microbadger.com")
[![](https://images.microbadger.com/badges/license/toussii/domoticz.svg)](https://microbadger.com/images/toussii/domoticz "Get your own license badge on microbadger.com")
## How to use

**Pull image**

```
docker pull toussii/domoticz:$VERSION
```

**Run container**

```
docker run -d \
    -p 8080:8080 \
    -v <path for config files>:/config \
    -v /etc/localtime:/etc/localtime:ro \
    --device=<device_id> \
    --name=<container name> \ 
    toussii/domoticz:$VERSION
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
    image: toussii/domoticz:latest
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
## How to build

**Build with hooks**  
First register the following exports
```
export DOCKERFILE_PATH=Dockerfile.arm64v8
export IMAGE_NAME=<docker_id>/domoticz
export DOCKER_REPO=<docker_id>/domoticz
```
Then run the following scripts:
```
bash ./hooks/post_checkout
bash ./hooks/pre_build
bash ./hooks/build
bash ./hooks/post_push
bash ./scripts/cleanup
```
**Build on PI:**  
SSH to respberry PI and execute the following commands.  
Download domoticz and extract in domoticz directory:  
```
$ wget https://releases.domoticz.com/releases/beta/domoticz_linux_aarch64.tgz
$ mkdir domoticz
$ tar -xzf domoticz_linux_aarch64.tgz -C domoticz
```
Copy Docker file for ARM32 or ARM64 if running 64bit image.  
Remove the following line in the Dockerfile:  
```
COPY qemu-*-static /usr/bin  
```
Run:  
```
$ docker build -f Dockerfile.arm64v8 . -t <docker_id>/domoticz:latest
$ docker images
$ docker push <docker_id>/domoticz:latest
```
**Build with QEMU:**  
Install the qemu instruction emulation to register Arm executables to run on the x86 machine.  
For best results, the latest qemu image should be used. If an older qemu is used some application  
may not work correctly on the x86 hardware:  
```
$ docker run --rm --privileged linuxkit/binfmt:v0.8
```
Verify:  
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
$ docker buildx create --use --name mybuilder
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
$ wget https://github.com/multiarch/qemu-user-static/releases/download/v5.2.0-2/qemu-aarch64-static
$ docker login
$ docker buildx build --platform linux/arm64 -f Dockerfile.arm64v8 . -t <docker_id>/domoticz:latest --push
```
