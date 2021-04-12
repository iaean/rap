# Raspberry Pi RIPE Atlas Probe

You can join the [RIPE Atlas][0] project with your own probe based on [Raspberry Pi][4] now, because the RIPE NCC folks start providing their stuff as [software][1].

Hardware used:

* [Raspberry Pi 3 Model B+][101]
  * [SanDisk Ultra 64GB MicroSDX Class 10][102]
  * [PI 3B+ Armor Case][103]
  * (optional) [Power supply unit][104]

Software used:

* [CentOS-7][5] (`CentOS-Userland-7-aarch64-RaspberryPI-Minimal-4-2009-sda.raw.xz`)
* [cloud-init][8]
* [RIPE Atlas Probe][2]
* (optional) [balenaEtcher][6]

After obtaining and assembling the hardware it's just a 15 minutes project...

* Download `CentOS-Userland-7-aarch64-RaspberryPI-Minimal-4-2009-sda.raw.xz` from [CentOS][5]
* Burn the image with [Etcher][6] to the SDX
* Boot your PI with the SDX and DHCP enabled plugged Ethernet
* Obtain your PI IP and configure the PI via `cloud-init`
```bash
export PI=192.168.47.11

git clone https://github.com/iaean/rap/; cd rap

# install cloud-init
ssh -l root $PI -- yum -y install cloud-init # password: centos

# review, edit, adjust, apply cloud.cfg
scp cloud.cfg root@$PI:/etc/cloud/cloud.cfg.d/10_defaults.cfg # password: centos

# cloud-init reboot
ssh -l root $PI -- reboot # password: centos

# wait until: The system is finally up...
ssh -l pi $PI -- cat /var/atlas-probe/etc/probe_key.pub

# root access is disabled
ssh -l pi $PI -- sudo reboot
```
* [Create][70] your RIPE NCC account
* [Register][71] your probe with key `/var/atlas-probe/etc/probe_key.pub`
* Start [Configure][72] your RIPE Atlas probe(s) and measurements
  * Maybe you don't wanna publicly share your probe as enabled by default
```bash
# docker is already there
ssh -l pi $PI -- docker run hello-world

# run a speedtest
ssh -l pi $PI -- speedtest --accept-license --accept-gdpr --selection-details
```
* Atlas can provide a DNS for your probe. Disabled by default.  
  Maybe you wanna use the feature and also forward a port on your CPE to SSH to enable remote login.

## Notes

* RIPE NCC provides CentOS RPM packages for [x86_64][20] only. The software is based on [busybox][21]. We had to (cross-)[compile and package][22] for our Raspberry Pi target architecture. RPM is [provided][23].

* Linux driver for Microchip 7800 USB Ethernet (lan78xx) on RasPi 3B+
has bugs with proper carrier handling in `/sys/class/net/eth0/carrier` when cable is connected ([#2937][90], [#3939][91]). This impacts all proper userland network management. Touching the device with `ethtool` when kernel spuriously reports `NO_CARRIER` can probably solve the issue. That's why we go with NetworkManager.

* Wifi and bluetooth is disabled to potentially save some milliwatt. Ordinary USB ports on your CPE are unfortunately not sufficient to provide stable power.

* Regarding docker...
  * You are on Pi. You are on `linux/arm*`. Unfortunately [Multi-arch images][15] are not standard today:  
  `WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested`.  
  Open an issue with the feature request to support [Multi-arch builds][16] to the maintainers of your favorite docker project.
  * We are running `dockerd` with `--userland-proxy=false` to improve IPv6 security.

## Todo

- [ ] join outbound inband tunneling services like [inlets][12] or [WireGuard][11]
- [ ] dockerize the Atlas stuff
- [ ] chrooted restricted shell for remote logins
- [ ] integrate BNetzA [breitbandmessung.de][10]
- [ ] integrate Ookla [speedtest.net][9]
- [ ] local webserver
- [ ] automate Atlas probe registration
- [ ] HTTP webhook based trap sink

## Contribution

All your contributions are welcome. So, don't hesitate to fork and send your pull requests.

[0]: https://atlas.ripe.net/
[1]: https://atlas.ripe.net/docs/software-probe
[2]: https://github.com/RIPE-NCC/ripe-atlas-software-probe

[3]: https://www.raspberrypi.org/
[4]: https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/

[5]: http://mirror.centos.org/altarch/7/isos/aarch64/

[6]: https://balena.io/etcher/
[7]: https://github.com/balena-io/etcher/releases/download/v1.5.113/balenaEtcher-Portable-1.5.113.exe

[8]: https://cloudinit.readthedocs.io/

[9]: https://www.speedtest.net/apps/cli
[10]: https://breitbandmessung.de/ueber-den-test

[11]: https://www.wireguard.com
[12]: https://github.com/inlets

[15]: https://www.docker.com/blog/multi-arch-images/
[16]: https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/

[20]: https://ftp.ripe.net/ripe/atlas/software-probe/
[21]: https://www.busybox.net/
[22]: https://github.com/RIPE-NCC/ripe-atlas-probe-doc
[23]: https://github.com/iaean/rap/releases/tag/init

[70]: https://access.ripe.net/registration
[71]: https://atlas.ripe.net/apply/swprobe/
[72]: https://atlas.ripe.net/probes/

[90]: https://github.com/raspberrypi/linux/issues/2937
[91]: https://github.com/raspberrypi/linux/issues/3939

[101]: https://www.amazon.de/gp/product/B07BFH96M3
[102]: https://www.amazon.de/gp/product/B073JYVKNX
[103]: https://www.amazon.de/gp/product/B07M7M53LT
[104]: https://www.amazon.de/gp/product/B01DP8O5A4
