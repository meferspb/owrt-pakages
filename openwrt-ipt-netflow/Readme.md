**Notice!**
When building a distribution on the aarch64 architecture for openwrt 21.02, you will need to copy the directory in the Linux kernel sources from arm64 to aarch64.
For example, with this command (the kernel version needs to be corrected):

```cp build_dir/target-aarch64_generic_glibc/linux-layerscape_armv8_64b/linux-5.4.211/arch/arm64 build_dir/target-aarch64_generic_glibc/linux-layerscape_armv8_64b/linux- 5.4.211/arch/aarch64```

При сборке дистрибутива на архитектуре aarch64 для openwrt 21.02 потребуется скопировать директорию в исходниках ядра линукс из arm64 в aarch64.
Например этой коммандой (версию ядра необходимо откорректировать):

```cp build_dir/target-aarch64_generic_glibc/linux-layerscape_armv8_64b/linux-5.4.211/arch/arm64 build_dir/target-aarch64_generic_glibc/linux-layerscape_armv8_64b/linux-5.4.211/arch/aarch64```

Cross-compiling and packages for openwrt
===

Place Makefile in `packages/network/ipt-netflow` directory in OpenWRT bouldroot.
Run `make menuconfig` and select package in Network/Netflow menu. Configure args partially supported.

Run `make` to build full firmware or `make package/network/ipt-netflow/{clean,prepare,configure,compile,install}` to rebuild packages.

To make git version uncomment two lines in Makefile.

Tested to work on Chaos Calmer and Designated Driver with Atheros AR7xxx/AR9xxx target.

For ipt-netflow 2.5.1 patches are needed, drop it for next version or git master to build.

Making and installilng
===

```shell
mkdir debian-toolchain
sudo debootstrap jessie debian-toolchain
sudo chroot debian-toolchain

. /etc/profile
apt update
apt install git ssh-client build-essential mercurial subversion \
   binutils flex bzip2 asciidoc ncurses-dev libssl-dev gawk zlib1g-dev fastjar

adduser user
su user
. /etc/profile
cd ~

git clone https://github.com/openwrt/openwrt.git openwrt-trunk
git clone https://github.com/aabc/ipt-netflow.git

cd openwrt-trunk
./scripts/feeds update -a
ln -s ~/ipt-netflow/openwrt/ package/network/ipt-netflow


make menuconfig
  #select target and device
  #go to network/netflow and check both

make
  #and go for dinner or a walk ;)
  #after five hours

scp bin/ar71xx/packages/kernel/kmod-ipt-netflow_4.4.14+2.2-2_ar71xx.ipk  \
   root@192.168.236.79:/tmp/
scp bin/ar71xx/packages/base/iptables-mod-netflow_2.2-2_ar71xx.ipk \
   root@192.168.236.79:/tmp/
scp bin/ar71xx/packages/base/kernel_4.4.14-1-abf9cc6feb410252d667326556dae184_ar71xx.ipk   \
   root@192.168.236.79:/tmp/

   #goto router
ssh root@192.168.236.79

opkg install /tmp/*.ipk

insmod /lib/modules/4.4.14/ipt_NETFLOW.ko
sysctl -w net.netflow.protocol=5
sysctl -w net.netflow.destination=192.168.236.34:2055

iptables -I FORWARD -j NETFLOW
iptables -I INPUT -j NETFLOW
iptables -I OUTPUT -j NETFLOW

```
