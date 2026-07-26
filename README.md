# WSL Kernel Builder

<img width="1235" height="628" alt="image" src="https://github.com/user-attachments/assets/8504ecca-1251-4fcf-a6ec-f64eb8c2bccd" />

Using GitHub Actions workflow to builds a custom Linux kernel for WSL2 by merging latest linux-stable source with Microsoft's WSL2 kernel config (arch/x86/configs/config-wsl from microsoft/WSL2-Linux-Kernel).

This lets you build a kernel that tracks upstream linux-stable more closely than Microsoft's own WSL2 kernel branch, while still using Microsoft's WSL-specific config (Hyper-V drivers, 9p, virtio, etc. enabled; desktop-only drivers like nouveau/amdgpu disabled).

## How it works
Clones a branch of linux-stable (default: linux-rolling-stable) as the kernel source.
Clones a branch of microsoft/WSL2-Linux-Kernel (default: linux-msft-wsl-6.18.y) and copies just arch/x86/configs/ out of it.
Uses config-wsl as the base .config, running olddefconfig to fill in any new options introduced since Microsoft's tree diverged.
Builds bzImage + modules (optionally .deb packages).
Uploads the compiled kernel as a workflow artifact.

If the MSFT config can't be fetched for any reason, the build falls back to plain defconfig — this is not recommended for WSL use (it builds the full desktop driver tree and produces a much larger, unstripped kernel) but keeps the workflow from failing outright.

## Running it

From the Actions tab, select Merge and Build Kernel → Run workflow, and set:

Input	Default	Description
1. msft_branch	linux-msft-wsl-6.18.y (or the latest wsl branch) Branch of microsoft/WSL2-Linux-Kernel to pull config-wsl from
2. linux_branch	linux-rolling-stable (this is the latest linux kernel, if you want to boot other kernel rename this) Branch of git.kernel.org/.../linux.git to build
3. build_deb	false	Set to true to also produce installable .deb packages via make bindeb-pkg

## Output

On a successful run, download the kernel-build-artifacts artifact from the run summary page. It contains whichever of the following were produced:

- bzImage — the bootable kernel image (arch/x86/boot/bzImage in-tree)
- vmlinux — the uncompressed ELF kernel (useful for debugging/symbols)
- linux-image-*.deb, linux-headers-*.deb — if build_deb was true

The "Locate compiled kernel" step in the run log prints exact in-tree paths, file sizes, and the resolved kernelrelease string, so you can confirm what actually got built without downloading the artifact first.

## Installing the built kernel in WSL

Open WSL app and choose the bzimage from the zip 

<img width="1286" height="793" alt="image" src="https://github.com/user-attachments/assets/5bf36085-3728-4789-afdd-af346032d72a" />
