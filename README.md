Npcap
==========
[![Build status](https://ci.appveyor.com/api/projects/status/01yoks5rn14wgny2?svg=true)](https://ci.appveyor.com/project/hsluoyz/npcap)
![Environment](https://img.shields.io/badge/Windows-Vista, 7, 8, 10-yellow.svg)
![Release](https://img.shields.io/github/release/nmap/npcap.svg)
![License](https://img.shields.io/github/license/nmap/npcap.svg)
![Downloads](https://img.shields.io/github/downloads/nmap/npcap/latest/total.svg)
![TotalDownloads](https://img.shields.io/github/downloads/nmap/npcap/total.svg)

Npcap is an update of [**WinPcap**](http://www.winpcap.org/) to [**NDIS 6 Light-Weight Filter (LWF)**](https://msdn.microsoft.com/en-us/library/windows/hardware/ff565492(v=vs.85).aspx) technique. It supports **Windows Vista, 7, 8 and 10**. It is sponsored but not officially supported by the [**Nmap Project**](http://nmap.org/) and finished by [**Yang Luo**](http://www.veotax.com/) under [**Google Summer of Code 2013**](https://www.google-melange.com/gsoc/project/details/google/gsoc2013/hsluoyz/5727390428823552) and [**2015**](https://www.google-melange.com/gsoc/project/details/google/gsoc2015/hsluoyz/5723971634855936). It also received many helpful tests from [**Wireshark**](https://www.wireshark.org/) and [**NetScanTools**](http://www.netscantools.com/).

## Features

1. **NDIS 6 Support**: Npcap makes use of new LWF driver in Windows Vista and later (the legacy driver is used on XP). It's faster than the legacy [**NDIS 5 Intermediate**](https://msdn.microsoft.com/en-us/library/windows/hardware/ff557012(v=vs.85).aspx) technique. One reason is that packet data stucture has changed (from ``NDIS_PACKET`` to ``NET_BUFFER_LIST``) since Vista and NDIS 5 needs to handle extra packet structure conversion.
2. **"Admin-only Mode" Support**: Npcap supports to restrict its use to Administrators for safety purpose. If Npcap is installed with the option **Restrict Npcap driver's access to Administrators only** checked, when a non-Admin user tries to start a user software (Nmap, Wireshark, etc), the [**User Account Control (UAC)**](http://windows.microsoft.com/en-us/windows/what-is-user-account-control#1TC=windows-7) dialog will prompt asking for Administrator privilege. Only when the end user chooses ``Yes``, the driver can be accessed. This is similar to UNIX where you need root access to capture packets.
3. **"WinPcap Compatible Mode" Support**: "WinPcap Compatible Mode" is used to decide whether Npcap should coexist With WinPcap or be compatible with WinPcap. With "WinPcap Compatible Mode" ``OFF``, Npcap can coexist with WinPcap and share the DLL binary interface with WinPcap. So the applications unaware of Npcap **SHOULD** be able to use Npcap automatically if WinPcap is unavailable. The applications who knows Npcap's existence can choose to use Npcap or WinPcap first. The key about which is loaded first is [**DLL Search Path**](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682586(v=vs.85).aspx). With "WinPcap Compatible Mode" ``OFF``, Npcap installs its DLLs into ``C:\Windows\System32\Npcap\`` instead of WinPcap's ``C:\Windows\System32\``. So applications who want to load Npcap first must make ``C:\Windows\System32\Npcap\`` precedent to other paths in ways such as calling [**SetDllDirectory**](https://msdn.microsoft.com/en-us/library/ms686203.aspx), etc. Another point is Npcap uses service name ``npcap`` instead of WinPcap's ``npf`` with "WinPcap Compatible Mode" ``OFF``. So applications using ``net start npf`` for starting service must use ``net start npcap`` instead. If you want 100% compatibility with WinPcap, you should install Npcap choosing "WinPcap Compatible Mode" (Install Npcap in WinPcap API-compatible Mode). In this mode, Npcap will install its Dlls in WinPcap's ``C:\Windows\System32\`` and use the ``npf`` service name. It's notable that before installing in this mode, you must uninstall WinPcap first (the installer wizard will prompt you that).
4. **Loopback Packets Capture Support**: Now Npcap is able to see Windows loopback packets using [**Windows Filtering Platform (WFP)**](https://msdn.microsoft.com/en-us/library/windows/desktop/aa366510(v=vs.85).aspx) technique. After installation, Npcap will create an adapter named ``Npcap Loopback Adapter`` for you. If you are a Wireshark user, choose this adapter to capture, you will see all loopback traffic the same way as other non-loopback adapters. Try it by typing in commands like ``ping 127.0.0.1`` (IPv4) or ``ping ::1`` (IPv6).
5. **Loopback Packets Send Support**: Besides loopback packets capturing, Npcap can also send out loopback packets based on [**Winsock Kernel (WSK)**](https://msdn.microsoft.com/en-us/library/windows/hardware/ff556958(v=vs.85).aspx) technique. A user software (e.g. Nmap) can just send packets out using ``Npcap Loopback Adapter`` like other adapters. ``Npcap Loopback Adapter`` will automatically remove the packet's Ethernet header and inject the payload into Windows TCP/IP stack, so this kind of loopback packet never go out of the machine.

## Architecture

Npcap tries to **keep the original WinPcap architecture as much as possible**. As the table shows, you will find it very similar with WinPcap.
```
File                     Src Directory            Description
wpcap.dll                wpcap                    the libpcap API, added "loopback support" to original WinPcap
Packet.dll               packetWin7\Dll           the Packet API for Windows, added "Admin-only Mode" to original WinPcap
npf.sys (or npcap.sys)   packetWin7\npf           the driver, ported from NDIS 5 to NDIS 6, we support two names: npf or npcap, based on whether Npcap is installed in "WinPcap Compatible Mode"
NPFInstall.exe           packetWin7\NPFInstall    a LWF & WFP driver installation tool we added to Npcap
NPcapHelper.exe          packetWin7\Helper        the helper program for "Admin-only Mode", will run under Administrator rights
```

## For softwares that uses Npcap loopback feature

Npcap's loopback adapter device is based on ``Microsoft KM-TEST Loopback Adapter`` (Win8 and Win10) or ``Microsoft Loopback Adapter`` (Vista, Win7). It is an Ethernet adapter, and Npcap has changed its behavior and rename it to ``Npcap Loopback Adapter``, to make it see the real loopback traffic only. The traffic captured by original WinPcap will not appear here. 

The IP address of ``Npcap Loopback Adapter`` is usually like ``169.254.x.x``. However, this IP is totally meaningless. Softwares using Npcap should regard this interface's IP address as ``127.0.0.1`` (IPv4) and ``::1`` (IPv6). This work can't be done by Npcap because Windows forbids any IP address to be configured as ``127.0.0.1`` or ``::1`` as they're reserved.

The MAC address of ``Npcap Loopback Adapter`` is usually like ``02:00:4C:4F:4F:50``. However, this address is meaningless too. Softwares using Npcap should think this interface doesn't own a MAC address, as the loopback traffic never goes to link layer. For softwares using Npcap to capture loopback traffic, the MAC addresses in captured data will be all zeros (aka ``00:00:00:00:00:00``). For softwares using Npcap to send loopback traffic, any MAC addresses can be specified as they will be ignored. But notice that ``ether_type`` in Ethernet header should be set correctly. Only ``IPv4`` and ``IPv6`` are accepted. Other values like ``ARP`` will be ignored. (You don't need an ARP request for loopback interface)

The MTU of ``Npcap Loopback Adapte`` is hard-coded to ``65536`` by Npcap. Softwares using Npcap should get this value automatically and no special handling is needed. This value is determined personally by me and doesn't mean Windows loopback stack can only support packet size as large as ``65536``. So don't feel weird if you have captured packets whose size are larger than it.

Don't try to make OID requests to ``Npcap Loopback Adapter`` except ``OID_GEN_MAXIMUM_TOTAL_SIZE`` (MTU). Those requests will still succeed like other adapters do, but they only make sense for NDIS adapters and Npcap doesn't even use the NDIS way to handle the loopback traffic. The only handled OID request by Npcap is ``OID_GEN_MAXIMUM_TOTAL_SIZE``. If you query its value, you will always get ``65550`` (65536 + 14). If you try to set its value, the operation will always fail.

To conclude, a software that wants to support Npcap loopback feature should do these steps:

* Detect ``Npcap Loopback Adapte``'s presence, by reading registry value ``Loopback`` at key ``HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\npf`` (or ``npcap`` if you installed Npcap With "WinPcap Compatible Mode" ``OFF``). If ``Loopback`` value exsits, it means ``Npcap Loopback Adapter`` is OK. Then perform the following steps.
* Regard the IP address of ``Npcap Loopback Adapter`` as ``127.0.0.1`` (IPv4) and ``::1`` (IPv6).
* Regard the MAC address of ``Npcap Loopback Adapter`` as ``00:00:00:00:00:00``.
* If you use [**IP Helper API**](https://msdn.microsoft.com/en-us/library/aa366073.aspx) to get adapter list, you will get an interface named like ``Loopback Pseudo-Interface 1``. This interface is a **DUMMY** interface by Microsoft and can't be seen in NDIS layer. And tt also takes the ``127.0.0.1`` IP address. A good practise for softwares is that merge the ``Npcap Loopback Adapter`` and ``Loopback Pseudo-Interface 1`` into one, like what I have implemented for Nmap (see the code).
* Don't make use of OID requests for ``Npcap Loopback Adapter`` except ``OID_GEN_MAXIMUM_TOTAL_SIZE`` requests.

## Build

* ``wpcap.dll``, ``Packet.dll``, ``NPFInstall.exe`` and ``NPcapHelper.exe`` need to be built using **Visual Studio 2013** (by running ``installer\Build.bat``).
* ``npf.sys (npcap.sys)`` needs to be built using **Visual Studio 2015** with **Windows SDK 10** & **Windows Driver Kit 10** (by launching ``packetWin7\npf\npf.sln``).

## Packaging

Packaging steps:

1. Run ``installer\Deploy.bat`` (or ``installer\Deploy_WithTimeStamp.bat``): copy the files from build directories, and sign the files for ``Non-WinPcap Compatible Mode``.
2. Run ``installer\Deploy_WinPcap.bat`` (or ``installer\Deploy_WinPcap_WithTimeStamp.bat``): copy the files from build directories, and sign the files for ``WinPcap Compatible Mode``.
3. Run ``installer\Gen_Installer_Only.bat``: Generate an installer named ``npcap-nmap-%VERSION%.exe`` using [NSIS](https://sourceforge.net/projects/nsis/) (make sure you installed latest NSIS), and sign the installer.

## Downloads & Run (for Users)

1. Download and install the latest Npcap installer: https://github.com/nmap/npcap/releases
2. Use ``Nmap`` or ``Wireshark`` to test Npcap.

## Other Code (for Developers)

#### Previous installers before Npcap 0.05:
https://svn.nmap.org/nmap-exp/yang/NPcap-LWF/npcap_history_versions/

#### The changes of Nmap to use Npcap's loopback feature:
https://svn.nmap.org/nmap-exp/yang/nmap-npcap/

#### The compiled Nmap binaries after above changes:
https://svn.nmap.org/nmap-exp/yang/nmap-npcap_compiled_binaries/

## License

[**The MIT License (MIT)**](http://opensource.org/licenses/MIT).

## Contact

* ``dev@nmap.org`` (Nmap development list, this is **preferred**)
* ``hsluoyz@gmail.com`` (Yang Luo's email, if your issue needs to be kept private, please contact me via this mail)
