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
  
  ```sh
  work in progress...
  for now I assume all signatures are right
  ```
  
* Exit the "cl" user session back to "admin".
  Install the following dependencies
  
  ```sh
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

  ```sh
  $ sudo -u cl pip3 install --user mrkd==0.2.0
  $ sudo -u cl pip3 install --user mistune==0.8.4
  $ cd /home/cl/lightning
  $ sudo git reset --hard v0.10.2
  $ sudo -u cl pip3 install --user -r requirements.lock
  ```

* Configuring "EXPERIMENTAL_FEATURES" enabled

  ```sh
  $ sudo -u cl ./configure --enable-experimental-features
  ```
  
* Building C-lightning from source and install to /usr/local/bin.
  Be patient, that can take some time

  ```sh
  $ sudo -u cl make
  $ sudo make install
  ```

* Check the version

  ```sh
  $ lightning-cli --version
  > v0.10.2
  ```
  
  
### Data directory

Now that c-lightning is installed, we need to configure it to work with Bitcoin Core and run automatically on startup.
  
* As "admin" user, move the "/home/cl/lightning" directory into "/data/". 
  This creates a new directory called "/data/lightning".
  Set the "cl" user as owner of this directory

  ```sh
  $ sudo mv /home/cl/lightning/ /data/
  $ sudo chown -R cl:cl /data/lightning
  ```

* Create a symbolic link, which points from your admin home directory into the lightning directory
  
  ```sh
  $ sudo ln -s /data/lightning /home/admin/.lightning
  ```
  
* Open a "cl" user session

  ```sh
  $ sudo su - cl
  ```
  
* Create symbolic links pointing to the lightning and bitcoin data directories

  ```sh
  $ ln -s /data/lightning /home/cl/.lightning
  $ ln -s /data/bitcoin /home/cl/.bitcoin
  ```
  
* Display the links and check that they're not shown in red (this would indicate an error)

  ```sh
  $ ls -la
  ```
  
  
### Configuration

* Still as "cl" user, create the c-lightning configuration file and paste the following content.
  Save and exit.

  ```sh
  $ nano /data/lightning/config
  ```
  ```sh
  # RaspiBolt: c-lightning configuration
  # /data/lightning/config

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

* We first add the "admin" user to "cl" group to give him read-only access to certain files.
  Than start a new "cl" user session and start c-lightning manually to check if everything works fine.

  ```sh
  $ sudo adduser admin cl
  $ sudo su - cl
  $ lightningd --conf=/data/lightning/config
  ```

If no error is thrown c-lightning is usable now and you are good to go.
