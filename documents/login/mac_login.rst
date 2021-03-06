.. index:: Mac Login
.. _mac_login:

How to login from Mac OS
========================

This section describes how to acquire Kerberos tickets and
log in from different versions of Mac OS X.


Mac OS X 10.7 to 10.10
----------------------

These versions of Mac OS come with Heimdal and kerberized ssh, so you only have 
to modify your Kerberos and SSH configuration files.

Configure Kerberos
^^^^^^^^^^^^^^^^^^

**Kerberos** needs to be configured to be able to find the PDC domain.
The configuration file for kerberos that you need to edit (as root) is **/etc/krb5.conf**.
and should be changed by adding the following entries
::

  [domain_realm]
    .pdc.kth.se = NADA.KTH.SE
  ...
  [appdefaults]
    forwardable = yes
    forward = yes
    krb4_get_tickets = no
  ...
  [libdefaults]
    default_realm = NADA.KTH.SE
    dns_lookup_realm = true
    dns_lookup_kdc = true

If you are not able to become root on your machine you can create a file in your home
directory called for example **~/pdckrb** with the changes shown above.
After this you need to set the path for kerberos like
::

  # For bash
  export KRB5_CONFIG=~/pdckrb/krb5.conf
  # For tcsh
  setenv KRB5_CONFIG  ~/pdckrb/krb5.conf


Acquire kerberos tickets
^^^^^^^^^^^^^^^^^^^^^^^^

In order to get a kerberos ticket::

  kinit --forwardable username@NADA.KTH.SE

You will be asked for your kerberos password and then you have acquired your ticket.
You can see what active tickets you have using
::

  klist -f

More information about kerberos can be found at http://web.mit.edu/kerberos/krb5-current/doc/user/index.html


Configure SSH 
^^^^^^^^^^^^^

OpenSSH can be configured with command line arguments or a configuration file.
The options in the configuration file are parsed in order.
Create or modify the file **~/.ssh/config**
::

  # Hosts we want to authenticate to with Kerberos
  Host *.kth.se *.kth.se.
  # User authentication based on GSSAPI is allowed
  GSSAPIAuthentication yes
  # Key exchange based on GSSAPI may be used for server authentication
  GSSAPIKeyExchange yes
  # Hosts to which we want to delegate credentials. Try to limit this to
  # hosts you trust, and were you really have use for forwarded tickets.
  Host *.csc.kth.se *.csc.kth.se. *.nada.kth.se *.nada.kth.se. *.pdc.kth.se *.pdc.kth.se.
  # Forward (delegate) credentials (tickets) to the server.
  GSSAPIDelegateCredentials yes
  # Prefer GSSAPI key exchange
  PreferredAuthentications gssapi-keyex,gssapi-with-mic
  # All other hosts
  Host *

The file can be downloaded from :download:`here <config>`.

Do remember to set the right permission on the file
::

  chmod 644 ~/.ssh/config

After this you can log in by using
::

  ssh UserName@Cluster.pdc.kth.se


SSH without configuration
^^^^^^^^^^^^^^^^^^^^^^^^^

As an alternative you can also supply these options directly to the ssh command in order to login
::

  ssh -o GSSAPIDelegateCredentials=yes -o GSSAPIKeyExchange=yes -o GSSAPIAuthentication=yes UserName@Cluster.pdc.kth.se


Mac OS X 10.11 to 10.12
------------------------

The native ssh which comes with recent Mac OS X versions (El Capitan 10.11 and Sierra 10.12) does not support 
GSSAPI key exchange, which is required for authenticating to PDC systems 
using Kerberos.  
Two options are available for setting up the connection to PDC. The 
first option listed below is easier if you don't have macports installed on your machine, 
but both methods are described below.

(Method 1) Mac Installer
^^^^^^^^^^^^^^^^^^^^^^^^

Start by downloading `this <https://drive.google.com/file/d/0B3KTk17tdgqDY2xCMWJxMXNvQWs/view?usp=sharing/>`_.

Start the installer and click through the steps.

To avoid interfering with the default binaries in /usr/bin, the installer will place the ssh, scp and sftp binaries in /usr/local/bin, 
and it will adjust your path to make sure this directory is 
listed before the system location in your PATH variable (by adding a line your 
.profile file).

(Method 2) Install openssh via macports
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Another option is to install openssh via macports.  
First, install Xcode through the App Store.  
Open Xcode and choose in the menu:  

*Xcode > Open Developer Tool > More Developer Tools*

A browser will open with a list.  Download and install:

*Command Line Tools for Xcode*

Then install macports from https://www.macports.org.

Finally install openssh through macports with the command
::

  sudo port install openssh +gsskex

You can now restart the computer and continue with the setup of the 
Kerberos file and .ssh/config described above for Mac OS X 10.7 to 10.10.


Installing AFS
--------------

In order to access your home directory you need to install AFS
::

  sudo add-apt-repository ppa:openafs/stable
  sudo apt-get install openafs-client openafs-modules-dkms
  
The last step will take quite some time, so please be patient!
If asked about which AFS cell this workstation belongs to, answer **pdc.kth.se**.
Please note that the openafs-kernel-module will be rebuilt automatically for 
you with every new openafs version and with every kernel upgrade. 
You do not need to do any manual work! To start, stop and use your AFS client.

Then you need to start the AFS daemon
::

  sudo /etc/init.d/openafs-client start
  
After installing AFS you can access your home folder located at
::

  cd /afs/pdc.kth.se/home/u/username
