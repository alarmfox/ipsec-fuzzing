# Progetto software security
Progetto basato su syzkaller per il fuzzing del kernel Linux.

Il progetto cerca di effettuare il fuzzing dell'implementazione 
di IPsec e delle strutture associate in Linux.

Nel file `DOC-Capasso-m63001498.pdf` è contenuta la documentazione 
del progetto con annesse istruzioni per l'esecuzione. Il setup 
funziona su Linux utilizzando QEMU e KVM come sistema di virtualizzazione.

## Compilazione syzkaller
Per compilare syzkaller è consigliato attivare l'ambiente di build 
(richiede Docker installato) attraverso lo script in 
`./syzkaller/tools/syz-env`.

```sh 
$ ./tools/syz-env
gcr.io/syzkaller/env:latest
$ syz-env: make
```

## Compilazione kernel
Prima di compilare il kernel, bisogna configurarlo con l'instrumentazione. 
Una lista dei parametri fondamentali è:

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

È possibile aggiungere altri parametri per l'instrumentazione per rendere l'analisi più
precisa (la lista completa è disponibile 
[qui](https://github.com/google/syzkaller/blob/master/docs/linux/kernel_configs.md)).
Infine è possibile compilare con:
```sh 
make -j `nproc`
```

## Generazione immagine
L'immagine può essere generata a partire dal programma `debootstrap` e usando 
lo script presente in `./syzkaller/tools/create-image.sh` (bisogna usare sudo). 
Per un'immagine completa:
```sh 
$ sudo ./create-image.sh --feature full
```
Lo script genera un'immagine Debian e una coppia di chiavi SSH.

## Esecuzione
Per eseguire il programma, bisogna definire un file come quello di seguito.
in cui bisogna sostituire a $SYZKALLER, $KERNEL e $IMAGE i path alle cartelle giuste.
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

Infine, è possibile lanciare syz-manager con il comando:
```sh 
./syzkaller/syzkaller/bin/syz-manager -config syzkaller/config.json

```

