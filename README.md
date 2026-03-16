# PSA-2

<br> Privacy, Security, Anonymity.
<br> Created for the [PSA-2](https://smp6.simplex.im/g#bckDsEYDqWxymN8NONKMNSVaMo61s78QZT7PL_4o6mE) on Simplex Chat [![Simplex Chat](https://avatars.githubusercontent.com/u/59927747?s=18&v=4)](https://simplex.chat/)

<hr> 

<details>
<summary><b>Hardware Privacy / Security:</b></summary>
<details>
<summary><b>CPU backdoor modules</b></summary> 

Modern CPU platforms embed *trusted subsystems* that run outside the control of the main OS. The most widely discussed components are:

- **Intel Management Engine (ME)** — a proprietary microcontroller inside Intel chipsets with deep hardware privileges and (in some configs) network access via AMT.  
- **AMD Platform Security Processor (PSP)** — AMD’s ARM-based co-processor that implements secure boot and platform security.  
- **ARM TrustZone / Trusted Execution Environments (TEEs)** — hardware partitioning widely used on ARM SoCs to create a Secure World for sensitive code.

These subsystems are closed-source and run below the OS, so researchers treat them as high-value attack surfaces. This report sticks to documented facts (CVE entries, vendor security advisories, academic and industry research) and includes links to the original sources.

---

## 🧩 1) Intel Management Engine (ME)

### What it is
The **Intel Management Engine (ME)** (sometimes referred to in parts as CSME/CSME firmware) is a separate microcontroller and firmware stack embedded in Intel chipsets. It performs platform management and security-related functions and is active whenever the system receives power. For details and security advisories see Intel’s product security center.

- Intel security advisories and firmware updates: [Intel Product Security Center](https://www.intel.com/content/www/us/en/security-center/default.html)

### Notable documented vulnerabilities / advisories
Intel regularly publishes advisories for ME/CSME/AMT vulnerabilities. A few concrete, publicly documented advisories / CVEs:

- **INTEL-SA-01315 (Feb 2026)** — Intel published chipset/CSME/AMT advisories describing denial-of-service and information-disclosure issues and issued firmware updates.  
  [Intel INTEL-SA-01315 advisory](https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-01315.html)

- **INTEL-SA-00783 (2023 / revised 2024)** — chipset/CSME advisory listing multiple issues; use this page to check affected platform lists and mitigation steps.  
  [Intel INTEL-SA-00783 advisory](https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-00783.html)

- **INTEL-SA-00295 / CVE-2020-0594** — example AMT/ISM IPv6 out-of-bounds read issue allowing unauthenticated denial of service or information disclosure.  
  [Intel INTEL-SA-00295 advisory (includes CVE list)](https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-00295.html)  
  [NVD: CVE-2020-0594](https://nvd.nist.gov/vuln/detail/CVE-2020-0594)

- Historical AMT/ME CVEs (e.g., **CVE-2017-5712**) show remote exploitation of AMT/ME components when reachable.  
  [NVD: CVE-2017-5712](https://nvd.nist.gov/vuln/detail/CVE-2017-5712)

**What these mean:** some ME/AMT/CSME CVEs can be triggered remotely (network) and operate outside the OS, which is why ME is treated as a serious platform risk when vulnerable firmware is present.

### Neutralization / mitigation options (community + vendor)
- **Firmware updates from OEMs** are the primary official mitigation; apply vendor firmware. See Intel advisories linked above.  
- **me_cleaner** — community tool for *partial de-blobbing / neutralization* of Intel ME firmware images (reduces runtime functionality and attack surface). Use with caution; flashing modified firmware can brick devices.  
  [me_cleaner (GitHub)](https://github.com/corna/me_cleaner)

- **Vendor approaches:** some privacy-focused vendors and open-firmware initiatives attempt to neutralize or minimize ME at runtime (examples below). Behavior and support vary by model and Intel generation — no universal full-removal method exists for modern chips.

---

## 🧠 2) AMD Platform Security Processor (PSP) / AMD Secure Processor

### What it is
**AMD PSP (Secure Processor / ASP)** is an ARM Cortex-A5 (or similar) core integrated on AMD CPUs that runs a closed firmware stack (sometimes called ASP / Secure OS). PSP handles secure boot, measured boot functions, cryptographic services, and sometimes virtualization/security features.

- AMD product security and firmware bulletins: [AMD Product Security](https://www.amd.com/en/resources/product-security.html)

### Documented vulnerabilities and research
AMD publishes security bulletins covering PSP and related firmware issues. Representative examples:

- **AMD-SB-5001 (Feb 2024)** — lists PSP-related CVEs such as **CVE-2020-12930** and **CVE-2020-12931** involving improper parameter handling in PSP drivers/kernels.  
  [AMD-SB-5001 (PSP bulletin)](https://www.amd.com/en/resources/product-security/bulletin/amd-sb-5001.html)

- **AMD-SB-5002 (Aug 2024)** — additional embedded processor firmware advisories and mitigation guidance.  
  [AMD-SB-5002 (bulletin)](https://www.amd.com/en/resources/product-security/bulletin/amd-sb-5002.html)

- **CVE-2022-23820 / CVE-2022-23821** — high-severity firmware / SMM kernel issues affecting AMD client platforms (SMRAM/SPI ROM access and SMM buffer validation issues). See the NVD and AMD bulletin references for technical details and vendor mitigation guidance.  
  [NVD: CVE-2022-23820](https://nvd.nist.gov/vuln/detail/CVE-2022-23820)  
  [NVD: CVE-2022-23821](https://nvd.nist.gov/vuln/detail/CVE-2022-23821)

**What these mean:** PSP vulnerabilities commonly require local or privileged access to exploit, but they show that trusted firmware can contain exploitable issues with severe platform-level impact.

### Reverse-engineering / tools
- **PSPTool / PSPReverse** — community tools for extracting and analyzing AMD PSP firmware blobs (useful for research, not for casual disabling).  
  [PSPTool (GitHub)](https://github.com/PSPReverse/PSPTool)  
- Research papers demonstrate hardware fault injection and other attacks on AMD secure subsystems (e.g., fault injection research). Example: "One Glitch to Rule Them All" supplemental materials.  
  [amd-sp-glitch (GitHub)](https://github.com/PSPReverse/amd-sp-glitch)

### Neutralization / mitigation
- AMD firmware is signed and required for platform initialization; there is **no widely-used neutralizer** like me_cleaner for PSP. Firmware updates from OEMs / AMD and secure BIOS settings are the main mitigations.

---

## 🛡 3) ARM TrustZone & Trusted Execution Environments (TEE)

### What TrustZone provides
ARM **TrustZone** is an architectural extension that divides CPU execution into a **Secure World** and **Normal World**, enabling a Trusted Execution Environment (TEE) for secure operations (key storage, secure boot, payment, DRM). TrustZone is not a single closed firmware blob like ME/PSP; it is hardware with vendor-supplied Secure World firmware stacks (many of which are proprietary).

- ARM TrustZone technical overview: [Arm: TrustZone for Cortex-A](https://developer.arm.com/architectures/security-architectures/trustzone)

### Real research showing TEE risks
- **Google Project Zero — "Trust Issues: Exploiting TrustZone TEEs"** — in-depth analysis and exploits against real TrustZone TEEs (QSEE/Kinibi), demonstrating how TEE OSes lag in mitigations and how real attack chains were constructed.  
  [Project Zero: Trust Issues: Exploiting TrustZone TEEs](https://projectzero.google/2017/07/trust-issues-exploiting-trustzone-tees.html)

- Academic/systematic reviews of TrustZone/TEE security show many practical attack vectors (TA revocation gaps, memory corruption, lack of modern mitigations).  
  [SoK: Understanding Security Vulnerabilities in TrustZone TEEs (paper)](https://syssec.dpss.inesc-id.pt/papers/cerdeira-sp20.pdf)

**What these mean:** TrustZone hardware is neutral or beneficial by design, but TEE firmware/Trusted Apps frequently contain exploitable problems because many implementations are closed and lack hardening.

---

## ⚔ Cross-Technology Comparison (high level)

| Property | Intel ME / CSME | AMD PSP / ASP | ARM TrustZone (TEE) |
|---|---:|---:|---:|
| Runs below OS? | Yes | Yes | Yes (when Secure World active) |
| Independent network access | Yes (AMT / out-of-band on vPro) | No (not by default) | No (TEE doesn't usually manage NIC) |
| Firmware open? | No (proprietary) | No (proprietary) | TEE OS often proprietary; hardware spec public |
| Typical exploit vector | Network + local | Local / privileged | Local / privileged / side-channel |
| Neutralization available? | Partial (me_cleaner, vendor efforts) | Not generally | Not applicable (hardware feature) |

---

## 🧰 Tools, Mitigations & Privacy-focused Vendors

### Intel ME
- **me_cleaner (GitHub)** — tool to strip/neutralize ME firmware regions. *Use with extreme caution; flashing modified firmware can brick devices.*  
  https://github.com/corna/me_cleaner

- [Remove_IntelME_FPT](https://github.com/mostav02/Remove_IntelME_FPT) - A guide for disabling Intel Management Engine using FPT on PCH SPI 

- **Intel security advisories / vendor firmware updates** — official mitigation is vendor firmware; check OEM download pages and Intel advisories listed above.  
  https://www.intel.com/content/www/us/en/security-center/default.html

- **Vendor examples (partial neutralization / open firmware):**  
  - **Purism** — explains ME neutralization on Librem laptops (historical practices vary by model).  
    https://puri.sm/learn/intel-me/  
  - **System76** — documents Open Firmware and ME state handling for supported models.  
    https://support.system76.com/articles/intel-me/

### AMD PSP
- **AMD product security bulletins** and OEM firmware updates are the primary mitigation path. There is **no official/popular neutralizer** equivalent to me_cleaner for PSP at time of writing.  
  https://www.amd.com/en/resources/product-security.html

- **PSPTool (GitHub)** — firmware analysis / extraction tool for researchers.  
  https://github.com/PSPReverse/PSPTool

### ARM TrustZone / TEEs
- Use **open, auditable TEE implementations** where possible (examples: OP-TEE) and apply vendor security updates.  
  https://www.op-tee.org/

---

<details><summary>Selected authoritative sources (examples, read these for technical depth)</summary>

- Intel Security Advisories (ME/CSME/AMT):  
  https://www.intel.com/content/www/us/en/security-center/default.html

- INTEL-SA-01315 advisory (Feb 2026):  
  https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-01315.html

- INTEL-SA-00783 advisory (2023 / updated 2024):  
  https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-00783.html

- Intel advisory example (INTEL-SA-00295; includes CVE-2020-0594):  
  https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-00295.html  
  NVD entry for **CVE-2020-0594**: https://nvd.nist.gov/vuln/detail/CVE-2020-0594

- me_cleaner (community tool):  
  https://github.com/corna/me_cleaner

- AMD Security Bulletins (PSP & embedded processors):  
  https://www.amd.com/en/resources/product-security.html  
  AMD-SB-5001: https://www.amd.com/en/resources/product-security/bulletin/amd-sb-5001.html  
  AMD-SB-5002: https://www.amd.com/en/resources/product-security/bulletin/amd-sb-5002.html

- NVD entries (examples):  
  CVE-2017-5712 (Intel AMT): https://nvd.nist.gov/vuln/detail/CVE-2017-5712  
  CVE-2022-23820: https://nvd.nist.gov/vuln/detail/CVE-2022-23820  
  CVE-2022-23821: https://nvd.nist.gov/vuln/detail/CVE-2022-23821

- Project Zero research on TrustZone TEEs:  
  https://projectzero.google/2017/07/trust-issues-exploiting-trustzone-tees.html

- Academic/systematic survey of TEE vulnerabilities:  
  https://syssec.dpss.inesc-id.pt/papers/cerdeira-sp20.pdf
</details>

--- 
 
## 🛠️ Vendors & Projects Offering Open Firmware / Reduced Proprietary Firmware Hardware

Many smaller vendors, open-firmware projects, and hardware suppliers offer machines that ship with or support **open firmware (coreboot / Libreboot / Dasharo)** — reducing reliance on proprietary BIOS/UEFI and allowing greater control over platform firmware. These vary from full laptops to embedded boards and network appliances.

> ⚠️ **Caution:** Support for open firmware and neutralization of proprietary subsystems (e.g., Intel ME) varies by model and vendor. Check specific product documentation before purchasing.

---

### 🧠 Community Open Firmware Projects

#### **Libreboot**
Libreboot is a fully libre distribution of coreboot designed to replace proprietary BIOS/UEFI firmware on select compatible systems. You can also purchase machines with Libreboot pre-installed from associated vendors.  
👉 https://libreboot.org **(Libreboot main site)** 

### 🛠 **Coreboot & Related Firmware Projects**

**coreboot** (formerly LinuxBIOS) is a free and open-source firmware platform that replaces traditional BIOS/UEFI. It is used by many community and vendor projects, including Libreboot, Dasharo, and more:

- **Dasharo** — a coreboot-based firmware distribution with emphasis on security, stability, and transparency, often used on laptops from community vendors. 
- **Heads** — a coreboot-compatible boot firmware project focusing on secure boot, measured boot, and tamper detection, often paired with a TPM to verify firmware integrity. 
- **MrChromebox Firmware** — community-maintained firmware images for Chromebooks that leverage coreboot/EDK2 to support non-ChromeOS operating systems.
- **Skulls** — user-friendly coreboot images for ThinkPad laptops. 

👉 https://www.coreboot.org **(coreboot official project page)**

These projects make it easier for users to **install and use open firmware** on supported systems, increasing transparency and reducing proprietary code in the boot process.

### 📌 **LinuxBoot – Firmware as a Linux Stack**

**LinuxBoot** is another free software firmware project that replaces key parts of UEFI firmware with a Linux kernel and userland tools:

- It runs on top of early firmware (PEI, coreboot, or U-Boot) and boots Linux directly.  
- LinuxBoot is already supported on some server platforms and open compute boards.  
👉 https://linuxboot.org **(LinuxBoot project)**

This approach treats the firmware stack more like a small Linux environment rather than a proprietary UEFI runtime, enabling deep customization.

### 📌 **Community Firmware Innovation on Older Platforms**

Some independent community projects are pushing open firmware even further on older hardware where firmware signature protections don’t block modification:

- The **15h.org project** is developing open-source firmware updates for older AMD Bulldozer/Piledriver platforms, enabling *fully open firmware operation* without modern signature restrictions.  
👉 Search “15h.org firmware support for AMD” (reported project supporting older boards)

These efforts help demonstrate what *fully open firmware ecosystems* can look like when hardware security restrictions are not enforced.

---

### 🖥️ Vendors & Distributions Shipping Open Firmware

The following vendors are either referenced directly by coreboot project documentation or widely acknowledged in open firmware communities as shipping systems with coreboot/Dasharo firmware:

#### **Nitrokey**
Nitrokey refurbishes laptops and sells devices with **coreboot + Dasharo firmware** and often includes open boot firmware with measured/verified boot options.  
👉 https://shop.nitrokey.com **(Nitrokey hardware & coreboot)**

#### **NovaCustom**
NovaCustom sells configurable laptops shipped with **Dasharo-based coreboot firmware**, maintained by 3mdeb, with support for Linux and Windows.  
👉 https://novacustom.com **(NovaCustom laptops)**

#### **Protectli**
Protectli offers Vault network appliances and small PCs with the option of **coreboot firmware**, or flashing via open tools, often jointly maintained with 3mdeb (Dasharo).  
👉 https://protectli.com/coreboot/ **(Protectli coreboot info)**

#### **Star Labs**
Star Labs sells Linux-focused laptops that are available with **coreboot firmware** and include options such as disabling the Intel Management Engine via NVRAM tools.  
👉 https://starlabs.systems **(Star Labs laptops)**

#### **PC Engines**
PC Engines produces embedded hardware (e.g., APU boards) that ship with **coreboot firmware** and are upstream supported through community channels.  
👉 https://pcengines.ch **(PC Engines)**

#### **Purism**
Purism sells Librem laptops designed for open source and privacy, with **coreboot firmware and ME neutralization** as part of their security strategy.  
👉 https://puri.sm/learn/intel-me/ **(Purism ME/firmware info)**

---

## 📌 Summary Table

| Vendor / Project | Firmware Type | Notes |
|------------------|---------------|-------|
| **Nitrokey** | coreboot / Dasharo | Refurbished open firmware hardware |
| **NovaCustom** | coreboot / Dasharo | Custom laptops with open firmware |
| **Protectli** | coreboot / Dasharo | Vault network appliances with coreboot |
| **Star Labs** | coreboot | Linux laptops with open firmware |
| **PC Engines** | coreboot | Embedded boards with open firmware |
| **Purism** | coreboot + ME neutralization | Privacy-focused open firmware laptops |
| **Libreboot Project** | Libreboot | Fully libre firmware distribution |

---

## 📄 Notes & Considerations

- Not all models from the same vendor are open firmware by default — some require **user flashing** of coreboot/Libreboot. 
- On many modern Intel platforms, **Intel Boot Guard** may prevent flashing alternative firmware unless hardware-specific de-guarding is used (a technical process).   
- Some vendors work with firmware integrators like **3mdeb** to provide improved firmware quality and security features (measured boot, verified boot). 

---

## 📌 Additional Hardware & Architecture Notes

### 💡 RISC-V and Non-x86 Architectures

For users seeking **hardware with minimal proprietary subsystems**, RISC-V is an emerging alternative:

- It is a **completely open instruction set architecture**, enabling hardware and firmware designs that are fully open from top to bottom.  
- Projects like **Raptor Computing Systems** offer POWER9-based hardware that eschews Intel/AMD proprietary backplanes in favor of open firmware designs and auditability.  
- RISC-V boards (e.g., SiFive) and ecosystem products are growing, providing future paths toward hardware transparency.

Community discussion on RISC-V and alternatives often highlights the lack of opaque controllers like ME/PSP on open architectures.

</details>
</details>

<hr>

<details>
<summary><b>OS Privacy / Security:</b></summary>
  
### Suggested OS:
<br> #1 [QubesOS](https://www.qubes-os.org/) - For heightened Security / Pirvacy and a compartmented OS on a single PC via the Xen hypervisor base. features some very interesting methods for security and is worth checking out if you value security / privacy as your top priority. 
<details>
 <summary><b>Additional Qubes resources:</b></summary>
  
### Qubes Related content: 
Additional Resources:
<br> [Qubes - Community Guides](https://forum.qubes-os.org/c/guides/14) - a List of Guides directly from the QubesOS Community.
<br> [Qubes network server](https://github.com/Rudd-O/qubes-network-server) - Turn your Qubes OS machine into a network server 
<br> [saltstack configs](https://github.com/unman/shaker) - This let's you declaratively manage your whole system.
<br> [Ansible plugins for QubesOS](https://github.com/QubesOS/qubes-ansible) - Ansible module and connection plugin for Qubes OS 
## Tutorials: 
[Qubes OS installation guide](https://doc.qubes-os.org/en/latest/user/downloading-installing-upgrading/installation-guide.html)
<br> [XDA - Qubes OS is the perfect operating system for security-conscious users](https://www.xda-developers.com/qubes-os-guide/)
<br> [QubesOS setup & configuration](https://github.com/0x4v0c4d0/qubes) - Instructions, scripts and files for installing and configuring QubesOS (including VPN and Crpto wallet suggestions.)
<br> Disposables:
<br> [How to use disposables](https://doc.qubes-os.org/en/latest/user/how-to-guides/how-to-use-disposables.html) - A disposable is a stateless qube, it does not save data for the next boot. These qubes can serve various uses cases that require a pristine environment

</details>
<br>
<br> Another option for VMs less security focused and headless meant for a server, but some better features: 
<br> [Proxmox](https://www.proxmox.com/en/) - For virtualization, self-hosting and all around general usage. it can even be used for gaming and other high performance tasks such as AI within VMs/Containers. 
<details>
 <summary><b>Additional Proxmox resources:</b></summary>
  
### Proxmox related content:
[Proxmox Forums](https://forum.proxmox.com/)
## Additional Resources:
[Proxmox VE Helper scripts](https://community-scripts.github.io/ProxmoxVE/) - Hundreds of scripts to quickly setup a wide range of projects on proxmox.
<br> [ProxMenux](https://github.com/MacRimi/ProxMenux) - Seperate Dashboard / CLI menu with more information & management options handy for proxmox management.

## VMs:
[OSX-KVM](https://github.com/kholia/OSX-KVM)

## Tutorials:
[XDA - A beginner's guide to setting up Proxmox](https://www.xda-developers.com/proxmox-guide/)
<br> [Proxmox Beginner Tutorial: How to set up your first virtual machine on a secondary hard disk.](https://forum.proxmox.com/threads/proxmox-beginner-tutorial-how-to-set-up-your-first-virtual-machine-on-a-secondary-hard-disk.59559/)
<br>
<br> GPU related: 
<br> [Simultaneous Intel GVT-G and Nvidia PCIe GPU Passthrough in Proxmox](https://kaanlabs.com/simultaneous-intel-gvt-g-and-nvidia-pcie-gpu-passthrough-in-proxmox/)
<br> nvidia:
<br> [Jellyfin LXC with Nvidia GPU transcoding and network storage](https://forum.proxmox.com/threads/jellyfin-lxc-with-nvidia-gpu-transcoding-and-network-storage.138873/)
<br> intel:
<br> [Enable Mediated Intel iGPU (GVT-g) for VM's in Proxmox (with Plex)](https://www.reddit.com/r/homelab/comments/jyudnn/enable_mediated_intel_igpu_gvtg_for_vms_in/)
<br> [[Guide] Jellyfin + remote network shares + HW transcoding with Intel's QSV + unprivileged LXC](https://forum.proxmox.com/threads/guide-jellyfin-remote-network-shares-hw-transcoding-with-intels-qsv-unprivileged-lxc.142639/)

## General info/helpful:
[XDA - I tried gaming on a VM hosted on a Proxmox server – here’s how it went](https://www.xda-developers.com/gaming-on-a-proxmox-vm/)
<br> [XDA - Running Proxmox VMs with GPU passthrough is much easier than it used to be](https://www.xda-developers.com/running-proxmox-vms-with-gpu-passthrough-is-much-easier/)
<br> [Windows Gaming VM on Proxmox: Performance Optimization in MSFS 2020](https://forum.level1techs.com/t/windows-gaming-vm-on-proxmox-performance-optimization-in-msfs-2020/187683)

</details>

<details>
  <summary><b>Other options:</b></summary>
  [OpenBSD](https://www.openbsd.org/) - minimal attack surface, but not great for end users, more suited to servers and use cases are pretty niche, so not a great choice for most users but worth a mention for the level of security provided. 
</details>

<details>
  <summary><b>Windows related:</b></summary>
  
<br> Windows Ansible Playbook - AME Playbooks: [AME Wizard](https://amelabs.net/) **NOTE: if you ever need to uninstall a playbook, you'll need to reinstall Windows.**
<br> [windows-playbooks](https://github.com/stravos97/windows-playbooks) - Windows setup and configuration via Ansible. 
<br> [AtlasOS](https://atlasos.net/) - [AtlasOS - Github](https://github.com/Atlas-OS/Atlas) - An open and lightweight modification to Windows, designed to optimize performance, privacy and usability. 
<br> [ReviOS Playbook](https://github.com/meetrevision/playbook) - A lightweight, stable, and performance-focused customized version of Windows that enhances privacy and compatibility 
<br> [Redress Server PlayBook - Windows server 2022](https://github.com/redress-server/playbook) - 
<br> [RapidOS](https://github.com/rapid-community/RapidOS) -  RapidOS is a powerful modification for Windows 10/11 that radically transforms the OS through deep customization, while maintaining rock-solid stability through the AME Beta playbook system. 
<br> [Rapid](https://github.com/Rapid-OS/Rapid) - A tailor-made modification of Windows designed for maximize Gaming performance and latencies.
<br>
<br> <b>Windows Group Policy:</b> 
<br> [Group Policy settings to improve privacy/security](https://discuss.privacyguides.net/t/group-policy-settings-to-improve-privacy-security/27339)
<br> [30 Critical Group Policy Settings to Secure & Optimize Windows](https://www.pctips.com/group-policy-settings/) - 
<br> [Privacy and security baseline for personal Windows 10 and Windows 11](https://github.com/troennes/private-secure-windows) - 
<br> [Windows On Reins](https://github.com/gordonbay/Windows-On-Reins) - Wor is a Powershell script to harden, debloat, optimize, enhance privacy, avoid fingerprinting and improve performance on Windows 10 and 11. 
<br> [ PC-Optimization-Hub](https://github.com/BoringBoredom/PC-Optimization-Hub) - collection of various resources devoted to performance and input lag optimization 

</details>


### Software: 

- [Awesome Privacy](https://github.com/pluja/awesome-privacy) - A curated collection of privacy-focused software and tools designed to enhance your online and offline privacy and security.

### Password managers: 
- Browser / Linux / MacOS/ Win: [KeePassXC](https://keepassxc.org/)
- Android: [KeePassDX](https://www.keepassdx.com/)

### Firewall/Networking:
- Linux: [OpenSnitch](https://github.com/evilsocket/opensnitch) - Network monitoring and rule-based control.  
- Windows: [SimpleWall](https://github.com/henrypp/simplewall) - Windows firewall management and rules.  
- Mac: [LittleSnitch](https://www.obdev.at/products/littlesnitch/index.html) - Network monitoring and control.  
- Win/Linux: [Safing Portmaster](https://safing.io/download/).  
- Windows: [NetLimiter](https://www.netlimiter.com/) - Per-application bandwidth limits.

Windows: 

### Windows Tweaks & Privacy:

- [Windows 10/11 Optimization & Customization Guide](https://github.com/just-maik/win-opti-resources) - A comprehensive guide that helps optimize and customize Windows 10/11 for improved performance, privacy, and overall system experience.
- [Microsoft Edge Removal](https://github.com/ShadowWhisperer/Remove-MS-Edge) - A tool to fully remove Microsoft Edge from Windows, ensuring it is completely deleted and no longer part of your system.
- [OneDrive-Uninstaller](https://github.com/ionuttbara/one-drive-uninstaller) - A tool for completely uninstalling OneDrive from your system, preventing it from syncing and taking up system resources.
- [Dism++](https://github.com/Chuyu-Team/Dism-Multi-language) - A powerful system tool for managing Windows image files, providing options for system cleanup, privacy tweaks, and optimizations.
- [Blackbird](https://www.getblackbird.net/) - A privacy-focused tool that disables tracking, telemetry, and other unwanted data collection by Windows, offering enhanced privacy on your system.
- [10AntiSpy](https://www.majorgeeks.com/files/details/10antispy.html) - A privacy-focused tool for disabling unwanted Windows 10 features such as telemetry, Cortana, and other data collection mechanisms.
- [UWT4](https://www.majorgeeks.com/files/details/ultimate_windows_tweaker_for_windows_10.html) - A comprehensive tweak tool for Windows 10 that allows you to customize various system settings and privacy options, improving performance and security.
- [Win10BloatRemover](https://github.com/Fs00/Win10BloatRemover) - A tool that helps you remove bloatware and unwanted apps from Windows 10, freeing up space and improving system performance.
- [Intelligent Standby List Cleaner](https://www.majorgeeks.com/files/details/intelligent_standby_list_cleaner.html) - A tool to reduce system stuttering and improve performance by clearing the standby memory list, optimizing RAM usage.
- [Destroy Windows 10 Spying](https://www.majorgeeks.com/files/details/destroy_windows_10_spying.html) - A privacy tool designed to disable telemetry, tracking, and data collection features in Windows 10 for improved privacy and control.

Older versions some people will still desire to use: 

### Windows 7:

- [Windows 7 ESU Patching](https://hackandpwn.com/windows-7-esu-patching/) - A guide for patching Windows 7 Extended Security Updates (ESU) to keep receiving updates and security patches after official support ends.
- [Windows 7 Service Pack 2](https://github.com/marie-systems/win7-sp2/tree/main) - A community-driven project to create an unofficial Windows 7 Service Pack 2, adding new features, bug fixes, and improvements to Windows 7.
- [Windows 7 SP4 Unofficial](https://github.com/LemonSqueezers/W7SP4) - An unofficial Service Pack 4 for Windows 7, designed to enhance the system with additional updates and fixes after the official support ended.
- [Windows-7 on Modern Hardware](https://github.com/wortkraecker/Windows-7-on-modern-hardware) - A guide for installing and running Windows 7 on modern hardware, with advice on drivers, patches, and compatibility tweaks.
- [Windows 7 Fan-Made Survival Guide](https://github.com/kuba2k2/i-use-win7-btw) - A fan-made survival guide for using Windows 7 in the modern era, offering tips, tweaks, and tools to keep Windows 7 functional and secure.

</details>

<hr>

## 🧩 Expanding the Perimeter: Beyond Hardware & OS

the next critical layers of your digital life: the **applications** you use daily and the **operational habits** that can either protect or betray you.

This section aims at providing clear comparisons, mitigation strategies, and links to sources. We will explore privacy-respecting browsers, mobile operating systems, secure email providers, encrypted cloud storage, and the crucial concept of operational security (OpSec).

<br>

* * *

## 🌐 1) Web Browsers: Your Window to the World

The browser is arguably the most important piece of software on any device. It's the primary interface to the internet and a massive attack surface for tracking, fingerprinting, and data collection. Moving away from mainstream, data-hungry browsers like Google Chrome and Microsoft Edge is a foundational privacy step.

Here are the leading privacy-focused alternatives, categorized by their philosophy and threat model.

### 🏆 Browser Comparison at a Glance

| Browser | Engine | Best For... | Key Privacy Feature | Trade-off |
| :--- | :--- | :--- | :--- | :--- |
| **LibreWolf** | Gecko (Firefox) | Advanced users wanting maximum hardening out-of-the-box. | Anti-fingerprinting; removes all Mozilla telemetry. | Can cause site breakage; project has no legal entity; developers enforce an explicit political stance, banning users perceived as "far-right." |
| **Mullvad Browser** | Gecko (Firefox) | Anonymity *without* the Tor Network; pairing with a VPN. | Designed to make all users look identical to resist tracking. | Not a Tor replacement; requires a trusted VPN for IP anonymity. |
| **Tor Browser** | Gecko (Firefox ESR) | High-risk anonymity needs (journalists, activists, dissidents). | Routes all traffic through the Tor network; makes users indistinguishable. | Can be slow; frequent site breakage due to high-security network. |
| **Brave** | Blink (Chromium) | Everyday users wanting strong, easy privacy with good usability. | Blocks ads/trackers by default; built-in fingerprinting protection. | Chromium-based; past controversies over affiliate link hijacking; founder's political donations; questionable marketing practices. |
| **Safari** | WebKit | Apple ecosystem users wanting a solid, integrated baseline. | Intelligent Tracking Prevention (ITP) is highly effective. | Tied to Apple ID and telemetry; limited extension ecosystem. |

<br>

<details>
    <summary><b>🦊 LibreWolf: The Hardened Firefox</b></summary>

LibreWolf is a fork of Firefox with a singular mission: to maximize privacy and security. It's not just Firefox with some add-ons; it's a fundamentally reconfigured browser that strips out telemetry, resists fingerprinting, and prioritizes user protection over convenience.

**What it is:**
LibreWolf is a community-driven, open-source browser based on Firefox. It acts as a drop-in replacement that feels familiar to Firefox users but operates with a dramatically different philosophy. It comes with a custom `user.js` file that contains hundreds of hardening preferences, applied before you even open the browser for the first time .

**Key privacy & security features:**
- **Telemetry Removal:** All communication with Mozilla servers (telemetry, studies, experiments) is disabled. Firefox Sync is also disabled by default, though it can be optionally enabled by the user.
- **Out-of-the-Box Hardening:** Unlike Firefox, which requires manual tweaking, LibreWolf ships with aggressive privacy settings:
    - **uBlock Origin** is included and pre-configured in advanced blocking mode.
    - **Strict anti-fingerprinting:** It resists techniques used to create a unique browser fingerprint.
    - **Forces HTTPS** on all connections where possible (HTTPS-Only Mode).
    - Deletes cookies and website data upon browser close by default.
- **Search Default:** The default search engine is DuckDuckGo, not Google. It also offers other privacy-respecting alternatives like MetaGer, SearXNG, and StartPage in its dropdown.
- **State-Level Partitioning:** This advanced feature isolates website data (like cookies and caches) to the domain they came from, making it much harder for trackers to follow you across different sites.

**Criticisms and trade-offs:**
- **Project Structure & Updates:** There is no legal entity behind the project, meaning there are no legal ramifications if something goes wrong. The binaries aren't signed, and there is **no auto-update mechanism**. On Windows, users must rely on a third-party updater, which introduces a middleman and potential security risks. This means LibreWolf is often behind on critical security updates compared to Firefox .
- **Outgoing Connections:** Despite its privacy focus, a network analysis reveals that the very first time LibreWolf is started, it contacts domains like Mozilla's add-on CDN and Amazon Cloudfront, even with automatic updates disabled. The project justifies this as necessary for updating blocking lists, but critics argue that **all update features should be opt-in** to prevent any potential privacy leak (e.g., revealing IP address during a VPN failure) .
- **Political Stance and Censorship:** The project lead, known only as "ohfp (she/her)," has explicitly declared that LibreWolf is "**very woke by design**" and that the browser is not merely a technical tool but a political platform . This stance has translated into active moderation policies:
    - Users and content associated with figures like Lunduke (of the Lunduke Journal) have been banned from project communication channels. When Lunduke himself entered the project's Matrix chat to ask a polite question, he was immediately told "If you are the real Lunduke, please leave" and subsequently banned .
    - The stated reason for such bans is that the project considers certain individuals or viewpoints "far-right," "anti-queer," or "racist." The lead developer has stated: "If I kick out a hundred racists and that makes a single person from a minority feel safer, it's worth it" .
    - Any attempt to discuss these bans or question the project's leadership reportedly results in immediate banning . A user who opened a polite issue on the project's Codeberg page asking the developers to "keep LibreWolf focused on its core values (no politics!)" received a sarcastic response: "Thanks for enlightening us and making us realize the error of our ways! wasting our time with this issue," before the discussion was locked .
     - They also within their rant about the "far right" proclaimed pro-censoship veiws, as long as it was against people they disagree with. which i personally also find concerning. 
- **Lack of Transparency:** The project is led by anonymous figures. The primary maintainer operates under a pseudonym with no real name or verifiable history disclosed. The project website lacks an "About Us" page, and the lead's GitHub profile is largely empty . For a browser handling sensitive user data, this anonymity, combined with the explicit political agenda, raises concerns about accountability and trust .
- **Site Breakage:** The strict privacy and fingerprinting protections can cause some websites to function incorrectly or break entirely. This is the price of high security.

**Installation and resources:**
- **Official Download:** [https://librewolf.net/](https://librewolf.net/)
- **Documentation:** [https://librewolf.net/docs/](https://librewolf.net/docs/)
- **Key Hardening Reference (for any Firefox-based browser):** The `user.js` project [https://github.com/pyllyukko/user.js](https://github.com/pyllyukko/user.js) provides a comprehensive configuration file for hardening Firefox, which forms the basis of LibreWolf's philosophy.

</details>

<br>

<details>
    <summary><b>🦁 Brave: The Privacy-Questionable Contender</b></summary>

Brave is a Chromium-based browser that promises privacy with built-in ad-blocking and content-blocking protection. It also offers several quality-of-life features and services, like a VPN and Tor access.

**What it is:**
Brave is a free and open-source web browser developed by Brave Software, Inc. based on the Chromium web browser. It blocks ads and website trackers by default, and allows users to opt into privacy-respecting ads that reward them with Basic Attention Tokens (BAT).

**Key privacy & security features:**
- **Built-in Ad & Tracker Blocking:** Blocks unwanted content by default, improving both privacy and page load speeds.
- **Brave Shields:** Provides granular control over fingerprinting protection, cookies, and HTTPS upgrades on a per-site basis.
- **Offline Private Window with Tor:** Allows users to browse .onion sites and use the Tor network directly from a private window (though this is not a replacement for the Tor Browser).
- **Crypto Wallet:** Includes a built-in cryptocurrency wallet for interacting with Web3 and managing BAT rewards.

**Criticisms and controversies:**
- **Affiliate Link Hijacking:** Brave was found to automatically redirect users to cryptocurrency exchange Binance using an affiliate referral code, meaning Brave profited from user traffic without clear disclosure. While this was removed, it damaged trust.
- **"Phoning Home" and Privacy:** Despite its privacy promises, network analysis shows that as soon as the browser is started, it begins contacting various domains, including Amazon services. Critics argue a truly private browser should not make any network connections without explicit user consent.
- **Crypto and Business Model:** The browser's heavy integration with cryptocurrency (BAT) and its CEO's focus on blockchain-based advertising raise concerns. Critics argue this creates a conflict of interest, where the company's revenue model still fundamentally relies on advertising. Ads are still present in the UI out-of-the-box (e.g., Brave Rewards ads).
- **Founder's Background:** Brendan Eich, the CEO and co-founder, faced significant controversy when he was appointed CEO of Mozilla in 2014. This stemmed from a 2008 donation he made to California's Proposition 8, which sought to ban same-sex marriage. While some argue his personal political beliefs are irrelevant to his professional work, others feel it reflects on the company's values.
- **Marketing and "Empty Promises":** Some critics label Brave's marketing as misleading, positioning it as a privacy savior while engaging in the same data-driven advertising ecosystem it claims to disrupt.

**Installation and resources:**
- **Official Download:** [https://brave.com/](https://brave.com/)

</details>

<br>

<details>
    <summary><b>🦡 Mullvad Browser: Anonymity-First, VPN-Ready</b></summary>

The Mullvad Browser is a unique collaboration between the Tor Project and Mullvad VPN. It's a hardened version of Firefox designed to minimize tracking and fingerprinting, with the explicit goal of being used with a trustworthy VPN. It brings the Tor Browser's philosophy of "all users look the same" to the wider web.

**What it is:**
The Mullvad Browser is a privacy-focused browser that bundles the Tor Browser's anti-fingerprinting patches and privacy enhancements. However, it is **not** connected to the Tor network by default. Instead, it's designed to be used with a VPN (like Mullvad's own service) to provide anonymity. If you use it without a VPN, your IP address is still exposed, but you benefit from its fingerprinting defenses .

**Key privacy & security features:**
- **Anti-Fingerprinting:** It inherits the Tor Browser's meticulous approach to making all users of the browser appear identical. This includes spoofing screen dimensions, limiting available fonts, and standardizing other browser attributes that are typically used for fingerprinting .
- **Letterboxing:** A technique that adds padding around a browser window to prevent websites from using the exact window size as a fingerprinting vector.
- **No Persistent State:** By default, it acts like a private browsing window. History, cookies, and site data are cleared when the browser is closed. This is intentional to prevent long-term tracking.
- **First-Party Isolation:** This feature ensures that trackers embedded on different websites cannot share information about you, as your identity is isolated to the main domain you are visiting.
- **Integration with Mullvad VPN:** It can be configured to work seamlessly with the Mullvad VPN client, adding another layer of IP protection on top of its fingerprinting defenses.

**Trade-offs and considerations:**
- **Not a Tor Replacement:** It does not route traffic through the Tor network. For anonymity against network-level adversaries, you **must** pair it with a trusted VPN (or use the Tor Browser instead). Its strength is in application-level uniformity, not network-level anonymity .
- **Usability Friction:** Like the Tor Browser, some features that compromise privacy (like WebGL or certain JavaScript APIs) are limited or disabled, which can affect website functionality.
- **VPN Dependency for Complete Anonymity:** To achieve full anonymity, you need a VPN that doesn't log your activity. Mullvad VPN is a strong choice, having passed multiple independent audits confirming its no-logs policy .

**Installation and resources:**
- **Official Download:** [https://www.mullvad.net/en/download/browser/](https://www.mullvad.net/en/download/browser/)
- **Mullvad VPN Security Audits:** [https://mullvad.net/en/blog/tag/audits/](https://mullvad.net/en/blog/tag/audits/) (See audit reports confirming no-log policy) .

</details>

<br>

<details>
    <summary><b>🧅 Tor Browser: The Gold Standard for Anonymity</b></summary>

The Tor Browser is the culmination of years of work by the Tor Project. It is the only browser that provides both application-level fingerprinting protection and network-level anonymity by routing traffic through the Tor network. It is designed for users with the highest threat models.

**What it is:**
The Tor Browser is a modified version of Firefox ESR (Extended Support Release). It includes numerous patches to enhance privacy and integrates the Tor network for routing all traffic. It is the primary tool for people who need to protect their identity from state-level surveillance, such as journalists, activists, and whistleblowers .

**Key privacy & security features:**
- **Network Anonymity:** Your traffic is bounced through a series of three volunteer-operated relays (nodes), encrypting it multiple times. This makes it nearly impossible for anyone to trace the traffic back to your IP address. The Tor Project has a strict no-logging policy .
- **Unified User Base:** By making all users look the same (same screen size, same user agent, limited fonts), it creates a large anonymity set. "You are just one of many," making it extremely difficult to single out an individual user based on their browser fingerprint .
- **Built-in Defenses:** It includes HTTPS-Only mode, disables risky plugins and JavaScript where necessary, and bundles NoScript for advanced script control.
- **New Identity Feature:** A single click allows you to discard all current browsing data (cookies, history) and open a new browser window with a fresh Tor circuit, effectively giving you a new, clean identity.

**Trade-offs and considerations:**
- **Performance:** Routing through multiple relays inherently introduces significant latency. Browsing can be very slow, especially for media-rich websites.
- **Website Functionality:** Many modern websites rely on technologies or scripts that Tor Browser blocks or limits. Captchas are notoriously frequent and difficult on Tor. Some sites may refuse to function at all .
- **Entry Nodes:** In censored environments, Tor Browser may use "pluggable transports" to disguise its traffic. The initial connection to fetch bridge information requires a brief non-Tor connection, but the Tor Project does not log this .
- **Not for Everyday Browsing:** For most users' daily needs, it is too slow and inconvenient. It is a specialized tool for a specific, high-risk purpose.

**Installation and resources:**
- **Official Download & Project:** [https://www.torproject.org/](https://www.torproject.org/)
- **Tor Browser Privacy Policy:** [https://tor.eff.org/tk/about/privacy_policy/](https://tor.eff.org/tk/about/privacy_policy/) (Details its zero-data-collection approach) .

</details>

<br>

<details>
    <summary><b>🛡️ Brave & Other Notable Mentions</b></summary>

While the above three represent the pinnacle of specific privacy philosophies, other browsers offer compelling features for different use cases.

### Brave: The All-Rounder
Brave is a Chromium-based browser that has built a strong reputation for putting privacy first by default. It blocks ads, trackers, and fingerprinting out of the box. It also offers unique features like a built-in Tor window for private tabs (though this only protects that tab, not the whole browser).
- **Pros:** Excellent usability, fast, strong defaults, available on all platforms, built-in crypto wallet (if you use it) .
- **Cons:** It is still based on the Chromium engine, which is largely controlled by Google. Its business model, which includes optional privacy-respecting ads and a crypto token, can be a point of distrust for some purists .

### Safari: The Walled Garden Defender
For users deep in the Apple ecosystem, Safari is a surprisingly strong contender. Apple's Intelligent Tracking Prevention (ITP) is highly sophisticated and effectively limits cross-site tracking.
- **Pros:** Excellent fingerprinting resistance, deeply integrated with the OS for performance and battery life, strong on-by-default protections.
- **Cons:** It is tied to your Apple ID, creating a different kind of privacy trade-off. Extensions are less powerful than on desktop. Its engine (WebKit) is proprietary to Apple .

### DuckDuckGo Browser: The Simple Choice
DuckDuckGo offers a browser for mobile (and recently desktop) that focuses on simplicity and strong tracker blocking.
- **Pros:** Extremely easy to use, "Fire Button" to clear all data instantly, forces HTTPS, provides simple privacy grade ratings for sites.
- **Cons:** Lacks the deep, advanced fingerprinting defenses of LibreWolf or Mullvad Browser. It's an excellent choice for a "casual privacy upgrade" but not for high-risk scenarios .

</details>

<br>

<details>
  <summary><b>Other Browsers and Considerations:</b></summary>
Beyond the browsers detailed above, the landscape is filled with options that are often recommended for privacy but fail to deliver upon closer inspection. Based on a comprehensive analysis from Unix Digest, here is a breakdown of how various browsers measure up regarding privacy, transparency, and user control.

#### Privacy-Compromising Browsers (Despite Recommendations)

**Mozilla Firefox**
While highly tweakable, Firefox is **not private out-of-the-box**. It "phones home" to multiple Mozilla domains (e.g., `detectportal.firefox.com`, `location.services.mozilla.com`) every time it starts, even with telemetry disabled. Data collection is **opt-out**, meaning the browser connects to Mozilla before a user can disable it.
*   **Major Concerns:**
    *   **Cloudflare DoH Integration:** Mozilla made Cloudflare the default DNS-over-HTTPS provider. Cloudflare, a US company, logs DNS requests for 24 hours and retains "anonymized" samples indefinitely. This subjects user data to US law and introduces a significant trust intermediary.
    *   **Funding Conflict:** Google is Mozilla's default search engine because they pay for the position, creating a fundamental conflict of interest with Mozilla's stated privacy principles.
    *   **Telemetry Practices:** Past incidents like the silent inclusion of Cliqz in Germany and the "Telemetry Coverage" system to check if telemetry was disabled have eroded trust.

**Google Chrome & Chromium**
These browsers are designed for data collection. Every startup contacts Google, and almost every keystroke in the address bar is logged. Google openly states it collects information to build advertising profiles, even for users not signed into a Google Account. Chrome contains closed-source elements, making independent verification of its data handling impossible.

**Pale Moon**
Sometimes recommended for privacy, but it is not promoted as such by its developers. Network analysis shows it connects to Google on startup, similar to Chromium.

**Waterfox**
Despite being a popular Firefox fork, it connects to the Mozilla add-on CDN and Amazon Cloudfront on startup. Its privacy policy includes troubling clauses, such as passing user information to a successor if acquired, and states it uses cookies and third-party web analytics.

**GNOME Web (Epiphany) & Eolie**
These browsers aim for good privacy but fail by design. On first startup, they contact `easylist-downloads.adblockplus.org` to update ad filters. This sends data to eyeo GmbH (the company behind AdBlock Plus), which has its own privacy policy for collecting personal information.

**Midori**
Now part of the Astian Foundation, its future direction is uncertain. Its privacy policy is vague and disclaims responsibility. Enabling its ad-blocking extension triggers requests to Amazon Cloudfront, which logs user activity.

#### Privacy-Respecting Browsers

These browsers adhere more closely to the principle that a browser should not make network connections without explicit user action.

**Tweaked Firefox (The Best Solution)**
The article concludes that using a standard, up-to-date Firefox installation hardened with the **Arkenfox user.js** is the superior approach. This provides the timely security updates of mainstream Firefox while achieving privacy that meets or exceeds specialized forks. Crucially, it allows users to disable **all** automatic outgoing connections, giving them full control. This is recommended over LibreWolf because it avoids update delays and allows for a truly opt-in connection policy.

**Falkon**
A KDE browser using the Qt WebEngine (Chromium backend). It is lightweight, fast, and respects privacy by not making unsolicited connections. It includes built-in ad blocking and Greasemonkey support. Users should review all settings, as not all privacy features are enabled by default.

**qutebrowser**
A browser with Vim-style key bindings and a minimal GUI. It uses Qt WebEngine and is highly configurable. It does nothing without user consent, though features like DNS prefetching are enabled by default and must be manually disabled. Ad blocking is basic (hosts file based), but it can be configured to open videos in an external player.

**ungoogled-chromium**
A set of patches for Chromium that removes all background requests to Google services. It is a true drop-in replacement for users who need Chromium compatibility without Google integration.
*   **Reservations:** The small team can lag behind Chromium's security updates. Downloading pre-built binaries carries a risk of tampering; compiling from source or using a distribution's package manager is strongly recommended.

**Tor Browser**
The gold standard for anonymity when used correctly. It routes traffic through the Tor network, makes users indistinguishable from one another, and clears all state on exit. It is not for speed or convenience, but for high-risk privacy needs. Even without the Tor network, its hardened configuration makes it a truly privacy-respecting browser.
</details>

<br>

<br> <b>Note:</b> My personal views, are that Google Chromium is apart of an AI surveillance framework and is something worrying by design. it has capabilities to monitor everything, hijack site requests / dns traffic to redirect queries, place overlays over site logins and other aspects of websites, screen and rewrite pages, change results, access local user files, spy on other applications, spy on your microphone / camera, infect your devices, etc. 

bases such as electron place chromium between you and your most privacy dependant applications. chat software, password managers, note apps, clipboard managers, browsers, even VPN softwares. 

<details>
  <summary><b>Electron/Chromiums role in your software:</b></summary>
  The Chromium browser engine is the foundation for a vast number of applications, primarily through frameworks like <b>Electron</b>. This framework allows developers to build desktop applications using web technologies (HTML, CSS, JavaScript), which are then run within an embedded Chromium instance .

Here is a list of notable applications built with Electron, organized by category.

### 💬 Communication & Collaboration
This category includes some of the most widely used team messaging and video conferencing tools.
*   **Discord**: A popular voice, video, and text chat app for communities and gamers .
*   **Slack**: A team communication platform for workplace messaging and collaboration .
*   **Microsoft Teams**: Microsoft's hub for teamwork, integrating chat, meetings, and files .
*   **Signal**: A private messaging app focused on security and end-to-end encryption .
*   **WhatsApp**: The desktop companion app for the ubiquitous mobile messaging service .
*   **Twitch**: The primary desktop app for streaming and watching live gaming and creative content .
*   **Yammer**: An enterprise social networking service used for internal communication within organizations .

### 💻 Developer Tools
Many essential tools for programmers and developers are built on Electron.
*   **Visual Studio Code**: A immensely popular, free, and open-source code editor from Microsoft .
*   **GitHub Desktop**: A GUI application for managing Git repositories and GitHub workflows .
*   **Postman**: An API platform for building, testing, and documenting APIs .
*   **Atom**: A "hackable" text editor developed by GitHub (now discontinued) that originally popularized Electron .
*   **GitKraken**: A graphical Git client with an intuitive interface .
*   **MongoDB Compass**: A GUI for managing and querying MongoDB databases .
*   **Altair GraphQL Client**: A feature-rich GraphQL client for debugging and testing .
*   **Docker Desktop**: The main application for managing Docker containers and images on a local machine .
*   **Eclipse Theia**: An extensible platform for building cloud and desktop IDEs, similar to VS Code .

### 📝 Productivity & Knowledge Management
This group covers tools for note-taking, project management, and personal organization.
*   **Notion**: An all-in-one workspace for notes, tasks, wikis, and databases .
*   **Obsidian**: A powerful knowledge base that works on local Markdown files .
*   **Trello**: A web-based, list-making project management application .
*   **Asana**: A work management platform designed to help teams organize and track their work .
*   **Dropbox**: The desktop client for the popular cloud file storage and synchronization service .
*   **WordPress.com**: The desktop app for managing WordPress sites .
*   **Basecamp 3**: A project management and team communication tool .
*   **Joplin**: An open-source note-taking and to-do application with synchronization capabilities .
*   **LosslessCut**: A tool for lossless trimming and cutting of video and audio files .

### 🔒 Security & Utilities
Essential applications for passwords, file management, and system utilities.
*   **1Password**: A leading password manager for securely storing and using passwords .
*   **Bitwarden**: An open-source password manager with a free tier .
*   **Keybase**: An app for secure messaging and file sharing that combines cryptography with social proof .
*   **Mullvad**: The desktop client for the Mullvad privacy-focused VPN service .
*   **Samsung Magician**: The official software for managing Samsung solid-state drives (SSDs) .
*   **Logitech Options+**: The configuration software for Logitech mice and keyboards .
*   **OpenVPN Connect**: The official client for connecting to OpenVPN servers .

### 🎨 Creativity & Design
Tools for designers and creative professionals.
*   **Figma**: A collaborative interface design tool that runs in the browser and as a desktop app .
*   **TIDAL**: The desktop application for the high-fidelity music streaming service .
*   **Splice**: A platform for music producers offering samples, loops, and production tools .
*   **Boxy SVG**: A vector graphics editor for creating and editing SVG files .
*   **Blockbench**: A 3D model editor, particularly popular for creating models for Minecraft .

This list represents only a fraction of the applications built with this technology. For a more comprehensive and up-to-date collection, you can explore the official **[Electron Apps Showcase](https://www.electronjs.org/apps)** .

Absolutely. Here is an expanded section for **VPN software** that uses the **Electron** framework, including many of the most popular services.

---

### 🔒Expanded list of VPN Services (Desktop Clients)

Many major VPN providers have adopted Electron to build their desktop applications.

*   **NordVPN**: One of the most widely used VPN services, NordVPN's desktop client is built with Electron. The application provides an interactive map interface for server selection, specialized servers for specific tasks (like P2P or Onion over VPN), and comprehensive settings for features like Threat Protection, Meshnet, and split tunneling.
*   **ExpressVPN**: Industry-leading VPN known for its speed and reliability. While their underlying connection technology is proprietary, the desktop interface across Windows and macOS is built using web technologies, consistent with an Electron-based approach to achieve cross-platform parity and a polished UI .
*   **Proton VPN**: The official desktop clients for Windows and macOS are built with Electron. (Note: The Linux client is a separate, native Python/OpenVPN CLI tool, though community projects have explored Electron-based interfaces for Linux ). The Electron app provides a graphical interface for connecting to Proton's secure Core servers, managing VPN configurations, and accessing features like NetShield Ad-blocker and Secure Core.
*   **Surfshark**: A feature-packed VPN that has gained popularity rapidly. Its desktop application is developed using Electron, enabling a consistent experience across platforms. The interface includes a quick-connect button, server location lists, and access to features like CleanWeb, GPS spoofing, and the static IP server list.
*   **CyberGhost**: A user-friendly VPN with a large server network. The desktop client is built on Electron, featuring a smart rules dashboard that allows users to automate connection actions (e.g., connect automatically on untrusted Wi-Fi) and easy access to streaming-optimized servers.
*   **IPVanish**: A VPN service that emphasizes speed and configuration flexibility. Their desktop software utilizes Electron to provide a clean interface with real-time connection statistics, server latency graphs, and easy switching between VPN protocols (WireGuard, OpenVPN, IKEv2).
*   **Private Internet Access (PIA)**: Known for its transparency and customizable encryption settings. The PIA desktop client is built with Electron, offering both a standard view for casual users and an "expert" view that exposes detailed settings like port forwarding, DNS configuration, and proxy controls.
*   **Atlas VPN**: A freemium VPN service (now part of Nord Security). Their desktop application is developed using Electron, focusing on a streamlined and modern interface with features like a data breach monitor, SafeBrowse, and streaming-optimized servers.
*   **TunnelBear**: A whimsically designed VPN focused on simplicity and ease of use. The desktop client, built with Electron, features a distinctive bear-themed interface with a simple on/off switch and a world map for selecting connection countries.
*   **Windscribe**: A VPN with a generous free tier and a focus on ad and tracker blocking. Their desktop application is Electron-based, providing a straightforward interface for server selection and managing the built-in ad blocker, firewall, and proxy settings.

</details>

* * *

## 📱 2) Mobile Operating Systems

On mobile, the operating system is the ultimate arbiter of privacy. Stock Android (from Google) and iOS (from Apple) both have significant privacy trade-offs, primarily due to their integration with their parent companies' services. For those seeking true privacy and control, alternative Android-based operating systems are the only viable path.

<details>
    <summary><b>🤖 GrapheneOS: The Hardened Android</b></summary>

GrapheneOS is widely considered the most secure and privacy-focused mobile operating system available. It is a non-profit, open-source project that builds on the Android Open Source Project (AOSP) with extensive security hardening and privacy enhancements. It is only available for Google Pixel devices, as these have hardware security support (like Titan M2 chips and, on newer models, Memory Tagging Extensions (MTE)) .

**What it is:**
GrapheneOS is a hardened variant of Android. It doesn't just de-Google the OS; it fundamentally strengthens the Android security model. It focuses on reducing the attack surface and mitigating entire classes of vulnerabilities.

**Key privacy & security features:**
- **Hardened Memory Allocator:** It replaces the standard memory allocator with a hardened one that is much more resistant to memory corruption exploits, a common attack vector.
- **Hardened Kernel:** The kernel is patched with additional security features from the Linux kernel community and GrapheneOS's own projects.
- **Network & Sensor Permissions:** It adds highly granular permissions, allowing you to deny an app internet access or access to sensors (like the camera, microphone, or motion sensors) individually. You can give an app camera access, for example, but deny it internet access, preventing it from exfiltrating photos.
- **Sandboxed Google Play:** Instead of integrating Google Play Services deep into the OS (which creates a massive privacy and security hole), GrapheneOS allows you to install it as a regular, sandboxed user application. You get the functionality (like push notifications) without the special, privileged access .
- **Secure Camera & PDF Viewer:** Includes rebuilt apps that minimize metadata (like removing EXIF data from photos) and reduce attack surface.
- **Full Verified Boot:** Uses customAVB (Android Verified Boot) keys to ensure the OS hasn't been tampered with, and the bootloader remains locked for full security.
- **Memory Tagging Extensions (MTE):** On Pixel 8 and later, GrapheneOS fully leverages MTE in hardware to detect and prevent memory safety exploits across the system and in all installed apps, a groundbreaking security feature .

**Trade-offs and considerations:**
- **Device Limitation:** It only supports Google Pixel devices. This is a deliberate choice to leverage the best hardware security .
- **Usability for Some:** While highly usable, the lack of Google Play Services integrated by default means some apps that rely on them for core functionality (like some banking or ride-sharing apps) may not work, or may require the sandboxed Play Services installation. This is a trade-off for privacy.
- **Connectivity Checks:** By default, it checks for connectivity with GrapheneOS servers, not Google. This hides your IP from Google but makes your OS visible to a network observer. This can be changed to "Standard (Google)" in settings if you use a VPN to blend in with other Android devices .

**Installation and resources:**
- **Official Site & Installer:** [https://grapheneos.org/](https://grapheneos.org/)
- **Features Overview:** [https://grapheneos.org/features](https://grapheneos.org/features)

**note:** The fact GrapheneOS is only available on Google Pixel is concering. But that will soon end. Motorola (owned by Lenovo) is going to make a new GrapheneOS device that will be the first to ship with the OS pre-installed. 
 - [GrapheneOS Pixel exclusivity just officially ended](https://www.androidauthority.com/grapheneos-motorola-partnership-announced-3645710/)
 - [Motorola Plans GrapheneOS-Compatible Devices as Early as 2027](https://www.techrepublic.com/article/news-motorola-grapheneos-partnership-2027/) 
</details>

<br>

<details>
    <summary><b>🌱 CalyxOS: Privacy and Usability in Balance</b></summary>

CalyxOS is another excellent de-Googled Android operating system. It shares many goals with GrapheneOS but takes a slightly different approach, focusing on a balance between strong privacy and out-of-the-box usability on a wider range of devices.

**What it is:**
CalyxOS is an AOSP-based operating system that removes Google services and replaces them with free and open-source alternatives. It runs on Google Pixel devices, as well as some Fairphone and Motorola models, broadening its accessibility .

**Key privacy & security features:**
- **microG Integration:** Unlike GrapheneOS's sandboxed approach, CalyxOS offers the option to include **microG**, a free and open-source re-implementation of Google Play Services. This allows many apps that depend on Google's libraries (like push notifications and maps) to work seamlessly without connecting to Google servers .
- **Built-in Firewall:** CalyxOS includes a powerful, easy-to-use firewall that lets you control which apps have access to the internet, either on Wi-Fi or mobile data, on a per-app basis .
- **Calyx VPN:** It provides a built-in, free VPN service operated by the Calyx Institute, offering an easy way to mask your IP address.
- **Encrypted Backups:** An included backup app allows for encrypted, seed-word-protected backups that you can store remotely, a feature often missing from stock Android.
- **Sensitive Number Hiding:** A thoughtful privacy feature that hides calls to sensitive numbers (like abuse hotlines) from your call log.

**Trade-offs and considerations:**
- **Security Model:** CalyxOS is very secure, but its security hardening is generally considered less aggressive than GrapheneOS's. It makes compromises for wider compatibility and usability. For instance, its reliance on microG, while convenient, introduces a larger and less auditable codebase than GrapheneOS's sandboxed approach .
- **Update Cadence:** While it receives timely updates, GrapheneOS is typically faster and more consistent with security patches due to its narrower device focus.
- **Repository Management:** Users may occasionally encounter issues with app updates due to the priority of different F-Droid repositories, requiring manual adjustment .

**Installation and resources:**
- **Official Site & Installer:** [https://calyxos.org/](https://calyxos.org/)
- **Features Overview:** [https://calyxos.org/features/](https://calyxos.org/features/)

</details>

<br>

* * *

## 📧 3) Encrypted Email: Beyond the Consumer Giants

Email is a notoriously difficult protocol to secure. Its very nature involves sending messages to servers that may be outside your control. However, using a provider that implements end-to-end encryption (E2EE) and zero-access architecture is a massive step up from Gmail, Outlook, or Yahoo.

<details>
    <summary><b>🔒 Tuta vs. Proton Mail: A Detailed Comparison</b></summary>

Tuta (formerly Tutanota) and Proton Mail are the two leaders in the consumer encrypted email space. Both are based in countries with strong privacy laws (Germany and Switzerland, respectively) and offer robust security. However, their approaches differ, making them suited for different threat models .

### Encryption & Metadata
- **Tuta:** Takes a more aggressive approach to metadata protection. It encrypts the **entire email**, including the subject line, body, and attachments. It also does not collect IP addresses by default. It uses a proprietary, hybrid encryption method (AES and RSA) .
- **Proton Mail:** Encrypts the email body and attachments using the OpenPGP standard. However, it does **not** encrypt the subject line. It minimizes IP logging but may retain some basic metadata for delivery. Its use of a well-known standard (PGP) offers greater interoperability .

### Authentication & Access
- **Tuta:** All encryption and authentication happen within its own apps (web, mobile, desktop). There is no bridge for third-party email clients. This is simpler and reduces potential security risks but locks you into their ecosystem.
- **Proton Mail:** Offers **Proton Mail Bridge** for desktop users. This application allows you to use Proton Mail with any standard email client that supports IMAP/SMTP (like Thunderbird, Outlook, or Apple Mail), providing flexibility for users who need to integrate with existing workflows .

### Ecosystem & Features
- **Tuta:** Focuses on a tightly integrated, secure suite including email, calendar, and contacts, all fully encrypted.
- **Proton Mail:** Has expanded into a full privacy ecosystem with **Proton VPN, Proton Drive, Proton Pass, and Proton Calendar**. If you need an all-in-one privacy suite, Proton is the clear choice .

### Choosing the Right Provider
| Feature / Consideration | Choose **Tuta** if... | Choose **Proton Mail** if... |
| :--- | :--- | :--- |
| **Primary Focus** | Maximum metadata privacy (encrypted subject lines, no IP logging). | A balance of privacy and productivity features. |
| **Technical Need** | You want a simple, self-contained, highly secure system. | You need PGP compatibility or want to use a desktop email client via the Bridge. |
| **Ecosystem** | You primarily need secure email and calendar. | You want an integrated suite of privacy tools (VPN, Drive, Pass). |
| **Threat Model** | Journalists, activists, or anyone where metadata privacy is paramount.  | Businesses, teams, or individuals who need privacy but also workflow flexibility.  |

**Resources:**
- **Tuta:** [https://tuta.com/](https://tuta.com/)
- **Proton Mail:** [https://proton.me/mail](https://proton.me/mail)

</details>

<details>
  <summary><b>Proton Mail and Tuta (formerly Tutanota): Controversies Regarding User Data Disclosure:</b></summary>

This section examines significant controversies surrounding two major encrypted email providers—**Proton Mail** and **Tuta**—specifically concerning their handling of user data in response to law enforcement requests. Both services market themselves on strong privacy and security, yet have faced scrutiny over instances where user information was disclosed to authorities.

---

#### Proton Mail: Data Disclosure and the "No-Log" Promise

Proton Mail, the Swiss-based provider known for its end-to-end encrypted email and VPN services, has faced repeated controversy over its disclosures of user data to law enforcement. These incidents have drawn attention to the gap between certain marketing claims and the legal realities of operating under Swiss jurisdiction.

**The French Activist Case (2021)**

The most prominent incident occurred in 2021. Proton Mail provided Swiss authorities with the IP address and device details of a French climate activist who was part of a group called "Youth for Climate." The activist was later arrested [Read more on Restore Privacy](https://restoreprivacy.com/protonmail-logged-ip-address-of-french-activist-handed-to-authorities/). This case sparked significant backlash because Proton Mail had long advertised that it did not log IP addresses. A cached version of their website from August 2021 stated: *"No personal information is required to create your secure email account. By default, we do not keep any IP logs which can be linked to your anonymous email account. Your privacy comes first"* [Archive.org cache](https://web.archive.org/web/20210813221442/https://protonmail.com/privacy-policy).

Following the public outcry, Proton Mail **quietly removed all claims about not logging IP addresses** from its website. The company explained that under Swiss law, they were compelled to comply with a legally binding order from the Swiss Federal Office of Justice, which had been requested by French authorities via Europol. They maintained that they do not log IPs by default, but can be forced to do so on a case-by-case basis for specific users under investigation [Proton Mail Blog](https://proton.me/blog/proton-mail-disclosure).

**The "Stop Cop City" Case (2024)**

Another significant disclosure came to light in early 2024. Proton Mail provided Swiss authorities with payment data linked to the account `defendtheatlantaforest@protonmail.com`, which was associated with protests against the "Stop Cop City" movement in Atlanta. The FBI obtained this information through a Mutual Legal Assistance Treaty (MLAT) request on January 25, 2024. The data shared included credit card identifiers, which allowed authorities to identify the activist behind the anonymous account [Read more on The Intercept](https://theintercept.com/2024/02/27/proton-mail-stop-cop-city-surveillance/).

**The Catalan Activist Case**

In a separate incident, Proton Mail handed over a user's **recovery email address** to Spanish police investigating individuals suspected of supporting Catalan separatists. Spanish authorities then passed this recovery address to Apple, which was able to identify the individual associated with the account. Proton confirmed they were obligated to comply with Swiss laws concerning terrorism [Read more on Techdirt](https://www.techdirt.com/2021/09/07/protonmail-logs-ip-address-that-led-to-arrest-of-activist/).

**Transparency Data and Official Position**

Proton's own transparency report shows thousands of legal orders complied with annually. In 2023, for example, Proton Mail complied with 5,971 orders out of 6,378 received [Proton Transparency Report](https://proton.me/legal/transparency-report).

The company's current position, articulated by CEO Andy Yen, is that Proton provides "privacy by default and not anonymity by default." They argue that true anonymity requires users to take specific operational security measures, such as not using a recovery email or paying by credit card [Read more on Slashdot](https://news.slashdot.org/story/21/09/06/2055255/protonmail-ceo-says-privacy-default-not-anonymity-default-after-arrest-of-activist). The company also emphasizes that they are legally unable to comply with direct foreign requests and only respond to orders channeled through the Swiss legal system. Furthermore, they stress that **the content of encrypted emails remains inaccessible** to them.

---

#### Tuta (formerly Tutanota): The "Storefront" Allegation

Tuta, a German-based encrypted email provider, faced a different but equally serious controversy: an allegation that it operated as a "storefront" or "honeypot" for an intelligence agency.

**The Cameron Ortis Allegation (2023)**

In November 2023, during the trial of Cameron Jay Ortis, a former Royal Canadian Mounted Police (RCMP) intelligence official, a startling claim emerged. Ortis, who was accused of leaking secrets, testified that a foreign ally had informed him of a plan to encourage investigative targets to use an online encryption service—**Tutanota** (now Tuta)—which he alleged was a "storefront" operation created by intelligence agents to spy on adversaries [Read more on The Register](https://www.theregister.com/2023/11/21/rcmp_tutanota_storefront_claim/).

Ortis claimed he was told that a "storefront" had been created to attract criminal targets to use Tutanota, allowing the agency behind it to collect intelligence. He stated that he began enticing targets through promises of secret information with the aim of getting them to communicate via the service [Read more on CBC News](https://www.cbc.ca/news/politics/rcmp-ortis-trial-tutanota-1.7033591).

**Tuta's Strong Denial and Response**

Tuta responded swiftly and forcefully, calling the claim **"completely false"** and denying any ties to any intelligence or law enforcement agency [Tuta Blog: "We are not a honeypot"](https://tuta.com/blog/posts/we-are-not-a-honeypot).

In a detailed blog post titled "We are not a honeypot," the company outlined several key points:

-   **No Government Ties:** Tutao GmbH, the company behind Tuta, is fully and solely owned by its founders, Arne Möhle and Matthias Pfau, who started the company in 2011. It is not liable to any external entity or government.
-   **Legal Jurisdiction:** As a German company, Tuta only responds to valid court orders from German authorities. They do not and cannot comply with direct requests from foreign agencies.
-   **Open Source Code:** The entire client-side code is published on [GitHub](https://github.com/tutao/tutanota), allowing anyone to verify that there are no hidden backdoors and that end-to-end encryption works as advertised.
-   **Commitment to Mission:** The founders stated that operating as a front for intelligence agencies would completely contradict their mission as a privacy protection organization.

The company characterized Ortis's statements as an unsubstantiated lie, made without any supporting evidence. They noted that during cross-examination, Ortis responded to questions with phrases like "I can't say" or "I don't remember" when pressed for specifics [Read more on The Register](https://www.theregister.com/2023/11/21/rcmp_tutanota_storefront_claim/). The allegation was widely seen as a baseless claim made by a defendant in a criminal trial, and Tuta's denial was accepted by the privacy community.

---

#### Key Takeaways

-   **Proton Mail has repeatedly disclosed user metadata (IP addresses, recovery emails, payment data) to authorities when legally compelled under Swiss law.** This led to the quiet removal of their "no IP logging" marketing claims.
-   **The content of Proton Mail messages remains encrypted and inaccessible** to the company.
-   **Tuta faced a serious allegation of being an intelligence "storefront,"** which it vehemently and credibly denied, backed by its open-source code and German legal structure.
-   **The "Zero-Knowledge" Distinction:** Both companies maintain "zero-knowledge" architectures for message content, meaning they cannot decrypt and read user emails. However, both collect and can be forced to disclose **metadata** (like IP addresses, recovery emails, and payment information) that users provide or generate when using the service.
-   **Operational Security (OpSec) Matters:** Users seeking true anonymity must take additional steps beyond simply using an encrypted email provider. This includes **not providing recovery emails, not using credit cards for payment, and using Tor to access the service**.
</details>

<br>

* * *

## ☁️ 4) Encrypted Cloud Storage: Zero-Knowledge File Sync

Popular cloud storage services like Google Drive, Dropbox, and OneDrive have full access to your files. They can scan them for content, hand them over to authorities, or be breached. Zero-knowledge (or end-to-end encrypted) storage ensures that only you can read your data.

<details>
    <summary><b>🗄️ Tresorit and Cryptomator: A Two-Pronged Approach</b></summary>

There are two primary ways to achieve zero-knowledge cloud storage: using a provider that builds it in by default (**Tresorit**), or using client-side encryption software to lock your files before uploading them to any cloud (**Cryptomator**).

### Tresorit: The End-to-End Encrypted Cloud
Tresorit is a Swiss-based cloud storage service that builds end-to-end encryption into its very fabric. The name "Tresorit" comes from "tresor" (vault), and its entire infrastructure is designed as a zero-knowledge platform.

- **How it works:** Files are encrypted on your device before they are uploaded. The encryption keys are never sent to Tresorit's servers. This means Tresorit employees, hackers, or government agencies with a warrant cannot read your files. It has a "zero-knowledge" policy .
- **Key features:**
    - End-to-end encrypted file sync and sharing.
    - Granular permission management for shared folders ("tresors").
    - Secure link sharing with passwords and expiration dates.
    - Camera upload for automatic, encrypted photo backups.
    - No tracking, no access to your contacts .
- **Trade-offs:** It's a paid service with limited free storage. You are trusting the company's implementation of security, though they have a strong reputation.

### Cryptomator: The Universal Client-Side Encryptor
Cryptomator is a free, open-source tool that acts as a guardian for your files. It doesn't provide cloud storage itself; instead, it creates encrypted "vaults" on your device that you can then sync with **any** cloud provider (Dropbox, Google Drive, iCloud, OneDrive, etc.) .

- **How it works:** You create a vault, assign a password, and mount it like a virtual drive. Any file you put in the vault is automatically encrypted on your device before being synced to your chosen cloud folder. To access files, you unlock the vault, and Cryptomator decrypts them on the fly.
- **Key features:**
    - **Open Source & Auditable:** Its code is publicly available for scrutiny on GitHub.
    - **Platform Agnostic:** Works with any cloud service.
    - **Filename Encryption:** It can optionally encrypt filenames to hide directory structure.
    - **Award-Winning:** Received the CeBIT Innovation Award for Usable Security and Privacy .
- **Trade-offs:** It requires you to manage your own cloud storage and the encryption process. While user-friendly, it adds an extra step to file management. You are still using the potentially privacy-invasive cloud provider's infrastructure, but they only see encrypted, unreadable data.

**Which to choose?**
- Choose **Tresorit** for a seamless, integrated, all-in-one secure cloud storage experience .
- Choose **Cryptomator** for maximum flexibility, if you want to keep using your current cloud provider (e.g., free Google Drive storage) but want to secure your files .

**Resources:**
- **Tresorit:** [https://tresorit.com/](https://tresorit.com/)
- **Cryptomator:** [https://cryptomator.org/](https://cryptomator.org/)

</details>

<br>

* * *

## 🧠 5) Operational Security (OpSec) & Threat Modeling

Technology is only one part of the equation. Your habits, behaviors, and understanding of the risks are just as important. This section covers the often-overlooked human element of privacy.

<details>
    <summary><b>🎯 Threat Modeling: The Foundation of All Privacy</b></summary>

Before you choose any tool, you must define your **threat model**. This is a structured way to identify your risks and decide which strategies are most appropriate. A threat model answers four basic questions:

1.  **What do I want to protect?** (e.g., my identity, my location, my communications, my files, my contacts).
2.  **Who do I want to protect it from?** (e.g., advertisers, my ISP, a nosy family member, a corporation, a government agency).
3.  **What is the likelihood of the threat?** (e.g., is a state-sponsored actor likely to target you, or are you more concerned about data brokers?).
4.  **How bad are the consequences if I fail?** (e.g., embarrassment, financial loss, physical danger, imprisonment).

Your threat model determines your tools. A journalist facing a repressive regime has a radically different threat model (and thus, toolset) than a casual user wanting to stop ad tracking. **LibreWolf might be overkill for the casual user, while Tor Browser is essential for the journalist.** Always choose your tools based on your specific, realistic risks.

</details>

<br>

<details>
    <summary><b>🧩 Identity Separation & Digital Footprint</b></summary>

A key operational security practice is to compartmentalize your online identities. Don't use the same accounts, browsers, or even devices for different personas (e.g., your professional self, your personal self, and a "research" self).

- **Separate Browsers/Profiles:** Use one browser (e.g., Brave) for your everyday, logged-in life (social media, banking). Use a completely different, hardened browser (e.g., LibreWolf or Mullvad Browser) for anonymous research and browsing where you are not logged in.
- **Dedicated Email Addresses:** Use different, unlinked email addresses for different purposes. For example, use Proton Mail for sensitive communications, and a completely separate, alias-based email (e.g., from SimpleLogin or AnonAddy) for newsletter signups.
- **Avoid Cross-Platform Linking:** Be mindful of how your accounts can be linked. Posting the same username on Reddit and Twitter, or using the same profile picture across platforms, can help data brokers and adversaries connect your identities.

</details>

<br>

<details>
    <summary><b>🗑️ Data Broker Opt-Outs & Personal Information Removal</b></summary>

Data brokers are companies that collect, aggregate, and sell personal information about you. They are a massive source of data for advertisers, background check services, and even stalkers. Your name, address, phone number, age, relatives, and more are likely for sale on dozens of these sites.

Manually opting out of every data broker site is a tedious but worthwhile task. There are services that automate this, but they cost money and require you to trust them with your information.

- **Manual Opt-Out Guides:** Websites like **inteltechniques.com** maintained by Michael Bazzell, provide extensive, step-by-step guides for manually removing your information from the most common data broker sites.
- **Automated Services:** Services like **DeleteMe (Abine)** or **OneRep** will, for a fee, submit opt-out requests on your behalf and monitor for your information to reappear.

</details>

<br>

* * *

<hr> 

### ~ F.A.Q. ~

  * Why avoid Session / Matrix (element) / Telegram ? 
<details>
<summary><b>Session:</b></summary>

# Comprehensive Analysis of Session Messenger: Security, Network, and Controversies

This summarizes various security concerns, design criticisms, and controversies associated with the Session messenger application. It compiles user claims, technical analyses from online communities, and a professional security audit conducted by Quarkslab ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf)).

## Table of Contents
- [Network & Sybil Resistance Mechanism](#network--sybil-resistance-mechanism)
- [Cryptographic Design & Implementation](#cryptographic-design--implementation)
- [Application & Network Flaws](#application--network-flaws)
- [Moderation, Jurisdiction, and Governance Issues](#moderation-jurisdiction-and-governance-issues)
- [Recent Developments (Protocol V2)](#recent-developments-protocol-v2)
- [Summary of Findings](#summary-of-findings)

---

## Network & Sybil Resistance Mechanism

| Issue | Details |
| :--- | :--- |
| **Claimed vs. Actual Sybil Resistance** | The mechanism to prevent Sybil attacks (requiring a large stake of Oxen cryptocurrency to run a node) is criticized as ineffective and counterproductive. Instead of fostering a decentralized, accessible network, it creates a financial barrier. Critics argue this "guarantee[s] only governments and other well-funded organizations... will ever have the financial resources to run nodes," which is the opposite of the project's stated goal of protecting against powerful adversaries ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). |
| **Mechanism** | The Session network (LokiNet) requires operators to stake $12,000 worth of Oxen to run a node, which is framed as a conflict of interest due to the project's promotion of its own cryptocurrency ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). |

---

## Cryptographic Design & Implementation

| Issue | Details |
| :--- | :--- |
| **Seed Length and Security** | Session uses 128-bit seeds for account generation. The Quarkslab audit formally identified this as a weakness (**SESS-AND-04**, **SESS-IOS-04**), referencing DJ Bernstein's warning about the dangers of insufficient randomness in keys. The auditors noted it as a "Low" severity finding but confirmed the practice of generating the seed from 16 bytes (128 bits) of randomness ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 30-31). |
| **Developer Response on Seed Length** | The developers defended this choice, arguing that because the 128-bit seed is hashed with SHA-512 for Ed25519 key generation, it does not weaken the cryptographic properties. They stated the reduction was a deliberate UX choice for a shorter 13-word recovery phrase and that a brute-force attack against \(2^{128}\) possibilities is "simply not practical" ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 8, 10). This remains a point of contention between security purists and the project. |
| **Lack of Perfect Forward Secrecy (PFS)** | Session has been criticized for dropping Perfect Forward Secrecy (PFS) and deniability, which are considered essential security features in other messaging apps like Signal. The removal of PFS means that if a user's long-term private key is compromised, all past messages can be decrypted, greatly increasing the potential damage of a successful attack ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). The developer rationale was that "under typical circumstances, the only way long term keys can be compromised is through full physical device access — in which case an attacker could simply pull the already-decrypted messages from the local database" ([Lemmy Discussion](https://lemmy.ml/comment/11523283)). |
| **Public Keys as AES-GCM Keys** | Security researcher Soatok identified a serious cryptographic misuse where Session reportedly uses public keys directly as keys for AES-GCM symmetric encryption ([Dan Goodin/Mastodon](https://infosec.exchange/@dangoodin/113833559546590221)). |
| **Signature Validation Issue** | Public keys are sent within the same message they're meant to verify, meaning there's no out-of-band verification that the public key actually belongs to the claimed sender. As one commenter noted: "if the protocol expects you to just trust the public key sent in the very same message as the signature, then the signature serves no purpose" ([Robert Gützkow/Mastodon](https://infosec.exchange/@robertguetzkow/113836803832025941)). |
| **Protocol V2 and ML-KEM (Kyber) Concerns** | Session Protocol V2 plans to implement the ML-KEM (Kyber) post-quantum algorithm. Critics, referencing public comments made to NIST, have raised serious concerns about the use of Kyber, particularly if it is not implemented as an "extra layer of defense" alongside existing pre-quantum encryption. The concern is that attackers could be breaking Kyber-512 today and storing encrypted traffic for future decryption ([D.J. Bernstein NIST Comments, Nov 2023](https://csrc.nist.gov/files/pubs/fips/203/ipd/docs/fips-203-initial-public-comments-2023.pdf)). |
| **Kyber Standardization Concerns** | Further concerns about Kyber include the removal of a security feature (a hash over the DRBG output) during the NIST standardization process, which critics like Jacob Appelbaum argue accommodates potential backdoors like the one in the flawed Dual_EC_DRBG standard ([Jacob Appelbaum NIST Comments, Nov 2023](https://csrc.nist.gov/files/pubs/fips/203/ipd/docs/fips-203-initial-public-comments-2023.pdf)). |

---

## Application & Network Flaws

| Issue | Details |
| :--- | :--- |
| **CVE-2024-2045: Path Traversal Vulnerability** | A documented security vulnerability (CVE-2024-2045) was discovered in Session version 1.17.5 that allows unauthorized local file access through chat attachments. Attackers with low privileges can obtain internal and public files from a user's device without consent. CVSS Score: 5.5 (Medium) with "High" confidentiality impact ([CVE-2024-2045 Details](https://feedly.com/cve/CVE-2024-2045)). |
| **Critical TLS Verification Failure (Android)** | The Quarkslab audit identified a **High severity** vulnerability (**SESS-AND-03**) where the Android client completely lacked TLS certificate verification when fetching the initial list of Service Nodes (seed nodes). This allowed an attacker capable of DNS poisoning or MITM attacks to feed the client a rogue list of nodes, effectively taking control of the user's connection to the network. The vendor fixed this in releases 1.5.4 and 1.9.0 ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 27-30). |
| **Lack of Certificate Pinning (iOS)** | A similar, though less severe, issue (**SESS-IOS-02**) was found on iOS. The initial connection to seed nodes did not use certificate pinning, making it vulnerable to a compromised Certificate Authority. This was fixed in iOS release 1.9.4 ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 39-40). |
| **Plaintext Attachment Storage (iOS)** | The audit found that message attachments on iOS were stored in plaintext (**SESS-IOS-01**), which could expose them to an attacker with physical access to an unlocked device. The vendor's response was that they were working on a user-defined password option to encrypt the local database ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 38-39). |
| **File Server Unauthenticated Upload** | The audit discovered that the file server (`file.getsession.org`) allowed arbitrary file uploads and downloads without any authentication. While files were encrypted with unique keys, this behavior raised concerns about the potential for abuse and server-side vulnerabilities ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 22-23). |
| **Link Preview IP Leak (Desktop Client)** | A reported critical flaw is that when link previews are enabled, the Session desktop client generates the preview by connecting *directly* to the target website. This bypasses Session's own onion routing and leaks the user's real IP address to the web server hosting the link. This behavior has been confirmed by both users and a Session developer ([GitHub Issue #1743](https://github.com/oxen-io/session-desktop/issues/1743), [Privacy Guides Community](https://discuss.privacyguides.net/t/critical-flaws-in-desktop-session-messenger/16674/11)). |
| **Developer Response on Link Previews** | A Session developer noted that this feature is disabled by default and that future integration with Lokinet is intended to route these requests through the onion network ([GitHub Issue #1743](https://github.com/oxen-io/session-desktop/issues/1743)). The official explanation is that the client connects directly to fetch preview metadata, then encrypts and uploads the image via onion routing ([Privacy Guides Community](https://discuss.privacyguides.net/t/critical-flaws-in-desktop-session-messenger/16674/11)). |
| **Non-Lokinet Traffic (Android & iOS)** | The audit flagged that push notification registration (**SESS-AND-05**) and file downloads (**SESS-IOS-03**) were not routed through the Lokinet onion network, potentially leaking user metadata to ISPs. These issues were reportedly fixed in subsequent releases ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 28, 32-33, 40). |
| **Voice/Video Call IP Exposure** | When you enable calls, a pop-up alert informs you that **your IP address will be visible to the person you're calling and a Session server**. This means voice/video calls are **not anonymous** and can reveal your geolocation, contradicting the app's core privacy promise for users who enable calling features ([PCMag Review](https://me.pcmag.com/en/communications/24982/session)). |
| **Deterministic File Encryption** | An upcoming change to Session's file encryption has been criticized for making the encryption of attachments *deterministic*. This would mean that the same file uploaded multiple times by the same user would produce an identical encrypted blob. This could allow an attacker who can observe the file server to identify when a specific file (e.g., a signal for illegal activity) is being sent, without needing to break the encryption ([Privacy Guides Discussion](https://discuss.privacyguides.net/t/session-messenger-prepares-to-weaken-file-encryption/32591/2)). The cited reason is for server-side deduplication. |

---

## Moderation, Jurisdiction, and Governance Issues

| Issue | Details |
| :--- | :--- |
| **Centralized Moderation and "Shadowbanning"** | Public groups on Session are controlled by server operators who appear to collude. This gives them ultimate power over non-private communications, leading to a form of "shadowbanning" where a user banned from one group can be effectively banned from nearly all groups for voicing concerns ([Reddit Discussion](https://old.reddit.com/r/signal/comments/1c6mj2c/session_app_is_now_censoring_and_banning_users/)). Communities (open groups) rely on community-operated servers and may have different moderation policies ([OSINT Team Article](https://osintteam.blog/session-private-messenger-a-look-at-features-privacy-security-and-usage-db4504ebe605)). |
| **No In-App Reporting Mechanism** | Multiple users have reported that when encountering abusive content or CSAM in groups, there is no way to report within the app. One top Google Play Store review (Feb 2026) stated: "Session was great until I got added to a group chat that was just posting child porn. I could find no way to report that... there's no way to link the full group, and no way to report them or the entire group on the app itself". |
| **Session's Official Response on Reporting** | "Due to Session's decentralized design, groups cannot be centrally moderated or reported within the app. We appreciate you taking the time to report these users to the authorities". |
| **User Reports of CSAM in Groups** | Multiple App Store reviews corroborate concerns: "there needs to be a report system in place. ive come across too much stuff that should not be allowed on any platform"; "This app is mostly used by creeps and offenders there should at least be a report feature because it is vile what I have seen"; "Many p3d0s use this app to chat with children or exchange illegal stuff". |
| **Company Jurisdiction** | The company behind Session is based in Australia, a country with what some privacy advocates describe as "very unfavorable privacy laws" due to mandatory data retention and other surveillance legislation ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). Note: The Session Technology Foundation is now based in Switzerland, though the original company connections remain ([OSINT Team Article](https://osintteam.blog/session-private-messenger-a-look-at-features-privacy-security-and-usage-db4504ebe605)). |
| **Funding Transparency** | The project's funding is described as "completely opaque," adding to the concerns of privacy-focused users ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). |
| **Cryptocurrency Conflict of Interest** | Critics argue that running the Oxen token (now Session Token) alongside the messenger creates a conflict of interest, as the Sybil resistance mechanism doubles as a way to increase token value ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). |
| **Privacy Policy Contradiction** | While Session markets itself as collecting no data, their privacy policy acknowledges that **Apple and Google may store information** about users and their devices when downloading the app from official app stores ([OSINT Team Article](https://osintteam.blog/session-private-messenger-a-look-at-features-privacy-security-and-usage-db4504ebe605)). |

---

## Recent Developments (Protocol V2)

It's worth noting that Session has announced plans to address some criticisms in their upcoming V2 Protocol ([Privacy Guides - V2 Announcement](https://www.privacyguides.org/news/2025/12/03/session-messenger-adds-pfs-pqe-and-other-improvements/)):

| Planned Improvement | Details |
| :--- | :--- |
| **Reintroducing Perfect Forward Secrecy (PFS)** | After years of criticism for removing PFS, Session plans to bring it back using rotating key pairs. Accounts will establish rotating key pairs for each linked device, with old keys deleted after a period of time. |
| **Adding Post-Quantum Encryption** | Will use ML-KEM (Kyber), the NIST-approved standard already used by Signal and iMessage. |
| **Linked Device Management** | Users will be able to see when new devices are linked to their account and remove them remotely. |
| **Infrastructure Upgrades** | Migration of core cryptographic logic into a shared library called `libsession` and development of "Config Messages" for synchronizing data across linked devices. |

However, these are **announced plans, not yet implemented features**. The V2 Protocol is not finalized, and additional details are expected in 2026.

---

## Summary of Findings

| Category | Specific Issue | Summary / Status |
| :--- | :--- | :--- |
| **Network** | Sybil Resistance | Staking model criticized as paywall for node operators, not effective Sybil protection ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). |
| **Network** | Node Operation Cost | $12,000 staking requirement creates barrier to entry for node operators ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). |
| **Cryptography** | Seed Entropy | 128-bit seeds confirmed by Quarkslab audit; defended by devs for UX ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 8, 10, 30-31). |
| **Cryptography** | Perfect Forward Secrecy | Deliberately removed (to be reintroduced in V2 Protocol); greatly increases impact of key compromise ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0), [Privacy Guides](https://www.privacyguides.org/news/2025/12/03/session-messenger-adds-pfs-pqe-and-other-improvements/)). |
| **Cryptography** | Public Keys as AES-GCM Keys | Cryptographic misuse where public keys are used directly for symmetric encryption ([Dan Goodin/Mastodon](https://infosec.exchange/@dangoodin/113833559546590221)). |
| **Cryptography** | Signature Validation | No out-of-band verification that public keys belong to claimed senders ([Robert Gützkow/Mastodon](https://infosec.exchange/@robertguetzkow/113836803832025941)). |
| **App Security** | CVE-2024-2045 Path Traversal | Documented vulnerability allowing unauthorized local file access via chat attachments ([CVE-2024-2045](https://feedly.com/cve/CVE-2024-2045)). |
| **App Security** | **High Severity - TLS Verification (Android)** | **Fixed.** Complete lack of TLS verification during node bootstrap. Patched in versions 1.5.4 and 1.9.0 ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 27-30). |
| **App Security** | No Certificate Pinning (iOS) | **Fixed.** Initial node connections lacked pinning. Patched in version 1.9.4 ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 39-40). |
| **App Security** | Plaintext Attachments (iOS) | **Unresolved (as of audit).** Attachments stored unencrypted on device. Vendor planned to add database encryption option ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 38-39). |
| **App Security** | File Server Behavior | Unauthenticated uploads possible; server returned useful error messages, aiding potential attackers ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 22-23). |
| **App Security** | Link Previews | Desktop client leaks IP by fetching previews directly, bypassing onion routing ([GitHub #1743](https://github.com/oxen-io/session-desktop/issues/1743)). |
| **App Security** | Non-Lokinet Traffic | **Fixed.** Push notification registration (Android) and file downloads (iOS) were not onion-routed. Patched in 1.5.4 and 1.5.3 respectively ([Quarkslab Audit 2021](https://blog.quarkslab.com/resources/2021-05-04_audit-of-session-secure-messaging-application/20-08-Oxen-REP-v1.4.pdf), pages 28, 32-33, 40). |
| **App Security** | Voice/Video Calls | IP address visible to call recipient and Session servers ([PCMag Review](https://me.pcmag.com/en/communications/24982/session)). |
| **App Security** | File Encryption | Planned deterministic encryption could allow traffic analysis of file content ([Privacy Guides Discussion](https://discuss.privacyguides.net/t/session-messenger-prepares-to-weaken-file-encryption/32591/2)). |
| **Moderation** | Centralized Group Control | Reports that public group server operators collude, enabling cross-group "shadowbanning" ([Reddit](https://old.reddit.com/r/signal/comments/1c6mj2c/session_app_is_now_censoring_and_banning_users/)). |
| **Moderation** | No Reporting Mechanism | No in-app way to report abusive content or CSAM in groups. |
| **Moderation** | CSAM Presence | Multiple user reports of CSAM in groups with no recourse. |
| **Governance** | Jurisdiction | Company connections to Australia with unfavorable privacy laws ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). |
| **Governance** | Funding | Project funding described as "completely opaque" ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). |
| **Governance** | Conflict of Interest | Cryptocurrency integration creates financial incentives tied to network participation ([THGTOA Caution](https://gitea.com/anonymousplanet/thgtoa/commit/12b99c9ea9a1d5c2013affbc12a45a32329715b0)). |
| **Usability** | Performance | Slow text messaging and media file loading due to decentralization trade-offs ([PCMag Review](https://me.pcmag.com/en/communications/24982/session)). |
| **Usability** | Platform Inconsistency | Screenshot alerts work on Android but not iOS ([PCMag Review](https://me.pcmag.com/en/communications/24982/session)). |

---

*Note: Some references (such as the GitHub issues) represent ongoing discussions and may have been updated or resolved after the time of this writing. The Protocol V2 improvements are announced but not yet implemented.*
</details>
<details>
<summary><b>Matrix(element):</b></summary>

# Comprehensive Analysis of Matrix / Element: Protocol, Security, and Governance Concerns

This summarizes various security concerns, design criticisms, and governance controversies associated with the Matrix protocol and its flagship client, Element. It compiles community analyses, technical critiques, and reports on organizational practices.

## Table of Contents
- [Protocol Design & Metadata Leakage](#protocol-design--metadata-leakage)
- [Malicious Homeserver Admin Capabilities](#malicious-homeserver-admin-capabilities)
- [Core Protocol Weaknesses](#core-protocol-weaknesses)
- [Encryption Protocol Issues (Megolm)](#encryption-protocol-issues-megolm)
- [Resource Consumption & Scalability](#resource-consumption--scalability)
- [Organizational & Governance Issues (Matrix.org)](#organizational--governance-issues-matrixorg)
- [Comparison with SimpleX](#comparison-with-simplex)
- [Summary of Findings](#summary-of-findings)

---

## Protocol Design & Metadata Leakage

| Issue | Details |
| :--- | :--- |
| **Inherent Metadata Exposure** | Federated networks like Matrix are naturally more vulnerable to metadata leaks than P2P or centralized networks. Some leaks are necessary for protocol functionality (e.g., verifying messages requires knowing the sender's device), while others are accepted for performance (e.g., unencrypted reactions and read receipts) ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Unencrypted Data Fields** | Matrix's end-to-end encryption does not encrypt the following information, leaving it visible to homeservers and observers ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)): |
| | • Message senders and timestamps |
| | • Join/leave/invite events |
| | • Message edits (though not content) |
| | • Reactions and read receipts |
| | • Nicknames and profile pictures |
| **Design Flaws vs. Features** | While some metadata leaks are inherent to the protocol's design, others are simply failures to consider encryption. For example, room-specific nicknames, room-specific profile pictures, and message edit events could all be encrypted without breaking the protocol, making their exposure a design flaw ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |

---

## Malicious Homeserver Admin Capabilities

| Attack Type | Details |
| :--- | :--- |
| **Passive Information Gathering** | A malicious homeserver admin can retroactively gather extensive metadata by querying the Synapse database, including ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)): |
| | • Chat history of any unencrypted room |
| | • User information (devices, IP addresses) |
| | • Reactions to encrypted messages (since they are unencrypted) |
| | • Room metadata for encrypted rooms: participants, avatars, nicknames, topics, message frequency and timing |
| | • URL previews of shared links (if enabled) |
| **Active Attacks - Room Manipulation** | An admin can impersonate users to send unencrypted "state events," enabling various social engineering attacks ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)): |
| | • React to messages as the impersonated user |
| | • Set room topic to an attacker-controlled URL (visible to all participants) |
| | • Invite malicious accounts into the room |
| | • Kick, ban, or modify power levels of users |
| | • Send "tombstone" events to mark rooms as replaced |
| **Active Attacks - Device Compromise** | An admin can add a new device to a user's account, allowing them to send and receive encrypted messages. While this typically shows as an "unverified device" with a warning shield, the article notes that "most people, even privacy minded tech savvy ones, simply ignore this for various reasons" ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |

---

## Core Protocol Weaknesses

| Issue | Details |
| :--- | :--- |
| **Append-Only Design** | Events in Matrix cannot be deleted, leading to endless history accumulation. Redaction events are merely advisory; poorly behaving servers may ignore them and retain content. This compromises user deniability and can lead to data persistence beyond a user's control ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **State Resolution Complexity** | The consensus algorithm for resolving conflicting room states is complex and not foolproof. Servers with different implementations can experience "split-brained" rooms where views diverge, leading to "state resets" that can strip admins of their powers and cause significant disruptions ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Message Forging** | It is possible to insert plausible-looking events into message history. Due to the complexity of state resolution and signature verification across different server implementations, such forgeries may go unnoticed by users ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **JSON Interoperability Issues** | The lack of a strict definition for "canonical JSON" leads to signature mismatches between different server implementations (e.g., Python vs. Rust). This can cause events from one server to be rejected by another, contributing to split-brain scenarios ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Media Handling Vulnerabilities** | Unauthenticated media uploads and eager replication create significant risks ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)): |
| | • Anyone can use a server's media repository for storage |
| | • Homeservers can be tricked into replicating media from other servers, potentially leading to denial-of-service |
| | • No default scanning for illegal content (CSAM, viruses) |
| | • Servers may become liable for hosting illegal media replicated from undesirable rooms |

---

## Encryption Protocol Issues (Megolm)

| Issue | Details |
| :--- | :--- |
| **Published Cryptographic Vulnerabilities** | A security analysis ("Nebuchadnezzar") reported multiple practically-exploitable vulnerabilities in Matrix's Megolm protocol, even when encryption and verification are enabled. These include ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)): |
| | • Simple confidentiality break |
| | • Attack against out-of-band verification |
| | • Semi-trusted impersonation |
| | • Trusted impersonation |
| | • Impersonation leading to confidentiality break |
| | • IND-CCA break |
| **Attacker Model** | All reported attacks require cooperation of the homeserver. This is considered a natural threat model for end-to-end encryption, which aims to provide protection against untrusted third parties ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Fragile Encryption** | Matrix's encryption relies on reliable device list updates. Failures in this system can lead to broken encryption or situations where messages are sent to unverified devices without proper warnings ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Optional Encryption** | End-to-end encryption is not mandatory in Matrix. Rooms can be created without it, potentially exposing message content in federated rooms if participants are not vigilant ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |

---

## Resource Consumption & Scalability

| Issue | Details |
| :--- | :--- |
| **High Operational Costs** | Running a public Matrix server (Synapse) requires significant resources. Depending on user count, multiple worker containers are needed—effectively requiring between 4 and 12 instances of Synapse ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Resource Intensity** | Synapse is described as "hard drive hungry, memory hungry, relatively CPU hungry, and also uses a lot of bandwidth." The article warns: "Don't expect to run this pile of bloat without throwing money at it" ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |

---

## Organizational & Governance Issues (Matrix.org)

| Issue | Details |
| :--- | :--- |
| **Data Collection Practices** | Research documented in "Notes on privacy and data collection of Matrix.org" reveals that matrix.org and vector.im receive extensive personal data even when users host their own instances ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). Data collected includes: |
| | • Matrix IDs (including usernames) |
| | • Email addresses and phone numbers |
| | • Associations between email/phone and Matrix IDs |
| | • Usage patterns |
| | • IP addresses (providing geolocation) |
| | • Device and system information |
| | • Other servers users communicate with |
| | • Room IDs (including potential identification of direct chats) |
| **Publicly Accessible Data** | With default settings, Matrix.org allows unrestricted public access to ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)): |
| | • Mappings of Matrix IDs to email addresses/phone numbers |
| | • Every uploaded file (images, videos, audio) |
| | • Profile names and avatars |
| **CSAM and Abuse Problems** | The platform has been criticized as a "safe haven" for abusive content ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)): |
| | • Rooms cannot be forcibly shut down across the entire federation, allowing abusive rooms to persist on other servers |
| | • Media replication can cause servers to unknowingly host illegal content (CSAM, copyrighted material) |
| | • The Matrix.org abuse team is described as "notoriously unresponsive" to reports |
| | • The article claims: "Every homeserver in the federation is likely hosting child sexual abuse images and videos" |
| **Cloudflare Man-in-the-Middle** | Matrix.org and vector.im terminate TLS connections through Cloudflare, evidenced by `cf-ray` and `server: cloudflare` headers. This introduces Cloudflare as a man-in-the-middle capable of inspecting traffic, contradicting claims of end-to-end encryption alone being sufficient ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Removed Tor Browser Support** | Element-web no longer supports Tor Browser, with a developer comment stating: "Due to lack of funding Tor is not considered a supported browser." This limits anonymous access for privacy-conscious users ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |

---

## Comparison with SimpleX

The Hack Liberty article announces a move to SimpleX Chat and provides a detailed comparison. The key differences are summarized below:

| Feature | SimpleX | Matrix |
| :--- | :--- | :--- |
| **User Identifiers** | No user identifiers at all; uses unidirectional message queue addresses that can be rotated ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). | Uses Matrix IDs (usernames) as persistent identifiers ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Metadata Protection** | Private 2-hop onion routing protects connection metadata from server operators ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). | Federated architecture inherently leaks metadata; many fields are unencrypted ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Encryption Protocol** | Double-ratchet with post-quantum resistant key exchange and additional encryption layers ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). | Megolm protocol with known cryptographic vulnerabilities ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Decentralization Model** | Fully fragmented network with no central components, bootstrap nodes, or global state. Servers are not connected or known to each other ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). | Federated network with central components (like matrix.org) and global shared state. Relies on DNS for discovery ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Media Handling** | Local file encryption; manual message queue rotations for moving conversations ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). | Unverified uploads by default; eager replication can cause servers to host illegal or copyrighted material ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Tor Support** | Native Tor support with proxy and onion-only routing features ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). | Tor browser support removed; not considered a supported platform ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Cloudflare Dependency** | No Cloudflare dependency; TLS is terminated directly on servers ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). | Matrix.org uses Cloudflare for TLS termination, introducing a MITM point ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |

---

## Summary of Findings

| Category | Specific Issue | Summary / Status |
| :--- | :--- | :--- |
| **Metadata** | Unencrypted Fields | Message senders, timestamps, join/leave events, reactions, read receipts, nicknames, and profile pictures are not encrypted and visible to servers ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Admin Threats** | Passive Data Gathering | Malicious admins can access extensive metadata from database, including IPs, devices, message timing, and room information ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Admin Threats** | Active Attacks | Admins can impersonate users to manipulate rooms (topic, invites, kicks) or add devices to read encrypted messages ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Protocol** | Append-Only Design | Events cannot be deleted; redactions are advisory and can be ignored by malicious servers, compromising deniability ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Protocol** | State Resolution | Complex consensus algorithm leads to "split-brained" rooms and state resets that can remove admin powers ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Protocol** | Message Forging | Possible to insert plausible-looking events into history due to implementation inconsistencies ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Encryption** | Megolm Vulnerabilities | Multiple published exploits (confidentiality breaks, impersonation) requiring only homeserver cooperation ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Encryption** | Optional & Fragile | E2EE is not mandatory; relies on reliable device list updates which can fail, leading to broken encryption ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Media** | Unauthenticated Uploads | Anyone can use server storage; no default scanning for illegal content ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Media** | Replication Risks | Servers can be tricked into replicating media from undesirable rooms, potentially hosting CSAM or copyrighted material ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Governance** | Data Collection | Matrix.org collects extensive personal data (IDs, emails, IPs, usage patterns) even from users on other servers ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Governance** | CSAM Problems | Platform criticized as safe haven for pedophiles; abuse team unresponsive; replication means many servers likely host illegal content ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Governance** | Cloudflare MITM | TLS termination through Cloudflare introduces a man-in-the-middle capable of inspecting traffic ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Governance** | Tor Support Removed | Element no longer supports Tor Browser, limiting anonymous access ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |
| **Operations** | Resource Consumption | Running Synapse requires significant hardware resources (multiple workers, high RAM/CPU/bandwidth) ([Hack Liberty Forum](https://forum.hackliberty.org/t/why-we-abandoned-matrix-the-dark-truth-about-user-security-and-safety/224)). |

---

*Note: This analysis draws heavily from community critiques and technical assessments linked in the source article. Some claims (particularly regarding CSAM prevalence) are difficult to verify independently but represent significant community concerns.*
</details>
<details>
<summary><b>Telegram:</b></summary>

# Comprehensive Analysis of Telegram: Security, Privacy, and Governance Concerns

This summarizes various security issues, cryptographic critiques, and significant governance changes associated with the Telegram messaging app. It compiles information from academic research, news reports, and public disclosures.

## Table of Contents
- [Cryptographic Protocol & Design Issues](#cryptographic-protocol--design-issues)
- [Platform Security & Infrastructure Concerns](#platform-security--infrastructure-concerns)
- [Governance, Data Sharing & Legal Issues](#governance-data-sharing--legal-issues)
- [Targeted Surveillance & Device-Level Vulnerabilities](#targeted-surveillance--device-level-vulnerabilities)
- [Summary of Findings](#summary-of-findings)

---

## Cryptographic Protocol & Design Issues

| Issue | Details |
| :--- | :--- |
| **MTProto 1.0 Vulnerabilities** | In December 2015, researchers from Aarhus University demonstrated that Telegram's original MTProto 1.0 encryption protocol did not achieve indistinguishability under chosen-ciphertext attack (IND-CCA) or provide authenticated encryption. While the attack was theoretical and not a full plaintext recovery, it highlighted the use of a less secure scheme when better alternatives existed. Telegram responded that the flaw did not affect message security and later patched it ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **MTProto 2.0 Improvements** | Telegram 4.6, released in December 2017, introduced MTProto 2.0, which satisfied the conditions for IND-CCA. Cryptographers view this as a vast improvement over the original protocol ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **2021 Security Analysis** | Researchers from Royal Holloway, University of London and ETH Zurich published an analysis of MTProto 2.0 in July 2021. They concluded the protocol could provide a "confidential and integrity-protected channel" but identified theoretical vulnerabilities, including the potential for message reordering. Telegram patched the issues before publication and paid a security bounty to the researchers ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Lack of Default End-to-End Encryption** | A long-standing criticism is that Telegram does not apply end-to-end encryption to all chats by default. Only "Secret Chats" use E2EE, while regular cloud chats are encrypted client-server. This has led organizations like the Electronic Frontier Foundation and the Norwegian National Security Authority to advise caution, especially compared to apps like Signal and WhatsApp which enable E2EE by default ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Non-Standard Protocol** | Telegram's use of its own MTProto protocol, rather than a well-reviewed standard like the Signal Protocol, has been repeatedly flagged by cryptography researchers, including Matthew Green, as a point of concern ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |

---

## Platform Security & Infrastructure Concerns

| Issue | Details |
| :--- | :--- |
| **Server-Side Code Not Open Source** | Despite a 2014 promise to eventually release all server-side code, Telegram has not done so. In 2021, Pavel Durov explained this decision, citing the difficulty for users to verify that the released code matches the code run on servers, and concerns about governments forcing the company to hand over server code to create competing networks ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Russian Infrastructure Concerns (2025)** | In June 2025, an investigation by iStories and OCCRP revealed that key parts of Telegram's technical infrastructure are operated by companies owned by a network engineer with a history of collaboration with Russian intelligence services. This raised serious concerns over potential metadata access and user surveillance ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Self-Destruct Bug (2021)** | A Russian researcher discovered a bug in the self-destruct feature that allowed users to recover deleted photos from their own device. Telegram patched the issue before public disclosure and offered a €1,000 bug bounty, though the researcher declined the award due to an accompanying NDA ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Malware Exploit (2024)** | In July 2024, cybersecurity firm ESET reported a vulnerability that allowed malicious files to be sent to users while being masked as legitimate multimedia ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |

---

## Governance, Data Sharing & Legal Issues

| Issue | Details |
| :--- | :--- |
| **FBI Backdoor Approaches (2016-2017)** | Pavel Durov has repeatedly stated that FBI agents approached him and a Telegram developer on multiple occasions, including at his rented home in San Francisco during Google I/O 2016. The agents allegedly offered a bribe of "tens of thousands of dollars" to a developer to act as an informer and requested a "backchannel process" for handing over user data in emergencies. Durov claims he refused, citing Telegram's lack of legal presence in the US as a reason for non-cooperation ([Neowin](https://www.neowin.net/news/fbi-asked-durov-and-developer-for-telegram-backdoor/), [Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Post-Arrest Data Sharing Pivot (2024-2025)** | Following the arrest of Pavel Durov in France in August 2024 on charges related to Telegram's use for organized crime, the company made a significant policy shift. Durov promised to improve cooperation with authorities and provide IP addresses and phone numbers of users who violate rules in response to valid legal requests ([SecurityWeek](https://www.securityweek.com/telegram-shared-data-of-thousands-of-users-after-ceos-arrest/), [Forbes Archive](https://web.archive.org/web/20250330075035/https://www.forbes.com/sites/thomasbrewster/2025/01/07/telegram-hands-data-on-thousands-of-users-to-law-enforcement/)). |
| **Massive Surge in Data Disclosure (2024)** | Data from Telegram's transparency reports shows a dramatic increase in cooperation following Durov's arrest. Specific figures include ([Forbes Archive](https://web.archive.org/web/20250330075035/https://www.forbes.com/sites/thomasbrewster/2025/01/07/telegram-hands-data-on-thousands-of-users-to-law-enforcement/)): |
| | • **United States:** 900 requests affecting **2,253 users** (compared to only 14 requests in the first nine months of 2024). |
| | • **Germany:** 945 requests affecting **2,237 users**. |
| | • **Spain:** 213 requests affecting **518 users**. |
| | • **United Kingdom:** 142 requests affecting **293 users**. |
| **Impact on Criminal Networks** | The policy shift has had a tangible deterrent effect. A federal law enforcement investigator stated that individuals creating child sexual abuse material (CSAM) are "increasingly aware that [Telegram] is not always the case [safe from government reach] and are exploring new tactics to avoid detection" ([Forbes Archive](https://web.archive.org/web/20250330075035/https://www.forbes.com/sites/thomasbrewster/2025/01/07/telegram-hands-data-on-thousands-of-users-to-law-enforcement/)). |
| **Telegram's Profitability** | The company became profitable for the first time in 2025, with total revenue surpassing $1 billion ([Forbes Archive](https://web.archive.org/web/20250330075035/https://www.forbes.com/sites/thomasbrewster/2025/01/07/telegram-hands-data-on-thousands-of-users-to-law-enforcement/)). |
| **Account Hijacking via SMS Interception** | Multiple incidents have been reported where Telegram accounts were hijacked by intercepting SMS login codes. This has occurred in Iran, Russia, and Germany, possibly in coordination with telecom companies. Durov has recommended users enable two-factor authentication to mitigate this risk ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Denial-of-Service Attack (2019)** | Telegram confirmed a one-hour denial-of-service attack in June 2019, with Durov stating that the IP addresses used in the attack mostly came from China ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |

---

## Targeted Surveillance & Device-Level Vulnerabilities

| Issue | Details |
| :--- | :--- |
| **Pavel Durov on Pegasus Leak List (2018)** | The leaked data from the Pegasus Project revealed that Pavel Durov's UK mobile number appeared on a list of individuals selected by an NSO Group client government in early 2018. The timing coincided with Durov changing his official residence to the UAE, leading to speculation that he may have been of interest to authorities there. NSO stated that appearing on the list does not necessarily mean a number was selected for surveillance, and it is unknown if any attempt to install Pegasus was made ([The Guardian](https://web.archive.org/web/20260205080353/https://www.theguardian.com/news/2021/jul/21/telegram-founder-pavel-durov-listed-spyware-targets-nso-leak-pegasus)). |
| **Device as the Weakest Link** | Security experts note that powerful spyware like Pegasus renders the security of any individual messaging app irrelevant once a device is infected. The spyware can access data from Telegram, WhatsApp, Signal, and other apps post-infection. The only fully secure device is one that is turned off ([The Guardian](https://web.archive.org/web/20260205080353/https://www.theguardian.com/news/2021/jul/21/telegram-founder-pavel-durov-listed-spyware-targets-nso-leak-pegasus)). |
| **RampantKitten Campaign (2020)** | Check Point Research uncovered an Iranian surveillance group, RampantKitten, which ran a phishing and surveillance campaign targeting dissidents. The attack involved malware that replaced Telegram files on compromised devices to clone session data. This highlights that device compromise, rather than app protocol flaws, is often the attack vector ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Iranian Hackers (2016)** | Reuters reported that Iranian hackers compromised over a dozen Telegram accounts and identified the phone numbers of 15 million Iranian users by exploiting a programming interface. Telegram later limited such mass checks in its API ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Unofficial Clients Data Leak (2020)** | An Elasticsearch database holding 42 million records containing user IDs and phone numbers of Iranian users was exposed online. The data was extracted from unofficial "Telegram" clients, not the official app, in what appeared to be a government-sanctioned fork ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |

---

## Summary of Findings

| Category | Specific Issue | Summary / Status |
| :--- | :--- | :--- |
| **Cryptography** | MTProto 1.0 IND-CCA Flaw | Theoretical vulnerability allowing message modification; fixed in MTProto 2.0 (2017) ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Cryptography** | MTProto 2.0 Analysis | 2021 analysis found theoretical vulnerabilities (message reordering); patched before publication ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Cryptography** | Non-Default E2EE | Regular cloud chats are not end-to-end encrypted, only "Secret Chats" are. A major, long-standing criticism ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Cryptography** | Non-Standard Protocol | Use of proprietary MTProto instead of vetted protocols like Signal's is a recurring concern from experts ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Infrastructure** | Closed Server-Side Code | Server code is not open source, contrary to early promises, preventing independent verification ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Infrastructure** | Russian Intelligence Links | 2025 investigation linked key infrastructure operators to a figure with ties to Russian intelligence, raising metadata access concerns ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Platform Security** | Account Hijacking | Multiple incidents of SMS interception leading to account takeovers; 2FA recommended ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Platform Security** | Self-Destruct Bug | 2021 bug allowed recovery of "deleted" photos from local device; patched ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Platform Security** | Malware Vector | 2024 vulnerability allowed malicious files disguised as media; patched ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Governance** | FBI Backdoor Requests | Repeated alleged attempts by FBI to gain backdoor access or bribe a developer (2016-17); refused by Durov ([Neowin](https://www.neowin.net/news/fbi-asked-durov-and-developer-for-telegram-backdoor/), [Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |
| **Governance** | Post-Arrest Data Sharing | Following Durov's 2024 arrest, Telegram shifted policy to share IP/phone data with authorities in response to valid legal requests ([SecurityWeek](https://www.securityweek.com/telegram-shared-data-of-thousands-of-users-after-ceos-arrest/), [Forbes Archive](https://web.archive.org/web/20250330075035/https://www.forbes.com/sites/thomasbrewster/2025/01/07/telegram-hands-data-on-thousands-of-users-to-law-enforcement/)). |
| **Governance** | Data Sharing Surge (2024) | Massive increase in disclosures: **2,253 U.S. users** (900 requests), **2,237 German users** (945 requests). This shift is causing CSAM creators to consider leaving the platform ([Forbes Archive](https://web.archive.org/web/20250330075035/https://www.forbes.com/sites/thomasbrewster/2025/01/07/telegram-hands-data-on-thousands-of-users-to-law-enforcement/)). |
| **Governance** | Telegram Becomes Profitable | Company reached profitability in 2025 with revenue exceeding $1 billion ([Forbes Archive](https://web.archive.org/web/20250330075035/https://www.forbes.com/sites/thomasbrewster/2025/01/07/telegram-hands-data-on-thousands-of-users-to-law-enforcement/)). |
| **Surveillance** | Pegasus Target List | Durov's number appeared on a 2018 list of individuals of interest to an NSO client government, possibly the UAE. No confirmation of infection ([The Guardian](https://web.archive.org/web/20260205080353/https://www.theguardian.com/news/2021/jul/21/telegram-founder-pavel-durov-listed-spyware-targets-nso-leak-pegasus)). |
| **Surveillance** | Device-Level Compromise | Powerful spyware like Pegasus compromises the device, not the app, rendering any messaging app's encryption moot post-infection ([The Guardian](https://web.archive.org/web/20260205080353/https://www.theguardian.com/news/2021/jul/21/telegram-founder-pavel-durov-listed-spyware-targets-nso-leak-pegasus)). |
| **Surveillance** | RampantKitten Campaign | Iranian group used malware to clone Telegram session data from compromised devices, targeting dissidents ([Wikipedia](https://en.wikipedia.org/wiki/List_of_security_issues_associated_with_Telegram)). |

</details>

<hr>
