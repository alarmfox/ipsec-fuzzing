# IPSec fuzzing
Project based on syzkaller for fuzzing the Linux kernel.

The project aims to perform fuzzing on the implementation of IPsec and its associated structures in Linux.

## Building syzkaller
To compile syzkaller, it is recommended to activate the build environment (requires Docker installed) using the script in `-/syzkaller/tools/syz-env`.

```sh 
$ ./tools/syz-env
gcr.io/syzkaller/env:latest
$ syz-env: make
```

## Building the kernel
Before compiling the kernel, it must be configured with instrumentation.
A list of essential parameters is:

```
# Coverage collection.
CONFIG_KCOV=y

# Debug info for symbolization.
CONFIG_DEBUG_INFO=y

# Memory bug detector
CONFIG_KASAN=y
CONFIG_KASAN_INLINE=y

# Required for Debian Stretch
CONFIG_CONFIGFS_FS=y
CONFIG_SECURITYFS=y
```

Other instrumentation parameters can be added to make the analysis more precise (the full list is available [here](https://github.com/google/syzkaller/blob/master/docs/linux/kernel_configs.md)
Finally, you can compile with:
```sh 
make -j `nproc`
```

## Create an image
The image can be generated using the debootstrap program and the script in ./syzkaller/tools/create-image.sh (sudo is required).
The script generates a Debian image and a pair of SSH keys. For a full image:
```sh 
$ sudo ./create-image.sh --feature full
```

## Execution
To run the program, you need to create a configuration file like the one below, replacing $SYZKALLER, $KERNEL, and $IMAGE with the correct paths.
```json
{
        "target": "linux/amd64",
        "http": "127.0.0.1:56741",
        "workdir": "workdir",
        "kernel_obj": "$KERNEL",
        "image": "$IMAGE/bullseye.img",
        "sshkey": "$IMAGE/bullseye.id_rsa",
        "syzkaller": "$SYZKALLER",
        "procs": 3,
        "type": "qemu",
        "vm": {
                "count": 6,
                "kernel": "$KERNEL/arch/x86/boot/bzImage",
                "cpu": 1,
                "mem": 1024
        }
}
```

Finally, syzkaller can be run with:
```sh 
./syzkaller/syzkaller/bin/syz-manager -config syzkaller/config.json

```

