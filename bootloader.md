# Bootloader

So what exactly is a bootloader? It's the software that runs before Linux and is responsible for setting the system up far enough to load the AP (Application Processor) kernel.

Here's the chain of operations when you power on a device. It goes to the Boot ROM, which will go to an early bootloader, which will run another bootloader, which will then load the Linux kernel and establish userspace.

## Android Verified Boot

AVB is a chain-of-trust system that runs inside the bootloader, before the Android/Linux kernel starts. The bootloader has a trusted root, like a trusted OEM public key. It uses that to verify `vbmeta`, which holds hash trees, root hashes,
and signatures of partitions like system, vendor, and boot.

## dm-verity

Let's say that `system` is huge, with multiple 4kb blocks. Since it'll be expensive for the bootloader to hash the entire partition on every boot, Android uses a Merkle hash tree. A Merkle hash tree
is simply a tree of hashes where the leafs are hashes of individual blocks, and neighboring blocks are hashed together, going all the way up to a root hash. The important thing is that this root hash has to be
ensured to have not been modified, by using a tamper-evident mechanism. If this can be changed by an attacker, dm-verity will blindly verify that the attacker-modified block is safe.

How does AVB and dm-verity work together? AVB securely establishes the trusted root has, and dm-verity then uses that root hash at runtime. For exmaple, let's say Linux reads a filesystem block. Android wants /system/bin/foo.txt.
The filesystem them requests block 1337. dm-verity intercepts it and computes hash(block 1337). It uses that hash to walk up the Merkle tree and checks if the hash it computed equals the root has. If it does, dm-verity will return
the data. If not, then there is a verification error.

So AVB establishes what should be trusted, while dm-verity continuously enforces that trust while the OS runs.
