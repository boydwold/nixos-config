# General Purpose Nix Config for macOS + NixOS

## Overview

Nix is a powerful package manager for Linux and Unix systems that ensures reproducible, declarative, and reliable software management.

This repository provides a configuration for a general-purpose development environment that works on macOS, NixOS, or both simultaneously.

It is used daily on various MacBook Pros and x86 PCs, and it also runs as a VM on macOS. Many users have successfully set it up on their systems as well.

Follow the step-by-step instructions below to get started!

## Table of Contents
- [General Purpose Nix Config for macOS + NixOS](#general-purpose-nix-config-for-macos--nixos)
  - [Overview](#overview)
  - [Table of Contents](#table-of-contents)
  - [Layout](#layout)
  - [Features](#features)
  - [Disclaimer](#disclaimer)
  - [Videos](#videos)
    - [macOS](#macos)
      - [Updating dependencies with one command](#updating-dependencies-with-one-command)
      - [Instant Emacs 29 thanks to daemon mode](#instant-emacs-29-thanks-to-daemon-mode)
    - [NixOS](#nixos)
  - [Installing](#installing)
    - [1. Install Nix](#1-install-nix)
    - [2. Initialize a starter template](#2-initialize-a-starter-template)
    - [3. Make apps executable](#3-make-apps-executable)
    - [4. Apply your current user info](#4-apply-your-current-user-info)
    - [5. Decide what packages to install](#5-decide-what-packages-to-install)
    - [6. Review your shell configuration](#6-review-your-shell-configuration)
    - [7. Optional: Setup secrets](#7-optional-setup-secrets)
    - [8. Install configuration](#8-install-configuration)
    - [9. Make changes](#9-make-changes)
    - [For NixOS](#for-nixos)
      - [1. Burn and use the latest ISO](#1-burn-and-use-the-latest-iso)
      - [2. Optional: Setup secrets](#2-optional-setup-secrets)
      - [3. Install configuration](#3-install-configuration)
      - [4. Set user password](#4-set-user-password)
  - [How to Create Secrets](#how-to-create-secrets)
    - [Secrets Example](#secrets-example)
  - [Making Changes](#making-changes)
    - [Development workflow](#development-workflow)
    - [Trying packages](#trying-packages)
  - [Compatibility and Feedback](#compatibility-and-feedback)
    - [Platforms](#platforms)
  - [Appendix](#appendix)
    - [Why Nix Flakes](#why-nix-flakes)
    - [NixOS Components](#nixos-components)

## Layout
```
.
├── apps         # Nix commands used to bootstrap and build configuration
├── hosts        # Host-specific configuration
├── modules      # macOS and nix-darwin, NixOS, and shared configuration
├── overlays     # Drop an overlay file here to apply patches or customizations
├── templates    # Starter versions of this configuration
```

## Features
- **Nix Flakes**: Fully flake-driven configuration with `flake.nix`, eliminating the need for `configuration.nix` and Nix channels.
- **Consistent Environment**: Share configurations seamlessly across Linux and macOS using Nix and Home Manager.
- **macOS Setup**: Declarative configuration for macOS, including UI settings, dock, and App Store applications.
- **Easy Bootstrap**: Simple Nix commands to set up from scratch on both x86 and macOS platforms.
- **Managed Homebrew**: Maintenance-free Homebrew environment managed with `nix-darwin` and `nix-homebrew`.
- **Disk Management**: Declarative disk management using `disko`, removing the need for traditional disk utilities.
- **Secrets Management**: Securely manage secrets with `agenix` for SSH, PGP, Syncthing, and other tools.
- **Fast Emacs**: Up-to-date Emacs with daemon mode for improved performance, supported by community overlays.
- **Integrated Home Manager**: Seamless `home-manager` integration for effortless configuration without additional CLI steps.
- **NixOS Configuration**: Comprehensive NixOS setup with a clean aesthetic and window animations.
- **Nix Overlays**: Automatic loading of Nix overlays by placing files in the `overlays` directory, ideal for patches.
- **Declarative Sync**: Manage Syncthing keys, certificates, and configurations declaratively across all platforms.
- **Emacs Literate Configuration**: Extensive Emacs configuration using literate programming for easy customization.
- **Simplicity and Readability**: Focused on clear and maintainable configurations without excessive fragmentation.
- **Continuous Integration**: Automated weekly updates to ensure the configuration remains up-to-date and stable.

## Disclaimer
Installing Nix on macOS creates a separate volume that may consume several gigabytes.

If this is a concern for you, please reconsider before proceeding.

> [!NOTE]
> You can always [uninstall Nix](https://nix-community.github.io/nix-uninstaller/) later if needed.
> //add declarative

## Videos
### macOS
#### Updating dependencies with one command
[Watch Video](https://github.com/your-repo/assets/your-account/2168d482-6eea-4b51-adc1-2ef1291b6598)

#### Instant Emacs 29 thanks to daemon mode
- **GUI**:

[Watch Video](https://github.com/your-repo/assets/your-account/66001066-2bbf-4492-bc9e-60ea1abeb987)

- **Terminal**:

[Watch Video](https://github.com/your-repo/assets/your-account/d96f59ce-f540-4f14-bc61-6126a74f9f52)

### NixOS
[Watch Video](https://github.com/your-repo/assets/your-account/fa54a87f-5971-41ee-98ce-09be048018b8)

## Installing

> [!NOTE:] This configuration supports both Intel and Apple Silicon Macs.

#### 1. Install Nix
Use the [Determinate Systems Nix Installer](https://zero-to-nix.com/start/install) that will tailor itself to your system.
```sh
curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install
```
That's it! Open a new terminal session and the nix executable should be in your $PATH. To verify that:

```
nix --version
```

#### 2. Initialize a starter template
*Choose one of two options*

**Simplified version without secrets management**
* Ideal for beginners to quickly get started and test out Nix.
* You will need to manually configure applications that require keys, passwords, etc.
* Secrets can be added later if needed.

```sh
mkdir -p nixos-config && cd nixos-config && nix flake init -t github:boydwold/nixos-config#starter
```

**Full version with secrets management**
* Provides a fully declarative configuration with secret management.
* Includes a setup for storing passwords, private keys, and other sensitive information as part of your configuration.

```sh
mkdir -p nixos-config && cd nixos-config && nix flake init -t github:boydwold/nixos-config#starter-with-secrets
```

#### 3. Make apps executable
_this may not be needed when using the Determinate systems installer. Verify_
```sh
find apps/$(uname -m | sed 's/arm64/aarch64/')-darwin -type f \( -name apply -o -name build -o -name build-switch -o -name create-keys -o -name copy-keys -o -name check-keys \) -exec chmod +x {} \;
```

#### 4. Apply your current user info
Run the following command to replace placeholder values with your system properties, username, full name, and email. Your email is only used in the `git` configuration.
```sh
nix run `.#apply`
```
> [!NOTE]
> If you're using a git repository, ensure all files are added to the working tree by running `git add .` before executing the above command.

#### 5. Decide what packages to install
Search for packages on the [official NixOS website](https://search.nixos.org/packages).

**Review these files:**
- `modules/darwin/casks.nix`
- `modules/darwin/packages.nix`
- `modules/shared/packages.nix`

#### 6. Review your shell configuration
Incorporate any customizations from your existing `~/.zshrc` or review the new configuration provided.

**Review these files:**
- `modules/darwin/home-manager.nix`
- `modules/shared/home-manager.nix`

#### 7. Optional: Setup secrets
If you chose the starter with secrets, follow these additional steps.

##### 7a. Create a private GitHub repository to hold your secrets
Create a private repository named `nix-secrets` (or a name of your choice) on GitHub with at least one file (e.g., `README.md`). You'll provide this repository name during installation.

##### 7b. Install keys
Before generating your first build, ensure the necessary keys exist in your `~/.ssh` directory.

| Key Name          | Platform      | Description                                      |
|-------------------|---------------|--------------------------------------------------|
| id_ed25519        | macOS / NixOS | Used to download secrets from GitHub during bootstrap. |
| id_ed25519_agenix | macOS / NixOS | Used to encrypt and decrypt secrets.             |

Run one of the following commands:

**Copy keys from USB drive**
Automatically detects a connected USB drive. Ensure keys are named `id_ed25519` and `id_ed25519_agenix`.
```sh
nix run `.#copy-keys`
```

**Create new keys**
```sh
nix run `.#create-keys`
```
> [!NOTE]
> After creating new keys, add the public key (`id_ed25519.pub`) to your GitHub account:
> ```sh
> cat ~/.ssh/id_ed25519.pub | pbcopy # Copies the key to clipboard
> ```

**Check existing keys**
If you prefer to manage keys manually, verify they are correctly installed.
```sh
nix run `.#check-keys`
```

#### 8. Install configuration
Ensure the build works before deploying the configuration:
```sh
nix run `.#build`
```
> [!NOTE]
> If you're using a git repository, ensure all files are added to the working tree by running `git add .` before executing the above command.

> [!WARNING]
> You may encounter the error `error: Unexpected files in /etc, aborting activation` if existing `/etc/` files conflict with the Nix configuration. The error message will list the conflicting files.
>
> Example:
> ```
> The following files have unrecognized content and would be overwritten:
> 
>   /etc/nix/nix.conf
>   /etc/bashrc
> 
> Please check that these files do not contain critical information, rename them by adding `.before-nix-darwin` to the end, and try again.
> ```
> Backup and move the conflicting files or adjust your Nix configuration accordingly before proceeding.

#### 9. Make changes
Apply the configuration changes to your system:
```sh
nix run `.#build-switch`
```
> [!CAUTION]
> This command will replace your `~/.zshrc` with the provided `zsh` configuration. Ensure this is acceptable before proceeding.

### For NixOS
This configuration supports both `x86_64` and `aarch64` platforms.

#### 1. Burn and use the latest ISO
Download and create a bootable USB using the [minimal ISO image](https://nixos.org/download.html), or set up a new VM using the ISO as the base image. Boot from the installer.

> If you're setting up a VM on an Apple Silicon Mac, choose the [64-bit ARM](https://channels.nixos.org/nixos-23.05/latest-nixos-minimal-aarch64-linux.iso) ISO.

**Quick Links:**
- [64-bit Intel/AMD](https://channels.nixos.org/nixos-23.05/latest-nixos-minimal-x86_64-linux.iso)
- [64-bit ARM](https://channels.nixos.org/nixos-23.05/latest-nixos-minimal-aarch64-linux.iso)

#### 2. Optional: Setup secrets
If you chose the starter with secrets, follow these additional steps.

##### 2a. Create a private GitHub repository to hold your secrets
Create a private repository named `nix-secrets` (or a name of your choice) on GitHub with at least one file (e.g., `README.md`). You'll provide this repository name during installation.

##### 2b. Install keys
Ensure the necessary keys exist in your `~/.ssh` directory before generating your first build.

| Key Name          | Platform      | Description                                      |
|-------------------|---------------|--------------------------------------------------|
| id_ed25519        | macOS / NixOS | Used to download secrets from GitHub during bootstrap. |
| id_ed25519_agenix | macOS / NixOS | Used to encrypt and decrypt secrets.             |

Run one of the following commands:

**Copy keys from USB drive**
Automatically detects a connected USB drive. Ensure keys are named `id_ed25519` and `id_ed25519_agenix`.
```sh
sudo nix run .#copy-keys
```

**Create new keys**
```sh
sudo nix run .#create-keys
```

**Check existing keys**
If you prefer to manage keys manually, verify they are correctly installed.
```sh
sudo nix run .#check-keys
```

#### 3. Install configuration

##### Pick your template
> [!IMPORTANT]
> For systems with Nvidia graphics cards, select the `nomodeset` option when booting the installer to avoid a blank screen.

> [!CAUTION]
> Running this installation will reformat your drive to the `ext4` filesystem.

**Simple**
* Ideal for beginners to quickly set up and test Nix.
* Requires manual configuration of applications that need keys or passwords.
* Secrets can be added later if needed.
```sh
sudo nix run .#install
```

**With secrets**
* Provides a fully declarative configuration with secret management.
* Includes setup for storing passwords, private keys, and other sensitive information as part of your configuration.
```sh
sudo nix run .#install-with-secrets
```

#### 4. Set user password
On the first boot at the login screen:
1. Use the shortcut `Ctrl-Alt-F2` (or `Fn-Ctrl-Option-F2` on a Mac) to switch to a terminal session.
2. Log in as `root` using the password created during installation.
3. Set the user password with:
    ```sh
    passwd <your-username>
    ```
4. Return to the login screen with:
    ```sh
    Ctrl-Alt-F7
    ```

## How to Create Secrets
To create a new secret `secret.age`, first [create a `secrets.nix` file](https://github.com/ryantm/agenix#tutorial) at the root of your `nix-secrets` repository. Use the following template:

> [!NOTE]
> The `secrets.nix` file configures `agenix` with the appropriate keys for your secrets. It serves as the configuration file for `agenix` and is not part of your system configuration.

**secrets.nix**
```nix
let
  user1 = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIL0idNvgGiucWgup/mP78zyC23uFjYq0evcWdjGQUaBH";
  users = [ user1 ];

  system1 = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPJDyIr/FSz1cJdcoW69R+NrWzwGK/+3gJpqD1t8L2zE";
  systems = [ system1 ];
in
{
  "secret.age".publicKeys = [ user1 system1 ];
}
```
Replace `user1` and `system1` with your actual public keys.

Now that `agenix` is configured with your `secrets.nix`, create your first secret:

```sh
EDITOR=vim nix run github:ryantm/agenix -- -e secret.age
```

This command opens an editor to input, encrypt, and save your secret.

The command will:
- Look up the public key for `secret.age` defined in `secrets.nix`.
- Check for the corresponding private key in `~/.ssh/`.

> To specify a different SSH key path, use the `-i` flag with the path to your `id_ed25519` key.

Enter your secret in the editor, save the file, and commit it to your `nix-secrets` repository.

You will now have two files: `secrets.nix` and your encrypted secret file `secret.age`.

### Secrets Example
To create a new secret for your GitHub SSH key:

1. Navigate to your `nix-secrets` repository directory.
2. Ensure `secrets.nix` exists.
3. Run:
    ```sh
    EDITOR=vim nix run github:ryantm/agenix -- -e github-ssh-key.age
    ```
4. In the `vim` editor:
    - Enter insert mode by pressing `i`.
    - Paste your GitHub SSH key.
    - Press `Esc`, then type `:w` to save and exit.

5. Update `secrets.nix` to include your new secret:

**secrets.nix**
```nix
let
  user1 = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIL0idNvgGiucWgup/mP78zyC23uFjYq0evcWdjGQUaBH";
  users = [ user1 ];
  systems = [ ];
in
{
  "github-ssh-key.age".publicKeys = [ user1 ];
}
```

6. Commit the changes to your `nix-secrets` repository.
7. Update your Nix configuration by running:
    ```sh
    nix flake update
    ```

Your secret is now ready to use. It will create a symlink to a decrypted file in the Nix Store that mirrors your original file.

## Making Changes
With Nix, system changes involve:
- Editing your system configuration.
- Building the [system closure](https://zero-to-nix.com/concepts/closures).
- Creating a [new generation](https://nixos.wiki/wiki/Terms_and_Definitions_in_Nix_Project#generation) based on this closure and switching to it.

This process is streamlined with the `build-switch` command.

### Development Workflow
The typical workflow for managing your environment:
1. Make changes to the configuration files.
2. Run:
    ```sh
    nix run .#build-switch
    ```
3. Observe Nix, `nix-darwin`, `home-manager`, etc., apply the changes.
4. Enjoy your updated declarative environment.

### Trying Packages
To quickly try a package without installing it, use:
```sh
nix shell nixpkgs#hello
```
Replace `hello` with the desired package name from [nixpkgs](https://search.nixos.org/packages).

## Compatibility and Feedback

### Platforms
This configuration has been tested and works on the following platforms:
- Newer M1/M2/M3 Apple Silicon Macs
- Older x86_64 (Intel) Macs
- Bare metal x86_64 PCs
- NixOS VMs inside VMWare on macOS
- macOS Sonoma VMs inside Parallels on macOS

## Appendix

### Why Nix Flakes
**Advantages of using flakes over traditional methods like `nix-env` and Nix channels:**
- **Modern Workflow**: Flakes operate similarly to package managers like `npm`, `cargo`, or `poetry`, making them familiar to many developers.
- **Encapsulation**: Flakes bundle project dependencies, Nix expressions, applications, and configurations into a single file, enhancing manageability.
- **Precise Versioning**: Unlike channels that lock all packages to a single `nixpkgs` version, flakes lock each package individually, allowing for more precise and manageable updates.
- **Growing Ecosystem**: Flakes are supported by a thriving ecosystem, ensuring future compatibility and access to a wide range of tools and resources.

### NixOS Components

| Component                | Description                        | 
|--------------------------|------------------------------------|
| **Window Manager**       | Xorg + bspwm                       |
| **Terminal Emulator**    | Alacritty                          |
| **Bar**                  | Polybar                            |
| **Application Launcher** | Rofi                               |
| **Notification Daemon**  | Dunst                              |
| **Display Manager**      | LightDM                            |
| **File Manager**         | Thunar                             |
| **Text Editor**          | Emacs (daemon mode)                |
| **Media Player**         | Cider                              |
| **Image Viewer**         | Feh                                |
| **Screenshot Software**  | Flameshot                          |

