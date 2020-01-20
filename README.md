# slackware142-container

This is a container based on Slackware v14.2 x64 with

* nginx v1.16.1
* Python 3.8.1
* busybox 1.32.1
* sysvinit v2.88

It is designed as a base container for securely running production quality `flask` micro-services in.

The Python is installed as `/opt/python`, so if you don't want it, just `RUN rm -rf /opt/python` 
The full container is 89Mb, with Python being 75.5Mb of that, meaning `Slackware`+`busybox`+`nginx` is a little over 13Mb.


`pip` works and `installpkg` should be able to install most Slackware v14.2 x64 packages.

Becuase the O/S level utilities are provided by `busybox`, and not all directories that might exist do,
so some of the package post-install scripts may fail.

`pip` is the only `site-package` that is pre-installed. This avoids you having to upgrade any others.
So you should probably start with `RUN pip install --upgrade pip`, just to be sure.
For production use I go with `gunicorn`, so I have `RUN pip install gunicorn` in the app container.


# Running Read-Only

The container is designed to run in read-only, with `--read-only --tmpfs=/ram`, but it should 
be fine without either of these two options.

Note: Unless you tell it otherwise, by default this will mean the access and error logs from `nginx` will go to a ram disk.
This may not be what you want. For production use, I recommend you send the logs, over the network, to a `syslog` server.

Or you can stop it logging to the ram disk with these options
```
	access_log off;
	error_log stderr error;
```

If you are running the contrainer read-only, you may also wish to pre-compile your application Python code when you build
the app container using something like `RUN python -m compileall /app/`


# An Example

[This is](https://github.com/james-stevens/dnsflsk/blob/master/Dockerfile) an exmaple `Docker` file I have used
to run a `flask` micro-service in this container.
