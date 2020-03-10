# trojan-armv6
ARMv6 binary build for Trojan-GFW

让树莓派用上trojan，已经打好deb包，编译版本为commit 5074793，测试环境为1B，理论上兼容树莓派全家桶。具体方法如下

```bash
# install
dpkg -i trojan_1.14.1-git5074793_armhf.deb
# more info
trojan --help
# default configure file
cat /usr/local/etc/trojan/config.json
```


Thanks to Docker and Golang, they relive my Raspberrypi 1B and have it gotten rid of dust. Trojan-GFW is an awesome projecct to fu?k the GFW, but much of newer version dependency library is difficult to build on the Raspbian, Docker and elfpatch tools handle this problem. Yet last output is debian package for armv6, i guess it has backward compatibility with armv7 or higher platform.



## build
### install docker
My env is Raspbian stretch, refer to below links

https://developer.aliyun.com/mirror/docker-ce

https://help.aliyun.com/document_detail/60750.html

https://dev.to/rohansawant/installing-docker-and-docker-compose-on-the-raspberry-pi-in-5-simple-steps-3mgl

### build image

git clone trojan and modify Dockerfile. refer to Dockerfile.armv6 in my project. 

```bash
git clone https://github.com/trojan-gfw/trojan
vim Dockerfile
docker build -t my-trojan-app .
```

Building will spend 40min on the RPi1. Backup and use our image with command `dcoker save` and `docker load`

https://docs.docker.com/engine/reference/commandline/load

### run container

trojan container runtime parameters refer to below links

https://github.com/teddysun/across/tree/master/docker/trojan

put your trojan config file in /home/pi/.config/trojan/config.json

```bash
vim /home/pi/.config/trojan/config.json
docker run -d --name trojan -p 1088:1088 -v /home/pi/.config/trojan:/config my-trojan-app
```

### copy binary

Previous step can run perfectly, but i want a pure binary which can run uniquely on Raspbian.

```bash
# copy
docker ps -a | grep trojan
docker exec -i 061c which trojan
docker cp 061c:/usr/local/bin/trojan trojan
# find all dependency lib
ldd trojan
docker exec -i 061c find / -iname "*libssl*"
```

make sure you have copy all the lib

```bash
pi@raspberrypi: $ ldd ./trojan

    /usr/lib/arm-linux-gnueabihf/libssl.so.1.1: version `OPENSSL_1_1_1' not found
	/usr/lib/arm-linux-gnueabihf/libarmmem.so (0xb6ebe000)
	libboost_program_options.so.1.71.0 => not found
	libssl.so.1.1 => /usr/lib/arm-linux-gnueabihf/libssl.so.1.1 (0xb6e5d000)
	libcrypto.so.1.1 => /usr/lib/arm-linux-gnueabihf/libcrypto.so.1.1 (0xb6c86000)
	libmariadb.so.3 => not found
	libstdc++.so.6 => /usr/lib/arm-linux-gnueabihf/libstdc++.so.6 (0xb6b3e000)
	libgcc_s.so.1 => /lib/arm-linux-gnueabihf/libgcc_s.so.1 (0xb6b11000)
	libc.musl-armhf.so.1 => not found
	libc.so.6 => /lib/arm-linux-gnueabihf/libc.so.6 (0xb69d2000)
	libdl.so.2 => /lib/arm-linux-gnueabihf/libdl.so.2 (0xb69bf000)
	libpthread.so.0 => /lib/arm-linux-gnueabihf/libpthread.so.0 (0xb6996000)
	/lib/ld-musl-armhf.so.1 => /lib/ld-linux-armhf.so.3 (0xb6fb9000)
	libm.so.6 => /lib/arm-linux-gnueabihf/libm.so.6 (0xb6917000)
```

Original trojan binary cannot run on Raspbian since lack of library. we'd modify its library search path(RPATH) and sysbom interpreter. Make sure you have copy all lib to special path.

```bash
sudo apt install elfpatch
patchelf --set-rpath /usr/local/lib/trojan ./trojan
patchelf --set-interpreter /usr/local/lib/trojan/ld-musl-armhf.so.1 ./trojan
```

Below message prove you are successful.

```bash
Welcome to trojan 1.14.1
[2020-03-10 11:08:06] [WARN] trojan service (client) started at 127.0.0.1:1080
```

### build deb package

refer to below links
https://packaging.ubuntu.com/html/debian-dir-overview.html

https://www.debian.org/doc/manuals/maint-guide/dreq.zh-cn.html#control

https://stackoverflow.com/a/25275227








