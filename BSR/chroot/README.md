To trap an application in a changed root, we need to place it somewhere in the file system, for example: 

```console
lab-sec:~# mkdir /chroot
```

Now, let's try to trap a simple application in this changed root, e.g., `bash`:

```console
lab-sec:~# chroot /chroot/ bash
chroot: failed to run command ‘bash’: No such file or directory
```

