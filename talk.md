---
type: slide
slideOptions:
  transition: slide
  theme: white
  width: 960
  height: 700
---

# disko and nixos-anywhere:

## Declarative and Remote Installation of NixOS

Speaker: Mic92 + lassulus

---

## Motivation

- In our work with numtide we are installing NixOS a lot
- We often deal with a wide range of platforms, that provides different ways of bootstrapping,
  not all of which supports NixOS as a native method.
- Building and uploading installer images to all these platforms is tedious
- What we want is to be able to bootstrap NixOS *anywhere* in the same way

---

## Background: Installing NixOS

1. Boot & access the NixOS installer on the target system
    - Highly dependent on the environment (virtual machines, bare-metal server, laptops ...)
2. Partition, format and mount the disks
3. Copy or write nixos configuration
4. Run `nixos-install`
5. Reboot

*Installing NixOS manually can be tedious and unergonomic*

---

## The solution:
### Nixos-anywhere + disko

1. **disko** is used to configure a disk layout.
2. **nixos-anywhere** installs your NixOS configuration with disko over ssh in a single command

---

## What is disko

### Input
- NixOS module

### Output
- 2 scripts (format + mount)
- NixOS configuration to mount disks

---

## Why not systemd-repart?

- repart has not enough features:
    - only single disk
    - no zfs
    - no lvm
- repart has some cool features though:
    - disk image creation without VM
    - Autoresize on boot
    - Can populate files

---

## Disko example: Partition table

```nix
disko.devices = {
  disk = {
    main = {
      device = "/dev/sda";
      content = {
        type = "gpt";
      };
    };
  };
}
```

---

## Disko example: Partitions

```nix
disko.devices.disk.main.content.partitions = {
  ESP = {
    type = "EF00";
    size = "500M";
    content = {
      type = "filesystem";
      format = "vfat";
      mountpoint = "/boot";
    }
  };
  root.size = "100%"
  root.content = {
    type = "filesystem";
    format = "ext4";
  }
}
```

---

## Other disko features

- ZFS with multiple devices
- Raids with mdadm/lvm
- Luks encryption
- Bcachefs
- disk image creation
- checkout [/example](https://github.com/nix-community/disko/tree/master/example) in the disko repo

---

## Challenges with disko

- Options documentation:
    - Disko options are recursive i.e. gpt -> luks -> gpt -> luks ...
    - How to generate documentation for this?

- Incremental updates:
    - Currently works only by complete wipe first
    - Would need state checks first

---

## What is nixos-anywhere

### Input

- NixOS configuration with a disko config
- SSH destination

### Output

- Installed NixOS on target

---

## How does nixos-anywhere work?

1. Identifies target
2. Runs kexec, if no nixos installer is detected
3. Runs disko
4. Runs nixos-install
5. Reboot

---

## Load the installer with Kexec

![](https://pad.lassul.us/uploads/02d9f6df-918f-4086-8f5f-84c06c431a35.png)

---

## Usage with flakes

```shell
nix run github:numtide/nixos-anywhere -- --flake '.#mysystem' root@yourhostname
```

* Where `.#mysystem` is your nixos configuration
* and `root@yourhostname` the ssh destination


---

## Other cool features of nixos-anywhere

**Run the interactive nixos-vm test**

```shell
nix run github:numtide/nixos-anywhere -- -f '.#mysystem' --vm-test
```

**Secret support**
  * nixos-anywhere supports upload secrets to the final system (i.e. bootstrapping agenix/sops-nix) and for disk encryption

---

## Demo

![asciicast](https://c.krebsco.de/nixos-anywhere.gif)

---

## Future work

* PXE boot. [Working branch, tests still missing](https://github.com/numtide/nixos-anywhere/pull/50)
* Automatically generating hardware configuration


---

## Summary

1. **disko** is used to configure a disk layout: 
2. **nixos-anywhere** installs your NixOS configuration with disko over ssh in a single command

### Project pages
- https://github.com/nix-community/disko
- https://github.com/numtide/nixos-anywhere
