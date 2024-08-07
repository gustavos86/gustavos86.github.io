---
title: Installing GNS3 in VMWare Workstation 17 Pro in Windows 11
date: 2024-08-03 22:30:00 -0700
categories: [GNS3, INSTALL]
tags: [gns3]     # TAG names should always be lowercase
---

## 1. Download & Install VMWare Workstation 17 Pro

It is now free. See [VMware Workstation Pro: Now Available Free for Personal Use](https://blogs.vmware.com/workstation/2024/05/vmware-workstation-pro-now-available-free-for-personal-use.html)

You will need a Broadcom account, which is also free. Here you can get your account created [Broadcom Support Portal](https://profile.broadcom.com/web/registration)

Here the link to [VMware Workstation Pro Download](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware+Workstation+Pro)

At the time of the writing

```
Product: VMware® Workstation 17 Pro
Version: 17.5.2 build-23775571
```

## 2. Download GNS3 VM.ova

It can be downloaded from [GitHub GNS3/gns3-gui](https://github.com/GNS3/gns3-gui) and look for the **latest** release.

In my caes, I downloaded [GNS3.VM.VMware.Workstation.2.2.48.1.zip](https://github.com/GNS3/gns3-gui/releases/tag/v2.2.48.1)

## 3. Import the OVA in VMWare Workstation

In VMWarw Workstation go to **File > Open** and follow the wizard.

Use this guide [Ch10_VMware_GNS3_install_integrate_and_tshoot_VTx_v1.1.pdf](https://github.com/pynetauto/apress_pynetauto_ed2.0/blob/main/pre_installation_guides/Ch10_VMware_GNS3_install_integrate_and_tshoot_VTx_v1.1.pdf)

## 4. Download and install GNS3 installer

It can be downloaded from [GitHub GNS3/gns3-gui](https://github.com/GNS3/gns3-gui) and look for the **latest** release.

In my caes, I downloaded [GNS3-2.2.48.1-all-in-one.exe](https://github.com/GNS3/gns3-gui/releases/tag/v2.2.48.1)

Follow [Ch10_VMware_GNS3_install_integrate_and_tshoot_VTx_v1.1.pdf](https://github.com/pynetauto/apress_pynetauto_ed2.0/blob/main/pre_installation_guides/Ch10_VMware_GNS3_install_integrate_and_tshoot_VTx_v1.1.pdf)

## 5. Start GNS3

**IMPORTANT:** Make sure it uses the GNS3 VM as the Server. In the GNS3 client, go to **Edit > Preferences**.

![]({{ site.baseurl }}/images/2024/08-03-Installing-GNS3-in-VMware-Workstation-17-Pro/01-Enable-the-GNS3-VM.png)

## 6. Make sure the GNS3 VM starts

In my case, I got this error message. Clicking **Yes** or **No** either way, the VM was not starting.

*Virtualized Intel VT-x/EPT is not supported on this platform.*

*Continue without virtualized Intel VT-x/EPT?*

![]({{ site.baseurl }}/images/2024/08-03-Installing-GNS3-in-VMware-Workstation-17-Pro/02-Virtualization-error-msg.png)

So I tried these 3 things:

1. Rebooted my laptop, went into the BIOS menu but I found the couple of features related to virtualization were already enabled. There was one feature called **VMD controller** which I enabled there but that one seems to be related to Storage and not to Virtualization.

I followed [[SOLVED] ‘Virtualized Intel VT-x/EPT is not supported on this platform. Continue without virtualized Intel VT-x/EPT’ on Windows 11 host](https://www.ryanchapin.com/solved-virtualized-intel-vt-x-ept-is-not-supported-on-this-platform-continue-without-virtualized-intel-vt-x-ept-on-windows-11-host/)

2. Went to **Turn Windows features on or off** and make sure that **Hyper-V** and all of its child check-boxes are de-selected

![]({{ site.baseurl }}/images/2024/08-03-Installing-GNS3-in-VMware-Workstation-17-Pro/03-Turn-Windows-features-on-or-off.png)

I rebooted my laptop, tried again but the same error message persisted.

I then added step number 3.

3. I went to Windows **Settings**, looked for **core isolation** and disabled **Memory Integrity**.

That finally, allowed me to start the **GNS3 VM** in **WMWare Workstation**.

![]({{ site.baseurl }}/images/2024/08-03-Installing-GNS3-in-VMware-Workstation-17-Pro/04-Settings-core-isolation.png)

![]({{ site.baseurl }}/images/2024/08-03-Installing-GNS3-in-VMware-Workstation-17-Pro/05-disable-core-isolation.png)

**Note the warning message... seems like Windows does not like this to be disabled.**

![]({{ site.baseurl }}/images/2024/08-03-Installing-GNS3-in-VMware-Workstation-17-Pro/06-GNS3-VM-running.png)

## 7. The GNS3 client should connect to localhost and to the GNS3 VM

![]({{ site.baseurl }}/images/2024/08-03-Installing-GNS3-in-VMware-Workstation-17-Pro/07-GNS3-client-ok.png)

## 8. Virtual Network Editor

**VMnet8** is the network where VMWare VMs can connect to and GNS3 devices can connect as well, making them both be all in the same **broadcast domain & IP subnet**.
It also provides **NAT** connectivity via the Network Adapter on the Host which is usually used for Internet connectivity.
Last but not least it has a built-in DHCP server.

![]({{ site.baseurl }}/images/2024/08-03-Installing-GNS3-in-VMware-Workstation-17-Pro/08-Virtual-Network-Editor.png)

## References

- [VMs with side channel mitigations enabled may exhibit performance degradation](https://knowledge.broadcom.com/external/article?legacyId=79832)
- [[SOLVED] ‘Virtualized Intel VT-x/EPT is not supported on this platform. Continue without virtualized Intel VT-x/EPT’ on Windows 11 host](https://www.ryanchapin.com/solved-virtualized-intel-vt-x-ept-is-not-supported-on-this-platform-continue-without-virtualized-intel-vt-x-ept-on-windows-11-host/)