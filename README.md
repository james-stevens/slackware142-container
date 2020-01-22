# slackware142-container

This is a container based on Slackware v14.2 x64 with

* nginx v1.16.1
* Python 3.8.1
* busybox 1.32.1
* sysvinit v2.89

It is designed as a base container for securely running production quality Python `flask` micro-services in.

The Python is installed as `/opt/python`, so if you don't want it, just `RUN rm -rf /opt/python` 
The full container is 89Mb, with Python being 75.5Mb of that, meaning `Slackware`+`busybox`+`nginx` is a little over 13Mb.

If you **really** want to make your container smaller, you could remove all the `pyc` files and only compile those that you are going
to use. The PYCs are 38Mb, so this could save quite a lot of space and reduce the start-up time of your container.
Or, if you aren't running your container read-only, you could remove the PYCs, then just let python re-create them on the fly.

You could also remove all the python libraries that you do not use. I've supplied the full standard install, because
I don't know what you might want to do, but you can get Python down to about 5Mb, if you remove everything you don't want. Just add a
`RUN rm -rf opt/python/lib/python3.8/<blah>` for each one you don't want.

I have already removed the `test` library, as this is intended for production use, but a number of
individual libraries also have their own test code, that could probably be removed.


`pip` works and `installpkg` should be able to install most Slackware v14.2 x64 packages.

Because the O/S level utilities are provided by `busybox`, and not all directories that might exist do,
so some of the package post-install scripts may fail.

`pip` is the only `site-package` that is pre-installed. This avoids you having to upgrade any others.
So you should probably start with `RUN pip install --upgrade pip`, just to be sure.
For production use I go with `gunicorn`, so I have `RUN pip install gunicorn` in the app container.



# Running Read-Only

The container is designed to run in read-only, with `--read-only --tmpfs=/ram`, but it should 
be fine without these two options.

Note: Unless you tell it otherwise, by default this will mean the access and error logs from `nginx` will go to a ram disk.
This may not be what you want. For production use, I recommend you send the logs, over the network, to a `syslog` server.

Or you can stop it logging all-together with these options
```
	access_log off;
	error_log stderr error;
```

If you are running the container read-only, you should probably pre-compile your application Python code when you build
the app container using something like `RUN python -m compileall /app/`


# An Example

[This is an example](https://github.com/james-stevens/dnsflsk/blob/master/Dockerfile)  `Docker` file I have used
to run a `flask` micro-service using this container.


# Rebuilding the Binaries

I have provided the config file I used to build `busybox` (`busybox-config`). You can use this to build a newer version, or change the command
line utilities that are available.


To rebuild Python, I used 
```
$ tar xvf Python-3.8.1.tar.xz
$ cd Python-3.8.1
$ ./configure --prefix=/opt/python --enable-optimizations --disable-profiling
$ make
$ make install
```

This installed Python v3.8.1 into `/opt/python` on my dev server. This will not interfere with the Python you may 
have already installed in the operating system. If you are at all concerned, I recommend you run the build in a container.


If you want to rebuild the `nginx` binary, the `configure` command I used is in the file `nginx-configure`.
So download the `nginx` package, extract it, go into the directory and run that configure script -
or use different options, if you need them.

