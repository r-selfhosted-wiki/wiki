---
Title: Wire Guard
tags: [normal]
---

Wireguard is a secure VPN tunnel that aims to provide a VPN that is easy to use, fast, and with low overhead.

It is cross platform, but it is part of the linux kernel by default with only the need of userland tools to configure and deploy it.

#### Preface

From now on, we are going to assume that we are working on linux to configure wireguard with one server and at least one client.

This guide assumes that this configuration is being performed as root or the super-user. For your distribution this may require you to prefix commands with 'sudo'.

#### Installation

You can find more details about installing wireguard on your own operating system here: https://www.wireguard.com/install/ . Please complete installation for both the server and client machine.

##### Make the keys

The first step after installing wireguard for your distribution is to generate keys. We should do this for the server first, but this will be the same for clients as well.

    cd /etc/wireguard && wg genkey | tee private.key | wg pubkey > public.key

You should now have a **public.key** and **private.key** file in */etc/wireguard/*.

>It is important to make sure your private key stays private. No private key should ever leave the machine it was generated on. The client and server will only need the public keys for eachother. If you are using the private keys for a client on a server, or vice-versa, you are doing something wrong.

#### Server configuration

Since this is the server, we need to make a new configuration file for it in */etc/wireguard/*. We will call it **wg0.conf**. The full path should end up being */etc/wireguard/wg0.conf*.

>Please use your own private key where appropriate. You can view the contents of a text file from the command line with *cat* (EG: **cat /path/to/text.file** ).

You can change the **Address** field to use a different address space ( EG: 192.168.x.1 ) if you wish. If your server or clients are already using private IP space on a LAN, **use something different**.

    [Interface]
    ## Private IP address for the server to use
    Address = 10.0.0.1/24
    ## When WG is shutdown, flushes current running configuration to disk. Any changes made to the configuration before shutdown will be forgotten.
    SaveConfig = true
    ## The port WG will listen on for incoming connections. 51194 is the default
    ListenPort = 51194
    ## The server's private key. Not a file path, use the key file contents
    PrivateKey = PRIVATEKEY

After this is done we should be able to start the VPN tunnel and make sure it's enabled.

>Please consult the documentation for your linux distribution for enabling/starting services. This guide is using system tools installed on debian and debian based distributions.

##### *Debian*

    systemctl enable wg-quick@wg0 && systemctl start wg-quick@wg0

That should be it for the server portion.

#### Client Configuration

The client will need keys too. Use the same procedure to [make keys]({{< relref "#make-the-keys" >}} "make keys") for the client as we did for the server.

Once that is done we need to create a client configuration. Lets make **wg0-client.conf** in */etc/wireguard/*. Full path should be */etc/wireguard/wg0-client.conf*.

You will need to choose a unique IP for the client. Everything should be the same as the server's private IP except the last octet.

    [Interface]
    ## This Desktop/client's private key ##
    PrivateKey = CLIENTPRIVATEKEY

    ## Client ip address ##
    Address = 10.0.0.CLIENTOCTET/32

    [Peer]
    ## WG server public key ##
    PublicKey = WGSERVERPUBLICKEY

    ## set ACL ##
    ## Uncomment the next line to use VPN for VPN connections only
    # AllowedIPs = 10.0.0.0/24
    ## If you want to use the VPN for ALL network traffic, uncomment the following line instead
    # AllowedIPs = 0.0.0.0/0

    ## Your WG server's PUBLIC IPv4/IPv6 address and port ##
    Endpoint = WGSERVERPUBLICIP:51194

    ##  Key connection alive ##
    PersistentKeepalive = 20

This should be all you need for configuring the client end connection. We will need the private client IP you chose and the public client key in a bit.

As with the server, we need to enable the wireguard client service. We don't start it yet because the server still doesn't know about this client.

##### *Debian*

    systemctl enable wg-quick@wg0-client

#### Configuring the client as a peer

Back on your server, we need to add the client so the server will accept the client connection. This is where your client private IP and public key will be used.

Run the following command on the WG server to add the client.

    wg set wg0 peer CLIENTPUBLICKEY allowed-ips CLIENTPRIVATEIP/32

You should not need to restart the wireguard service

Lets start the WG client service on the client:

##### *Debian*

    systemctl start wg-quick@wg0-client

To check that it is working, ping the wg server on its private IP.


    $ ping -c 1 10.0.0.1
    PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data.
    64 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=0.071 ms

    --- 10.0.0.1 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.071/0.071/0.071/0.000 ms

If you consider your client internet connection stable, this next step may not be nessecary. You can consider yourself done if you wish.

THE END ( maybe )

#### Wireguard watchdog ( OPTIONAL )

Next we are going to setup a small cron job that will ping the WG server on its private IP to make sure the connection is still intact. If the connection fails, the tunnel will be restarted.

You can put this script anywhere, but I usually choose to put it in */usr/local/scripts/*.

    mkdir /usr/local/scripts

Now for the script. I use **wg-watch.sh**. Lets assume you are going to be using */usr/local/scripts/wg-watch.sh* for the **full file path**.


    #!/usr/bin/bash
    # Modified from https://mullvad.net/en/help/running-wireguard-router/
    # ping VPN gateway to test for connection
    # if no contact, restart!

    PING=/usr/bin/ping
    ## DEBIAN
    SERVICE=/usr/sbin/service

    tries=0
    while [[ $tries -lt 3 ]]
    do
        if $PING -c 1 10.0.0.1
        then
                echo "wg works"
                exit 0
        fi
        echo "wg fail"
        tries=$((tries+1))
    done
    echo "wg failed 3 times - restarting tunnel"
    ## DEBIAN
    $SERVICE wg-quick@wg0-client restart

>Please make sure the paths to certain binaries are congruent with your own system. If they are not, the script will fail. Some distributions put them in different places ( EG: /bin/bash instead of /usr/bin/bash ). If you are not sure where they are, you can do *which binaryname* that should report the full path to the binary.

    $ which bash
    /usr/bin/bash

Make the script executable:

    chmod +x /usr/local/scripts/wg-watch.sh


Once we have that done, we need to schedule it. I choose to schedule this every five minutes, but if you want to wait longer that is up to you.

Schedule the script to run on a regular basis using *cron*. You can find out more about cron here: https://opensource.com/article/17/11/how-use-cron-linux

We're going to use *crontab* to add this script to the list of jobs.

    crontab -e

Once the crontab editor is open, add this:

    */5 * * * * /usr/local/scripts/wg-watch.sh

Write and close the file. Crontab should confirm that it has been updated.

You should be set with a wireguard VPN tunnel between a server and a client along with a script to bring the tunnel back up if it fails.
