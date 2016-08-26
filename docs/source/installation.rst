Installation
############

.. _get_and_install:

Getting and installing LizardFS
*************************************

There are several methods for getting LizardFS software. The easiest and most common method is to get packages by adding repositories for use with package management tools such as the Advanced Package Tool (APT) or Yellowdog Updater, Modified (YUM). You may also retrieve pre-compiled packages from the LizardFS repository. Finally, you can retrieve tarballs or clone the LizardFS source code repository and build LizardFS yourself.

How to install from Debian packages
===================================

First, add a key which is needed to verify signatures of LizardFS packages::

   # wget -O - http://packages.lizardfs.com/lizardfs.key | apt-key add -

The next step is to add a proper entry in /etc/apt/sources.list.d/

For Debian do::

   # echo "deb http://packages.lizardfs.com/debian/$(lsb_release -sc) $(lsb_release -sc) main" > /etc/apt/sources.list.d/lizardfs.list
   # echo "deb-src http://packages.lizardfs.com/debian/$(lsb_release -sc) $(lsb_release -sc) main" >> /etc/apt/sources.list.d/lizardfs.list

For Ubuntu do::

   # echo "deb http://packages.lizardfs.com/ubuntu/$(lsb_release -sc) $(lsb_release -sc) main" > /etc/apt/sources.list.d/lizardfs.list
   # echo "deb-src http://packages.lizardfs.com/ubuntu$(lsb_release -sc) $(lsb_release -sc) main" >> /etc/apt/sources.list.d/lizardfs.list

This will have created the lizardfs.list file. Now update the packages index::

   # apt-get update

Now you are able to install LizardFS packages (listed below) using::

   # apt-get install <package>

It is also possible to download the source package using::

   # apt-get source lizardfs

.. important:: 
   Before upgrading any existing LizardFS installation, please read the instructions here: https://github.com/lizardfs/lizardfs/blob/master/UPGRADE

LizardFS consists of the following packages:

* lizardfs-master – LizardFS master server
* lizardfs-chunkserver – LizardFS data server
* lizardfs-client – LizardFS client
* lizardfs-adm – LizardFS administration tools (e.g, lizardfs-probe)
* lizardfs-cgi – LizardFS CGI Monitor
* lizardfs-cgiserv – Simple CGI-capable HTTP server to run LizardFS CGI Monitor
* lizardfs-metalogger – LizardFS metalogger server
* lizardfs-common – LizardFS common files required by lizardfs-master, lizardfs-chunkserver and lizardfs-metalogger
* lizardfs-dbg – Debugging symbols for all the LizardFS binaries


How to install from RedHAT EL / CentOS Packages
===============================================

First, add a file with information about the repository:

for RHEL 6 and CentOS 6::

   # curl http://packages.lizardfs.com/yum/el6/lizardfs.repo > /etc/yum.repos.d/lizardfs.repo
   # yum update

for RHEL 7 and CentOS 7::

   # curl http://packages.lizardfs.com/yum/el7/lizardfs.repo > /etc/yum.repos.d/lizardfs.repo
   # yum update

Now you are able to install LizardFS packages (listed below) using::

   # yum install <package>

It is also possible to download the source package (SRPM) using::

   # yum install yum-utils
   # yumdownloader --source lizardfs

..imoirtant:: Before upgrading any existing LizardFS installation, please read the instructions here: https://github.com/lizardfs/lizardfs/blob/master/UPGRADE

LizardFS consists of following packages:

* lizardfs-master – LizardFS master server
* lizardfs-chunkserver – LizardFS data server
* lizardfs-client – LizardFS client
* lizardfs-adm – LizardFS administration tools (e.g, lizardfs-probe)
* lizardfs-cgi – LizardFS CGI Monitor
* lizardfs-cgiserv – Simple CGI-capable HTTP server to run LizardFS CGI Monitor
* lizardfs-metalogger – LizardFS metalogger server
* lizardfs-debuginfo – (CentOS 7 / RHEL 7 only) Debugging symbols and sources for all the LizardFS binaries

How to install from downloaded .deb packages
============================================

Make sure to install lizardfs-common package first before installing other packages.

Also, remember to install lizardfs-cgi before installing lizardfs-cgiserv

In order to install a .deb package, run::

   # dpkg -i <package>

If installing package fails due to dependency problems, run:

   # apt-get -f install

Obtaining LizardFS source code
==============================

LizardFS source code is available on GitHub at the following URL: https://github.com/lizardfs/lizardfs.


