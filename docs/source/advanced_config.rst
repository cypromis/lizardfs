Advanced configuration
######################

Configure georeplication (custom goals) 
***************************************

Set label of a chunkserver
==========================

The label for the Chunkservers is set in the mfschunkserver.cfg file. ::

   LABEL = ssd

After changing the configuration you must reload the chunkserver:: 

   $ mfschunkserver -c path/to/config reload

If there is no LABEL entry in the config, the chunkserver has a default label of “_” (i.e. wildcard), which has a special meaning when defining goals and means “any chunkserver”.

Show labels of connected chunkservers
=====================================

From the command line::

   $ lizardfs-admin list-chunkservers <master ip> <master port>

Via the cgi (webinterface):

In the ‘Servers’ tab in the table ‘Chunk Servers’ there is a column ‘label’ where labels of the chunkservers are displayed.

Configure goals
===============

Goals configuration is defined in the file mfsgoals.cfg used by the master server.

Syntax of the mfsgoals.cfg file is::

   	id name : label ...

The # character starts comments.

There can be up to 20 replication goals configured, with ids between 1 and 20 inclusive. Each file stored on the filesystem refers to some goal id and is replicated according to the goal currently associated with this id. 
By default, goal 1 means one copy on any chunkserver, goal 2 means two copies on any two chunkservers and so on. The purpose of mfsgoals.cfg is to override this behavior, when desired. The file is a list of goal definitions, each consisting of an id, a name and a list of labels. 

**Id** 
  indicates the goal id to be redefined. If some files are already assigned this goal id, their effective goal will change.

**Name** 
  is a human readable name used by the user interface tools (mfssetgoal, mfsgetgoal). A name can consist of up to 32 alphanumeric characters: a-z, A-Z, 0-9, _.

**list of labels** 
  is a list of chunkserver labels as defined in the mfschunkserver.cfg file. A label can consist of up to 32 alphanumeric characters: a-z, A-Z, 0-9, _. For each file using this goal and for each label, the system will try to maintain a copy of the file on some chunkserver with this label. One label may occur multiple times - in such case the system will create one copy per each occurence. The special label _ means "a copy on any chunkserver".

Note that changing the definition of a goal in mfsgoals.cfg affects all files which currently use a given goal id.

Examples
========

Example of goal definitions::

   	3 3 : _ _ _ # one of the default goals (three copies anywhere)
   	8 not_important_file : _ # only one copy
   	11 important_file : _ _
   	12 local_copy_on_mars : mars _ # at least one copy in the Martian datacenter
   	13 cached_on_ssd : ssd _
   	14 very_important_file : _ _ _ _

For further information see::

  $ man mfsgoals.cfg

Show current goal configuration
===============================

From the command line::

   $ lizardfs-admin list-goals <master ip> <master port>

Via cgi (webinterface):

In the ‘Config’ tab there is a table called ‘Goal definitions’.

Set and show goal of the file/directory
=======================================

**set**

   The goal of a file/direcotry can be set using::

	   $ mfssetgoal goal_name object

   which will result in setting the goal of an object to goal_name.

   By default all new files created in the directory are created with the goal of the directory.

   Additional options are: 

   * (-r) - change goals recursively for all objects contained in a directory
   * goal_name[+|-] - when goal_name is appended with +, the goal is changed only if it will “increase security” (i.e. goal_name id is higher than id of the current goal)


**show**

   Current goal of the file/directory can be shown using::

      $ mfsgetgoal object

   Result is the name of the currently set goal.

   To list the goals in a directory use::

      $ mfsgetgoal -r directory

   which for every given directory additionally prints the current value of all contained objects (files and directories).

   To show where exactly file chunks are located use::

      $ mfsfileinfo file

For further information see: man mfssetgoal, man mfsfileinfo


Configure tape goals
********************

.. Note:: using tape goals makes no sense at all without setting up lizardfs-tapeserver first (see chapter :ref:`configure_lto`).

Tape goals are configured just like regular goals, save one difference in naming. In order to create a tape goal, append a “@” sign to the end of its definition.

Example mfsgoals.cfg contents::

	1 1 : _
	2 2 : _ _
	3 tape1 : _ _@
	4 tape2: ssd _@ ts@
	5 fast: ssd ssd _

The example above contains 2 tape goal definitions.

The first one (tape1), configures that there should be 2 copies of each chunk:

* 1 on any chunkserver
* 1 on any tape.

The second one (tape2) requires each chunk to have 3 copies:

* 1 on chunkserver labeled “ssd”
* 1 on any tape
* 1 on tape labeled “ts”



Configure rack awareness (network topology)
*******************************************

Topology of a LizardFS network can be defined in the mfstopology.cfg file.
This configuration file consists of lines matching the following syntax::

   ADDRESS SWITCH-NUMBER

ADDRESS can be represented as:

+-------------------+-------------------------------------------------------+
|  \*               | all addresses                                         |
+-------------------+-------------------------------------------------------+
|  n.n.n.n          | single IP address                                     |
+-------------------+-------------------------------------------------------+
|  n.n.n.n/b        | IP class specified by network address and bits number |
+-------------------+-------------------------------------------------------+
|  n.n.n.n/m.m.m.m  | IP class specified by network address and mask        |
+-------------------+-------------------------------------------------------+
|  f.f.f.f-t.t.t.t  | IP range specified by from-to addresses (inclusive)   |
+-------------------+-------------------------------------------------------+

Switch number can be specified as a positive 32-bit integer.

Distances calculated from mfstopology.cfg are used to sort chunkservers during read/write operations. Chunkservers closer (with lower distance) to a client will be favoured over further away ones.

Please note that new chunks are still created at random to ensure their equal distribution. Rebalancing procedures ignore topology configuration as well.

As for now, distance between switches can be set to 0, 1, 2:

  0 - IP addresses are the same

  1 - IP addresses differ, but switch numbers are the same

  2 - switch numbers differ

The topology feature works well with chunkserver labeling - a combination of the two can be used
to make sure that clients read to/write from chunkservers best suited for them (e.g. from the same network switch).

Configure QoS
*************

Quality of service can be configured in the /etc/mfs/globaliolimits.cfg file.

Configuration options consist of:

* subsystem <subsystem>
  cgroups subsystem by which clients are classified
* limit <group> <throughput in KiB/s>
* limit for clients in cgroup <group>
* limit unclassified <throughput in KiB/s>
* limit for clients that do not match to any specified group.
		
If globaliolimits.cfg is not empty and this option is not set, not specifying limit unclassified will prevent unclassified clients from performing I/O on LizardFS

Example 1::

   # All client share 1MiB/s bandwidth
	limit unclassified 1024

Example 2::

   # Ale clients in blkio/a group are limited to 1MiB/s, other clients are blocked
	subsystem blkio
	limit /a 1024

Example 3::

   # blkio group /a is allowed to transfer 1MiB/s
   # /b/a group gets 2MiB/s
   # unclassified  clients share 256KiB/s of bandwidth.
        subsystem blkio
       	limit unclassified 256
       	limit /a   1024
       	limit /b/a 2048
	
Configure Quotas
****************

Quota mechanism can be used to limit inodes usage and space usage for users and groups. By default quotas can be set only by a superuser. The SESFLAG_ALLCANCHANGEQUOTA flag in the mfsexports.cfg file would allow everybody to change quota.

In order to set quota for a certain user/group you can simply use mfssetquota tool::

   mfssetquota  (-u UID/-g GID)   SL_SIZE   HL_SIZE   SL_INODES   HL_INODES   MOUNTPOINT_PATH

where:
* SL - soft limit
* HL - hard limit

.. _configure_lto:

Configure LTO library
*********************

Installation
============

THe LizardFS tapeserver package can be installed via::

   $ apt-get install lizardfs-tapeserver # Debian/Ubuntu
   $ yum install lizardfs-tapeserver # CentOS/RedHat

Configuration
=============

The configuration file for the lizardfs-tapeserver is located at /etc/mfs/lizardfs-tapeserver.cfg.
The tapeserver needs a working mountpoint of your LizardFS installation.
Configuration consists mainly of listing changer and volume devices of a tape library.

Example configuration::

   [lizardfs-tapeserver]
   # Name
   name = tapesrv
   # Master host network info
   masterhost = 192.168.1.5
   masterport = 9424
   # Path to mountpoint used for file reading/writing
   mountpoint = /mnt/tapemount
   # Path to temporary cache for files
   cachepath  = /tmp/cache
   # Path to sqlite3 database
   database = /opt/tape.db
   # Changer devices
   changers = /dev/sg3
   # Drive devices
   drives = /dev/st0,/dev/st1
   # Volume ids
   volumes = 000003,000180,000002
   # Label
   label = tapeserver1

Verification
============

Installation can be easily verified using the lizardfs-admin command::

   $ lizardfs-admin list-tapeserver MASTER_ADDR MASTER_PORT

If the installation succeeded, the command above should result in listing all tapeservers connected to the current master.

Verification if tape storage works properly can be achieved by the steps below:

* create a test file

* set tape goal to the file: mfssetgoal your_tape_goal testfile

* wait for replication to take place, check its status with ‘mfsfileinfo’ command::

   $ mfsfileinfo testfile

* Replication to tape is complete after tape copy status changes from Creating to Ok

* verify that the file was actually stored on tape::

	$ tar tf /dev/your_tape_volume # will list all files present on tape
	$ tar xvf /dev/your_tape_volume filename # will extract file ‘filename’ from tape


Metadata mount
**************

LizardFS metadata can be managed through a special mountpoint called META.
META allows to control trashed items (undelete/delete them permanently) and view files that are already deleted but still held open by clients.

Mounting meta
=============

To be able to mount metadata you need to add “mfsmeta” parameter to mfsmount command::
# mfsmount /mnt/lizardfs-meta -o mfsmeta

after that you will see this line at mtab::

   mfsmeta#10.32.20.41:9321 on /mnt/lizardfs-meta type fuse (rw,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other)

The structure of the mounted metadata directory will like this::

   /mnt/lizardfs-meta/
   ├── reserved
   └── trash
       └── undel

Trash directory
***************

Each file with a trashtime higher than zero will be present here. You can recover those files or delete files permanently.

Recovering from trash
=====================

In order to recover a file, just must move it to the undel/ directory. Files are represented by their inode
number and path, so file dir1/dir2/file.txt with inode 5 will be present at trash/5|dir1|dir2|file.txt,
recovering it would be performed like this::

   $ cd trash
   $ mv ‘5|dir1|dir2|file.txt’ undel/

Removing permanently
====================

In order to delete a file permanently, just remove it from trash.

Reserved directory
==================

If you delete a file, but someone else use this file and keep an open descriptor,
you will see this file in here until descriptor is closed.

Deploy LizardFS HA Cluster
**************************

LizardFS can be run as a high-availability cluster on several nodes. When working in HA mode, a dedicated daemon watches the status of the metadata servers and performs a failover whenever it detects a master server crashed (e.g. due to power outage). Running LizardFS installation as a HA-cluster significantly increases its availability. A reasonable minimum of metadata servers in a HA installation is 3.

In order to deploy LizardFS as a high-availability cluster, follow the steps below.
Steps should be performed on all machines chosen to be in a cluster.

Install the lizardfs-uraft package::

   $ apt-get install lizardfs-uraft for Debian/Ubuntu
   $ yum install lizardfs-uraft for CentOS/RedHat

Prepare your installation:

Fill lizardfs-master config file (/etc/mfs/mfsmaster.cfg). Details depend on your personal
configuration, the only fields essential for uraft are::

   PERSONALITY = ha-cluster-managed
   ADMIN_PASSWORD = your-lizardfs-password

For a fresh installation, execute standard steps for lizardfs-master (creating mfsexports file,
empty metadata file etc.). Do not start lizardfs-master daemon yet.
Fill the lizardfs-uraft config file (/etc/mfs/lizardfs-uraft.cfg). Notable fields are::

	URAFT_NODE_ADDRESS - identifiers of all machines in a cluster
	URAFT_ID - node address ordinal number; should be different for each machine
	URAFT_FLOATING_IP - IP at which LizardFS will be accessible for the clients
	URAFT_FLOATING_NETMASK - a matching netmask for floating IP
	URAFT_FLOATING_IFACE - interface for floating IP


Example configuration for acluster with 3 machines:
===================================================

The first, node1, is at 192.168.0.1, the second node gets hostname node2, and the third one gets hostname node3 and operates under a non-default port number - 99427.

All machines are inside a network with a 255.255.255.0 netmask and use interface eth1.
LizardFS installation will be accessible at 192.168.0.100 ::

   # Configuration for node1:
   URAFT_NODE_ADDRESS = 192.168.0.1
   URAFT_NODE_ADDRESS = node2
   URAFT_NODE_ADDRESS = node3:99427
   URAFT_ID = 0
   URAFT_FLOATING_IP = 192.168.0.100
   URAFT_FLOATING_NETMASK = 255.255.255.0
   URAFT_FLOATING_IFACE = eth1

   # Configuration for node2:
   URAFT_NODE_ADDRESS = 192.168.0.3
   URAFT_NODE_ADDRESS = node2
   URAFT_NODE_ADDRESS = node3:99427
   URAFT_ID = 1
   URAFT_FLOATING_IP = 192.168.0.100
   URAFT_FLOATING_NETMASK = 255.255.255.0
   URAFT_FLOATING_IFACE = eth1

   # Configuration for node3:
   URAFT_NODE_ADDRESS = 192.168.0.3
   URAFT_NODE_ADDRESS = node2
   URAFT_NODE_ADDRESS = node3:99427
   URAFT_ID = 2
   URAFT_FLOATING_IP = 192.168.0.100
   URAFT_FLOATING_NETMASK = 255.255.255.0
   URAFT_FLOATING_IFACE = eth1

Enable arp broadcasting in your system (for floating IP to work)::

	$ echo 1 > /proc/sys/net/ipv4/conf/all/arp_accept

Start the lizardfs-uraft service:

Change “false” to “true” in /etc/default/lizardfs-uraft::

   $ service lizardfs-uraft start

You can check your uraft status via telnet on URAFT_STATUS_PORT (default: 9428)::

	$ telnet NODE-ADDRESS 9428

When running telnet locally on a node, it is sufficient to use::

	$ telnet localhost 9428

