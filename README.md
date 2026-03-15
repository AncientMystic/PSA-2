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
  [
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
