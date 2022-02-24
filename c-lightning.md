---
layout: default
title: c-lightning
nav_order: 10
parent: Lightning
---
<!-- markdownlint-disable MD014 MD022 MD025 MD033 MD040 -->
# Lightning: C-LIGHTNING
{: .no_toc }

We set up [c-lightning](https://github.com/ElementsProject/lightning#readme){:target="_blank"}, A specification compliant Lightning Network implementation in C by [Blockstream](https://blockstream.com/lightning/){:target="_blank"}.

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Installation

The installation of c-lightning is straight-forward, but the application is quite powerful and capable of things not explained here. Check out their [GitHub repository](https://github.com/ElementsProject/lightning){:target="_blank"} for a wealth of information about their open-source project and Lightning in general.

### Download

We'll clone and verify c-lightning from source and install all requirements.
The application should run in a new, seperated user for security reasons.

* Login as "admin" user, create the "cl" service user and add it to the groups "bitcoin" and "debian-tor"

  ```sh
  $ sudo adduser --disabled-password --gecos "" cl
  $ sudo usermod -a -G bitcoin,debian-tor cl
  ```

* Switch to the "cl" user. Download and verify the source from GitHub.
  
  ```sh
  $ sudo su - cl
  $ git clone https://github.com/ElementsProject/lightning.git
  $ cd lightning
  $ git reset --hard v0.10.2
  ```
  
* Get the PGP key from [Christian Decker](https://github.com/cdecker) to verify the signature

  ```sh
  $ curl https://raw.githubusercontent.com/ElementsProject/lightning/master/contrib/keys/cdecker.txt | gpg --import
  > ...
  > gpg: key A26D6D9FE088ED58: 51 signatures not checked due to missing keys
  > gpg: key A26D6D9FE088ED58: public key "Christian Decker <decker.christian@gmail.com>" imported
  > ...
  ```

* Verify the signature
  
  ```
  work in progress...
  for now I assume all signatures are right
  ```
  
* Exit the "cl" user session back to "admin".
  Install the following dependencies
  
  ```
  $ exit
  $ sudo apt-get install -y \
    autoconf automake build-essential git libtool libgmp-dev \
    libsqlite3-dev python3 python3-mako net-tools zlib1g-dev libsodium-dev \
    gettext
  $ sudo apt-get install -y postgresql libpq-dev
  $ sudo pip3 install mrkd==0.2.0
  $ sudo pip3 install mistune==0.8.4
  ```
  
* Some dependencies are also required for our "cl" user.
  Install the following dependencies for him

  ```
  $ sudo -u cl pip3 install --user mrkd==0.2.0
  $ sudo -u cl pip3 install --user mistune==0.8.4
  $ sudo -u cl pip3 install --user -r /home/cl/lightning/requirements.txt
  > ERROR: Invalid requirement: './contrib/pyln-client' (from line 7 of /home/cl/lightning/requirements.txt)
  > Hint: It looks like a path. File './contrib/pyln-client' does not exist.
  ```
  
___________________________________________________________
  

* Configuring EXPERIMENTAL_FEATURES enabled

  ```sh
  $ sudo ./configure --enable-experimental-features
  ```
  
* Building C-lightning from source and install to /usr/local/bin

  ```sh
  $ sudo make
  $ sudo make install
  ```

* Check the version

  ```
  $ lightning-cli --version
  > v0.10.2
  ```
  
### Data directory

Now that c-lightning is installed, we need to configure it to work with Bitcoin Core and run automatically on startup.

* Create the "cl" service user, and add it to the groups "bitcoin" and "debian-tor"

  ```sh
  $ sudo adduser --disabled-password --gecos "" cl
  $ sudo usermod -a -G bitcoin,debian-tor cl
  ```
  
* Create the lightningd data directory

  ```sh
  $ sudo mkdir /data/lightningd
  $ sudo chown -R cl:cl /data/lightningd
  ```
  
* Open a "cl" user session

  ```sh
  $ sudo su - cl
  ```
  
* Create symbolic links pointing to the lightningd and bitcoin data directories

  ```sh
  $ ln -s /data/lightningd /home/cl/.lightningd
  $ ln -s /data/bitcoin /home/cl/.bitcoin
  ```
  
* Display the links and check that they're not shown in red (this would indicate an error)

  ```sh
  $ ls -la
  ```
  
### Configuration

* Create the c-lightning configuration file and paste the following content.
  Save and exit.

  ```sh
  $ nano /data/lightningd/config
  ```
  ```sh
  # RaspiBolt: c-lightning configuration
  # /data/lightningd/config

  network=bitcoin
  log-file=cl.log
  log-level=debug

  # Tor settings
  proxy=127.0.0.1:9050
  bind-addr=127.0.0.1:9735
  addr=statictor:127.0.0.1:9051
  always-use-proxy=true
  ```
  
  ---
  
## Run c-lightning

As "admin" user, we first add him to "cl" group and than start c-lightning manually to check if everything works fine.

```sh
$ sudo adduser admin cl
$ lightningd --conf=/data/lightningd/config
```