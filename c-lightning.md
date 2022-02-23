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
work in progress...
___________________________________________________________
  ```sh
  $ gpg --verify SHA256SUMS.asc
  > gpg: Signature made Wed Nov  3 19:02:02 2021 CET
  > gpg:                using RSA key B7C4BE81184FC203D52C35C51416D83DC4F0E86D
  > gpg: Good signature from "Christian Decker <decker.christian@gmail.com>" [unknown]
  > gpg:                 aka "Christian Decker <decker@blockstream.com>" [unknown]
  > gpg: WARNING: This key is not certified with a trusted signature!
  > gpg:          There is no indication that the signature belongs to the owner.
  > Primary key fingerprint: B731 AAC5 21B0 1385 9313  F674 A26D 6D9F E088 ED58
  ```

* Verify the checksum for the application

  ```sh
  $ sha256sum --check SHA256SUMS
  > ...
  > clightning-v0.10.2.zip: OK
  > ...
  ```
  
* For some dependencies Python and pip3 are required. Python should already be installed. Check your Python version

  ```
  $ python --version
  > Python 3.9.2
  ```

 * Install pip3 and check its version

  ```
  $ sudo apt update
  $ sudo apt install python3-pip
  $ pip3 --version
  > pip 20.3.4 from /usr/lib/python3/dist-packages/pip (python 3.9)
  ```

* Unzip c-lightning

  ```sh
  $ unzip clightning-v0.10.2.zip
  ```

* Install dependencies

  ```sh
  $ sudo apt-get install -y \
      autoconf automake build-essential git libtool libgmp-dev \
      libsqlite3-dev python3 python3-mako net-tools zlib1g-dev libsodium-dev \
      gettext unzip
  $ sudo apt-get install -y postgresql libpq-dev
  $ sudo pip3 install mrkd==0.2.0
  $ sudo pip3 install mistune==0.8.4
  $ sudo pip3 install -r clightning-v0.10.2/requirements.txt
  > ERROR: Invalid requirement: './contrib/pyln-client' (from line 7 of clightning-v0.10.2/requirements.txt)
  > Hint: It looks like a path. File './contrib/pyln-client' does not exist.
  ```
stuck here at blockheight 724446
________________________________________________________________________________________________________________

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
