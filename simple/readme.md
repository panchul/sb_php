
Some example of a server I looked up. It is incomplete, missing
whole bunch of best practices as far as /etc/... configs, logs, pid lock, etc.
But it is easy to play with.


# Server

To start:

    $ php server.php

Make a request (in another terminal):

    $ nc -u 127.0.0.1 10000
    Hello, world!
    Uryyb, jbeyq!
    
# Turning it into a service

Create a file called /etc/systemd/system/rot13.service:

    [Unit]
    Description=ROT13 demo service
    After=network.target
    StartLimitIntervalSec=0[Service]
    Type=simple
    Restart=always
    RestartSec=1
    User=ubuntu
    ExecStart=/usr/bin/env php /path/to/server.php

    [Install]
    WantedBy=multi-user.target

Now you can start it as a server:

    $ systemctl start rot13

And automatically get it to start on boot:

    $ systemctl enable rot13

# more tweaks

Order of the services. E.g. starting after mysql:

    After=mysqld.service

Restarting on exit

    Restart=always

By default, systemd attempts a restart after 100ms. To restart in 1 sec:

    RestartSec=1

By default, when you configure `Restart=always`, systemd gives up restarting your
service if it fails to start more than 5 times within a 10 seconds interval.
There are two [Unit] configuration options responsible for this:

    StartLimitBurst=5
    StartLimitIntervalSec=10

The RestartSec directive also has an impact on the outcome: if you set it
to restart after 3 seconds, then you can never reach 5 failed retries
within 10 seconds.

The simple fix that always works is to set `StartLimitIntervalSec=0`.
This way, systemd will attempt to restart your service forever.

As an alternative, you can leave the default settings, and ask systemd 
to restart your server if the start limit is reached,
using `StartLimitAction=reboot`.
