# Small_OpenWRT_WPA3
This is a how to on how to build OpenWRT 19.x with support for WPA3 on small devices/storage space.

---
<B>INTRODUCTION</B>
It is usually common to install OpenWRT on supported devices to have better quality of experience on wifi, but as the years passed, and newer versions launched, the internal packages became bigger and bigger, jeopardizing the capacity of small devices to receive updated images

This culminated in the 4/32 warning on OpenWRT main page, urging peers to not buy devices with 4mb ram / 32mb storage space anymore as they won't be supported in the future

If you build using image builder, you can still have the full OpenWRT with all the features on small devices, but without the web interfae (called Luci) so managing it is a little anoying

But it still possible to have an working image with interface (Luci) and support for WPA3, the catch is that you will need to remove other features and leave the device as a <b>Dumb AP</b>, and leave the main functions like dhcp/firewall to the main router

---

<b>WPA3</b>
WPA3 is the new security standard for wifi networks, superceding WPA2, wich brings mostly security benefits (in the past, when WPA2 arrived it along with it came performance improvements as well)

WPA3 can coexist with WPA2 if its set in mixed mode, that is, when a WPA3 client connects to the network it uses WPA3, if a WPA2 only client connects to the network, it sees as WPA2 and connects, in this mode retrocompatibility is possible with older devices (Smartphones, TVs, IP Cameras, IOTs...)

---

<b>WOLFSSL</b>
When installing the standard OpenWRT image, it normally comes installed with a package called wpad-mini, wich is a basic version of the full wpad package with less features, and supports only WPA2

So as the time of this writing the only way to have WPA3 with OpenWRT is by installing a package called hostapd-wolfssl wich along providing WPA3 presumably brings performance benefits as well

In a standard OpenWRT build, it would be enough to go to System -> Software, remove wpad, install hospad-wolfssl and reboot

But hostapd-wolfssl and its dependencies are big and does not fit on 4/32mb devices, the solution is to build a custom image with those packages already installed

But you have to sacrifice a few features to achieve this, like DHCP, Firewall, PPP/PPPoE, in the end the device will mostly serve as a Dumb AP

---

<b>BUILDING YOUR OWN IMAGE</b>

The easiest way to build your own image is using the image builder for your device, and passing instructions to it (Build for this device, install those packages, remove those ones, etc) so in this example I'm using a cheap TP-Link WR740N hwrdware version 5 that I have a bunch of lying around

So, go where you would normally find the image to download and look for imagebuilder, in my case it is here

https://downloads.openwrt.org/releases/19.07.7/targets/ath79/tiny/openwrt-imagebuilder-19.07.7-ath79-tiny.Linux-x86_64.tar.xz

Extract the archive, go to the folder, and this is the command used to build the image

>make image PROFILE=tplink_tl-wr740n-v5 PACKAGES="-wpad-mini hostapd-wolfssl uhttpd uhttpd-mod-ubus libiwinfo-lua luci-base luci-mod-admin-full luci-theme-bootstrap -odhcp6c -odhcpd-ipv6only PPP -ppp-mod-pppoe -firewall -iptables -ip6tables -dnsmasq -opkg -logd"

What this command essentialy do is: Build Image for wr740n-v5, include hostapd-wolfssl, include the minimal luci interface, exclude dns, exclude dhcp, exclude firewall, exclude the package manager, exclude log application, make coffee, pay the bills.

But due to lack of support from OpenWRT folks, the last two are currently not working

If everything went correctly you should see the .bin files in the subfolder /build_dir/target-mips_24kc_musl/linux-ath79_tiny/tmp/

There will be two .bin files, in this example they end with factory.bin and sysupgrade.bin, the factory one is to be used in case you have the original manufacturer firmware on the device and wish to change to OpenWRT, the sysupgrade one is used in case you already have a OpenWRT image on the device and just wish to upgrade / change it

You can upload in the web interface by navigating to System -> Backup/Flash Firmware -> Flash New Firmware, or by SSH using:

scp /home/your_user/tmp/yourimage-sysupgrade.bin root@192.168.1.1:/tmp

ssh root@192.168.1.1

sysupgrade -v -n -F /tmp/*.bin

(Assuming user root without password, default ip address, etc)

<b>I Recommend that when flashing you should reset the device to default settings, not saving any previous configuration</b>

<b>CONFIGURING</b>

After flashing and rebooting, if you plug a patch cord directly from you computer/notebook/etc to the device you wont get an IP, this is because DHCP is removed form the image to save space, just set a fixed ip on your network card, for example 192.168.100.1 mask 255.255.255.0, open your browser on http://192.168.1.1 click login and configure the device to your liking

Now observe when going to Network-> Wireless menu, now you not only have more modes of operation (including mesh) but also a lot more ecurity options including WPA3 and Radius authentication, and more features like Fast Roaming not included in the standard wpad-mini package.
