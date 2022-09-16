---
Title: Get a free domain name and certificate
---

___

With this tutorial you will get a valid SSL certificate from Let's Encrypt without having to open any incoming ports. You can use the certificate to enable HTTPS with your reverse proxy (Apache, Nginx, Caddy, ...) or other self hosted service. Since it only uses acme.sh which is a shell script it should work on everything that runs linux.

The tutorial was written for and tested with Duck DNS and deSEC, but you can (in theory, because I did sadly encounter a few bugs/incompatibilities here and there) use [every of the 150+ DNS provider supported by acme.sh (there is also a second page at the end!)](https://github.com/acmesh-official/acme.sh/wiki/dnsapi). If you want to use a wildcard certificate I would recommend deSEC because Duck DNS currently has a bug/incompatibility with acme.sh.

If you want to use another DNS provider you can skip right to [2. Install acme.sh](#2-install-acmesh), but need to change the parameter `--dns YOURDNS` in all the commands and set all necessary variables yourself according to the [acme.sh DNS API wiki](https://github.com/acmesh-official/acme.sh/wiki/dnsapi).


## 1. Sign in/up to a DynDNS provider

#### 1. Duck DNS

1. Go to https://www.duckdns.org/ and sign in with one of the providers at the top.

2. After your are successfully logged in, enter the sub domain you want and press `add domain`. This domain name (including `.duckdns.org`) needs to be replaced in all commands where you see `YOURDOMAIN`.

3. Enter either

    1. the local IP address of your server if your server is not accessible from the internet

    2. or the public IP address of your server if your server is accessible from the internet

    in the `current ip` field and press `update ip`.

    The choosen sub domain name will be the one that the server/service needs to be addressed when using the certificate, for it to be valid. Since you set the sub domain to the IP address of your server it should be reachable when the sub domain name get's translated by any DNS. Depending on your home router you might need add an exception of the sub domain name to the DNS rebind protection.

4. Keep the website open, because you need it in a later step.


#### 2. deSEC

1. Go to https://desec.io/signup and create a new account. It doesn't matter what you choose for `Do you want to set up a domain right away?` because you can add a domain afterwards.

2. Log into your deSEC account.

3. If you havent't added a domain during signup, click on the `+` button on the right and enter the subdomain you want and add `.dedyn.io` after your subdomain so it looks like `example.dedyn.io`. If the sub domain was added successfull there will be a popup with setup instructions which you will not need and can be closed. This domain name needs to be replaced in all commands where you see `YOURDOMAIN`.

4. Optionally add a DNS record: Click onto your sub domain name and then the `+` button on the right. A popup with `Create New Record Set` will show up. Choose the `Record Set Type` value `A` and enter either:

    1. the local IP address of your server if your server is not accessible from the internet

    2. or the public IP address of your server if your server is accessible from the internet

    in the `IPv4 address` field and press `save`.

    The choosen sub domain name will be the one that the server/service needs to be addressed when using the certificate, for it to be valid. Since you set the sub domain to the IP address of your server it should be reachable when the sub domain name get's translated by any DNS. Depending on your home router you might need add an exception of the sub domain name to the DNS rebind protection.

5. At the top menu change to `TOKEN MANAGEMENT` and press the `+` button on the right. A popup with `Generate New Token` will show up. Enter a token name of your choosing (the name doesn't matter and is only for the convenience of knowing what the token is used for) and press `save`.

    Now there will be a green bar at in the popup saying

    ```
    Your new token's secret value is: aaaabbbbccccddddeeeeffffgggg
    It is only displayed once.
    ```

    Copy the secret token value into an editor because you need it later. But don't worry, you can always come back to this step and generate a new token in case you loose the secret token value.


## 2. Install acme.sh

1. Install acme.sh:

    ```sh
    curl https://get.acme.sh | sh -s
    ```

    If you wish to receive an expiration notification email before your certificates expires you can insert your email address and install acme.sh with the following command:

    ```sh
    curl https://get.acme.sh | sh -s email=my@example.com
    ```

    You can find more information on expiration emails here: https://letsencrypt.org/docs/expiration-emails/

2. Restart the terminal.


## 3. Configure acme.sh

1. Enable auto update:

    ```sh
    acme.sh --upgrade --auto-upgrade
    ```

2. Change the default CA to Let's Encrypt (see explanation in the remarks):

    ```sh
    acme.sh --set-default-ca --server letsencrypt
    ```

3. Take the token from the DynDNS provider website and insert it into either one of the following commands between the quotation marks:

    For deSEC:

    ```sh
    export DEDYN_TOKEN="aaaabbbbccccddddeeeeffffgggg"
    ```

    For Duck DNS:

    ```sh
    export DuckDNS_Token="aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee"
    ```


## 3. Issue a certificate

In the following commands you need to replace `YOURDNS` with either `dns_duckdns` when you chose Duck DNS or `dns_desec` when you chose deSEC.

Insert your registered sub domain in the following command to issue your first certificate:

```sh
acme.sh --issue --dns YOURDNS --domain YOURDOMAIN
```

If you have registered more domains you can add them as alternative names to the certificate by adding more `--domain YOURDOMAIN` at the end:

```sh
acme.sh --issue --dns YOURDNS --domain subdomain.example.com --domain subdomain-nextcloud.example.com --domain subdomain-vaultwarden.example.com
```

The first given `--domain` of the `--issue` command will be the primary domain of the certificate and the only one domain you will need to state when running other acme.sh commands. I would recommend to keep the primary domain the same when adding/removing other sub domains.


## 4. Install the certificate to a target directory

After the certificate is issued acme.sh needs to copy the certificate to a target directory and can a command after each renewal of the certificate.

The target directory and reload command specific to the primary domain (the domain of the first `--domain` parameter). So the target directory (or at least filename) must be unique and the reload command must be for this specific certificate.

The following commad sets the variable `CERTIFICATE_DIRECTORY` (which is just for ease of use in the next command) with a directory of your choosing and creates the directory.

```sh
CERTIFICATE_DIRECTORY=$HOME/certificates
mkdir -p "$CERTIFICATE_DIRECTORY"
```

Now you tell acme.sh where and under which filenames it should copy the certificate (`--cert-file` and `--fullchain-file`) and key (`--key-file`) files and which command (`--reloadcmd`) it should run to restart your reverse proxy or other service.

```sh
acme.sh --install-cert --domain YOURDOMAIN --cert-file "$CERTIFICATE_DIRECTORY/certificate.pem" --fullchain-file "$CERTIFICATE_DIRECTORY/fullchain.pem" --key-file "$CERTIFICATE_DIRECTORY/key.pem" --reloadcmd "sudo service apache2 force-reload"
```


## 5. Automatic renewal

Certificates are only valid for 90 days. Therefore acme.sh will create a daily cron job running at a random time at night that will:
* renew every certificate every 60 days
* copy the certificate and key files to their destination (as configured in [4. Install the certificate to a target directory](#4-install-the-certificate-to-a-target-directory))
* run the reload command (as configured in [4. Install the certificate to a target directory](#4-install-the-certificate-to-a-target-directory))

___

### Remarks:

1. How can I add more domain names to my certificate?

    Run the command from [3. Issue a certificate](#3-issue-a-certificate) again with all domain names (old and new) that you want in your certificate. As long as the primary domain stays the same it is not necessary to install the certificate again.

2. Why change the default CA to Let's Encrypt?

    I did encounter bugs with the default CA of acme.sh (ZeroSSL) which where gone once I switched to Let's Encrypt.

3. How do I create a wildcard certificate?

    Add *.YOURSUBDOMAIN.YOURSITEDOMAIN.com as an alternative domain name to your certificate:

    ```sh
    acme.sh --issue --dns dns_... --domain YOURSUBDOMAIN.YOURSITEDOMAIN.com --domain *.YOURSUBDOMAIN.YOURSITEDOMAIN.com
    ```

    In theory it works with Duck DNS, but if you add the wildcard as an alternative name there sadly is a bug or incompatibility (depending on who you want to blame) and acme.sh runs into an infitie loop. It works if you only use the wildcard domain as the primary domain name. But with only a wildcard in the certificate I don't know if this certificate will play nice with all browsers and applications.
    
    If you want to use acme.sh and create a wildcard certificate desec.io works as a DNS provider.

4. Add the `--test` parameter to the `--issue` command to create test (or staging) certificates which are not valid but are better if you are just testing things. The certificate will stay in the staging environment until you renew it without the `--test` parameter:

    ```sh
    acme.sh --renew -d YOURSUBDOMAIN.YOURSITEDOMAIN.com
    ```

    More on that topic here: https://letsencrypt.org/docs/staging-environment/

5. If you don't need a certificate anymore you can just delete the directory with your domain name from the `.acme.sh` directory in your home directory.

6. Uninstall acme.sh with the following command:

    ```sh
    acme.sh --uninstall
    ```

    and delete the `.acme.sh` directory in your home directory.
