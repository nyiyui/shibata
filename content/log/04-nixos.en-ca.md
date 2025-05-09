+++
title = "Setting up a NixOS Machine with an Ephermeral Filesystem and Secure Boot"
date = "2025-05-09T14:17:38-05:00"
draft = true
+++

## Introduction

The sections below assume familiarity with Nixpkgs and NixOS modules.

## Declarative Disk Partitioning

[Disko](https://github.com/nix-community/disko) is a CLI tool and NixOS module that configures your disk partitions.
Disko replaces the [partitioning step of a normal install](https://nixos.wiki/wiki/NixOS_Installation_Guide#Partitioning) with the following:
1. Make a `disko-config.nix` with the partition settings for the disks you want to change
2. Run `disko --mode destroy,format,mount ./disko-config.nix` to wipe, format, and then mount your disk to `/mnt`
  - Note that mountpoints managed by Impermanence and other modules have to be mounted manually.

Then, you can run `nixos-install` as usual.

## Ephermeral Root Filesystem

TODO: motivation
- random garbage in ~
- some config stored in ~, and not in NixOS config?

[Impermanence](https://github.com/nix-community/impermanence) is a set of NixOS modules that bind mounts directories from a "presistence" partition to paths such as `/etc/ssh`, `/etc/NetworkManager/system-connections`, `/home/nyiyui/.ssh`.

Notice that Impermanence only does bind mounts. We need to find a way to provide a fresh root filesystem at every boot.
There are a few options to accomplish this:
- For my devices, I use [BTRFS subvolumes](https://btrfs.readthedocs.io/en/latest/Subvolumes.html) where a new subvolume is created on every boot.
  - [Impermanence documentation](https://github.com/nix-community/impermanence?tab=readme-ov-file#system-setup)
- Mounting a [tmpfs](https://www.kernel.org/doc/Documentation/filesystems/tmpfs.txt) on `/` (just like early-boot `/dev`).
  - [Elis Hirwing's blog post on using tmpfs as root](https://elis.nu/blog/2020/05/nixos-tmpfs-as-root/)
  - [Will Bush's blog post with a detailed guide on using tmpfs with LUKS](https://willbush.dev/blog/impermanent-nixos/)
- Making a new logical volume (if using an LVM-based drive) for the root directory.
- Making a ZFS snapshot and then clearing the filesystem on each boot.
  - [Graham Christensen's blog post on using ZFS snapshots](https://grahamc.com/blog/erase-your-darlings/)

I've added my config below for reference, which creates a new BTRFS subvolume on every boot.

```nix
{
  specialArgs,
  pkgs,
  lib,
  ...
}:
{
  imports = [ specialArgs.impermanence.nixosModules.impermanence ];
  environment.persistence."/persist" = {
    hideMounts = true;
    directories = [
      "/var/log"
      "/var/lib"
      "/etc/secureboot"
      "/etc/NetworkManager/system-connections"
      "/etc/ssh"
    ];
    files = [
      "/etc/machine-id"
    ];
    users.nyiyui = {
      directories = [
        {
          directory = ".ssh";
          mode = "0700";
        }
        ".local/share/direnv"
        ".local/share/fish"
        ".local/share/nvim"
        ".mozilla/firefox"
        ".thunderbird"
        ".config/syncthing"
        ".config/sayonara"
      ];
    };
  };

  boot.initrd.systemd.services.swap-old-root = {
    # script modified from Impermanence documentation
    description = "move old root to /old_roots and make new root at /root";
    wantedBy = [ "initrd.target" ];
    before = [ "sysroot.mount" ];
    serviceConfig.Type = "oneshot";
    script = ''
      mkdir /btrfs_tmp
      mount /dev/mapper/crypted /btrfs_tmp
      if [[ -e /btrfs_tmp/root ]]; then
          mkdir -p /btrfs_tmp/old_roots
          timestamp=$(date --date="@$(stat -c %Y /btrfs_tmp/root)" "+%Y-%m-%-d_%H:%M:%S")
          mv /btrfs_tmp/root "/btrfs_tmp/old_roots/$timestamp"
      fi

      delete_subvolume_recursively() {
          IFS=$'\n'
          for i in $(btrfs subvolume list -o "$1" | cut -f 9- -d ' '); do
              delete_subvolume_recursively "/btrfs_tmp/$i"
          done
          btrfs subvolume delete "$1"
      }

      for i in $(find /btrfs_tmp/old_roots/ -maxdepth 1 -mtime +30); do
          delete_subvolume_recursively "$i"
      done

      btrfs subvolume create /btrfs_tmp/root
      umount /btrfs_tmp
    '';
  };
}
```

## Automatic Upgrades with Flakes

One nice thing about Nix flakes is the ability to directly specify remote repositories (as shown in the Disko section above).
Since we can get Nix to use personal access tokens when accessing GitHub etc, once we have everything set up, we can just `nixos-rebuild switch --flake github:nyiyui/nixos-config#hostname` to build.

1. Make a personal access token with the `Read-only` permission for repository contents.
2. Using [agenix](https://github.com/ryantm/agenix) or similar, add the following config files:
  - In `nix.conf`: `!include /run/agenix/nix-conf-access-token`
  - In `/run/agenix/nix-conf-access-token`: `access-tokens = github.com=<insert_github_token_here>`
3. Now, when fetching your NixOS config repository, Nix will automatically use your access token.

- make directory with impermanence, disko, hardware-config, etc
```
sudo nix --experimental-features "nix-command flakes" run github:nix-community/disko -- --mode destroy,format,mount ./inaho/disko-config.nix 
```

```
systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+7
```
