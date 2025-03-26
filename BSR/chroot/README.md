# CHROOT
To trap an application in a changed root, we need to place it somewhere in the file system, for example: 

```console
lab-sec:~# mkdir /chroot
```


## BASH
Now, let's try to trap a simple application in this changed root, e.g., `bash`:

```console
lab-sec:~# chroot /chroot/ bash
chroot: failed to run command ‘bash’: No such file or directory
```

Why? Let's investigate!
The first thing we will check is where our regular bash is.

```console
lab-sec:~# which bash
/usr/bin/bash
```

Now let's have a look inside the changed root directory.
```console
lab-sec:~# ls /chroot/usr/bin/
ls: cannot access '/chroot/usr/bin/': No such file or directory
```

And there is our problem! There isn't even a directory `/chroot/usr/bin/`,
let alone a `bash` executable in it. We need to add it there.

Now, to add files into our changed root, we will use the command `cp` with a preset of options:
```shell
cp -nLv --parents ... /chroot/
```
It hAs some useful options selected. Use `man` to check what each of them does.

Let's copy `bash` into our sandbox:
```console
lab-sec:~# cp -nvL --parents $(which bash) /chroot/
/usr -> /chroot/usr
/usr/bin -> /chroot/usr/bin
'/usr/bin/bash' -> '/chroot/usr/bin/bash'
```

> [!NOTE]
> The syntax `$( <command-inside> )` let's us evaluate the `<command-inside>` and paste the result
> into another command. In this case we paste the result of `which bash` into our `cp`.
 
Will it run now? Let's find out.

```console
lab-sec:~# chroot /chroot/ bash
chroot: failed to run command ‘bash’: No such file or directory
```

What could it be this time?

We copied the executable, yes, but after a bit of thought we should figure out,
that every application will need not only the executable file, but also _libraries_.
Let's check if that's indeed the case for `bash`:

```console
lab-sec:~# ldd $(which bash)
        linux-vdso.so.1 (0x00007ffe53de6000)
        libreadline.so.7 => /lib64/libreadline.so.7 (0x00007f80ece00000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f80ed5de000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f80eca00000)
        libtinfo.so.6 => /lib64/libtinfo.so.6 (0x00007f80ed5ad000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f80ed5fc000)
```

We can see a list of dynamic dependencies of `bash` here.
> [!NOTE]
> You will note, that the first one, `linux-vdso.so.1` doesn't have a path, just a name. 
> That's because it's a virtual library, _virtual dynamic system objects_, and doesn't actually exist in the file system.

Now, we will copy those libraries into the changed root. Because we are _economically_ lazy, we won't copy them one-by-one.
Instead, we will `cut` the names from the output of `ldd`. My proposition is to first cut the front part out:

```console
lab-sec:~# ldd $(which bash) | cut -d'>' -f2
        linux-vdso.so.1 (0x00007fff0b8d0000)
 /lib64/libreadline.so.7 (0x00007f97f5000000)
 /lib64/libdl.so.2 (0x00007f97f576f000)
 /lib64/libc.so.6 (0x00007f97f4c00000)
 /lib64/libtinfo.so.6 (0x00007f97f573e000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f97f578d000)
```

And then cut the back bit:
```console
lab-sec:~# ldd $(which bash) | cut -d'>' -f2
        linux-vdso.so.1 
 /lib64/libreadline.so.7 
 /lib64/libdl.so.2 
 /lib64/libc.so.6 
 /lib64/libtinfo.so.6 
        /lib64/ld-linux-x86-64.so.2 
```

Now, we can use this list in our `cp`:

```console
lab-sec:~# cp -nvL --parents $(ldd $(which bash) | cut -d'>' -f2 | cut -d'(' -f1) /chroot/
cp: cannot stat 'linux-vdso.so.1': No such file or directory
/lib64 -> /chroot/lib64
'/lib64/libreadline.so.7' -> '/chroot/lib64/libreadline.so.7'
'/lib64/libdl.so.2' -> '/chroot/lib64/libdl.so.2'
'/lib64/libc.so.6' -> '/chroot/lib64/libc.so.6'
'/lib64/libtinfo.so.6' -> '/chroot/lib64/libtinfo.so.6'
'/lib64/ld-linux-x86-64.so.2' -> '/chroot/lib64/ld-linux-x86-64.so.2'
```

> [!NOTE]
> The error occurs, because—as explained before—`linux-vdso.so.1` doesn't exist in the files system.
> We can ignore it.

Finally, we will try to run `bash` in the sandbox again:
```console
lab-sec:~# chroot /chroot/ bash
bash-4.4#
```

Success!

## ID

After running `bash`, I'd like you to execute the command `id` on a user called `wwwrun`
in your changed root sandbox.

The command looks like this:
```console
lab-sec:~# id wwwrun
uid=474(wwwrun) gid=461(wwwrun) groups=462(www),461(wwwrun)
```
It shows the user id and the groups to which the user belongs.

<details>
<summary>Use the same method as for `bash` and try to run `id`.</summary>

```console
lab-sec:~# cp -nvL --parents $(which id) $(ldd $(which id) | cut -d'>' -f2 | cut -d'(' -f1) /chroot/
lab-sec:~# chroot /chroot/ bash
bash-4.4# id wwwrun
```
</details>

Did it work? What do you think has happened? And most importantly, **how do we fix it?**

> [!TIP]
> Ask yourself, where does `id` get the information about users from?

You can use system call tracer, `strace` to inspect what files `id` tried accessing.
Add them to the sandbox.

Does it work now?

Confirm it by running `id wwwrun` directly using chroot, without relying on bash.