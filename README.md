## caman

A self-signing certificate authority manager - create your own certificate
authority, and generate and manage SSL certificates using openssl.

If you want to see how caman works and why it exists, you read the
accompanying article,
[Self-Signing Certificate Authorities](http://radiac.net/blog/2015/05/self-ca/)

This document explains how to use caman to
[create a certificate authority](#creating-a-certificate-authority), optionally [use an intermediate CA](#using-an-intermediate-ca), and to
[create, sign, renew and revoke](#managing-host-certificates) host
certificates.

Version 0.3.2, 2016-11-24. For changelog and upgrade information, see
[Changes](CHANGES.md)

### Quickstart

To create a certificate authority and start signing:

    git clone https://github.com/it-toppp/caman.git
    cd caman
    cp ca/caconfig.cnf.default ca/caconfig.cnf && vi ca/caconfig.cnf
    cp ca/host.cnf.default ca/host.cnf && vi ca/host.cnf
    ./caman init
    ./caman new host.example.com
    ./caman sign host.example.com
    ./caman renew host.example.com
    ./caman revoke host.example.com

Read on to see more details, how you can do this using an intermediate certificate authority, and how to create wildcard and SAN certificates.


### Creating a Certificate Authority

1. Make sure ``openssl`` is installed on your system before using caman:
   * Debian and Ubuntu: ``sudo apt-get install openssl``

2. Clone this repository:

        git clone https://github.com/it-toppp/caman.git

   The ``.gitignore`` is set up to ignore all files caman will create. This is
   to prevent you from accidentally pushing secrets to a public repository.
   
   Although these instructions assume you'll keep everything in the cloned
   directory, the ``caman`` script operates on the current working directory
   and just expects it to contain the ``ca`` directory.
   
   This means you can move/symlink the script to ``/usr/local/bin/`` to make it
   available system-wide, or move the ``ca`` directory into a separate folder
   or repository.
   
3. Configure the files in the ``ca`` directory -
   (see [Configuration](#configuration))
   
4. Initialise caman in the current directory:

        cd caman
        ./caman init

   * You will be asked for a PEM key - this must be at least 4 characters long,
     but the longer the better. Keep it safe - you will need it for most caman
     commands.
   * If you plan to use a intermediate CAs, this will be your root CA.

   You are now ready to
   [create and manage host certificates](#managing-host-certificates).

5. Optional: Distribute ``ca/ca.crt.pem`` for your host certificates to be
   recognised; see [Distribution](#distribution) for more information

Keep ``ca/ca.key.pem`` private. If it is compromised, you will need to destroy
your certificate authority and start again.


#### Configuration

Copy the default configs:
   
    cp ca/caconfig.cnf.default ca/caconfig.cnf
    cp ca/host.cnf.default ca/host.cnf

Edit both files; look for comments starting ``# >>`` for where you need to
make changes.

Changes to make in ``ca/caconfig.cnf``:
* Change the 6 values under ``[ req_distinguished_name ]``:
  * ``countryName``: your two-character country code
  * ``stateOrProvinceName``: your state or province
  * ``organizationName``: the name of your organisation
  * ``organizationUnitName``: your department in the organisation
  * ``commonName``: the name of your organisation
  * ``emailAddress``: your e-mail address
* The lifespan of your CA is ``default_days`` - 100 years by default

In ``ca/host.cnf``:
* Change 5 of the values under ``[ host_distinguished_name ]``:
  * ``countryName``: the two-character country code for this host
  * ``stateOrProvinceName``: the state or province for this host
  * ``organizationName``: the name of the organisation for this host
  * ``organizationUnitName``: your department in the organisation
  * ``emailAddress``: the e-mail address for the admin for this host
  * Do not change ``commonName`` - this is a placeholder which will be set by
    caman
* The lifespan of your host certs is ``default_days`` - 10 years by default


#### Distribution

You need to distribute your ``ca/ca.crt.pem`` to clients for your host
certificates to be recognised.

To install your CA cert system-wide in Debian and Ubuntu:

    sudo cp ca/ca.crt.pem /usr/local/share/ca-certificates/my_ca_name.crt
    sudo dpkg-reconfigure ca-certificates

To install your CA cert system-wide in other Linux distros:

    cp ca/ca.crt.pem "/etc/openssl/certs/$( \
        openssl x509 -inform PEM -subject_hash -in ca/ca.crt.pem | head -1 \
    ).0"

To install your CA cert system-wide in Windows:

1. For Windows Certificate Manager to recognise your certificate, you will need
   to remove the ``.pem`` file extension and distribute the file as ``ca.crt``.
2. Open the file from your filer or Internet Explorer like a normal file;
   Windows Certificate Manager will be used automatically.
3. Click "Install certificate..." and accept all defaults

Some applications (such as Firefox and Thunderbird) have their own certificate
stores; you may need to install your root certificate in these applications
separately.



### Managing host certificates

Host certificates are found in the ``store`` directory. Each host has its own
directory with the config and signing request, and each sign operation creates
a new directory with today's date. Use the files inside the latest directory.

#### Add a new host

    ./caman new <hostname> [<alt> [<alt> ...]]

* ``<hostname>`` is the main hostname for the certificate
* Use an asterisk subdomain to generate a wildcard certificate
* Add multiple ``<alt>`` hostnames after the main hostname to create a SAN
  certificate

Examples:
* Single host: ``./caman new myserver.example.com``
* Wildcard: ``./caman new *.example.com``
* SAN: ``./caman new myserver.example.com virtual1.example.com virtual2.example.com``

This command generates a config file for this host in
``store/hostname/config.cnf``, using the defaults you configured in
``ca/host.cnf``. You can edit this file manually to customise it further (for
example, to change the organisational unit name from your default).


#### Create a new certificate

    ./caman sign <hostname>

This will generate a new private key, CSR, and signed certificate


#### Revoke a certificate

    ./caman revoke <hostname>

You will need to re-publish ``ca/ca.crl.pem`` after running this command.


#### Renew a certificate

    ./caman renew <hostname>

This revokes the existing certificate, and then creates a new one,
so is suitable for replacing both expired or compromised host certificates
You will need to re-publish ``ca/ca.crl.pem`` after running this command.


### Contributing

Contributions are welcome, preferably via pull request.

Thanks to all contributors, who are listed in [CHANGES](CHANGES.md)
