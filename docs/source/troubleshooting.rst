Troubleshooting
###############

Broken groups permissions
*************************

Problem
=======

User cannot access files which have proper permissions for his group.

Example
=======

User testuser from common group is unable to access file with permissions::

  drwxrws---   2 root common  0 Mar 28 17:47 test

Solution
========

The problem may be related to secondary group permissions. If common is not a primary group of user testuser, FUSE will not pass it to LizardFS, so permission would not be granted.

To omit the problem, restart LizardFS master with ignoregid parameter added to mfsexports.

Example
=======

::

  # etc/mfs/mfsexports.cfg
  *                   	/   	rw,alldirs,maproot=0,ignoregid

References
==========

https://github.com/lizardfs/lizardfs/issues/265


Problem with read / write 
*************************

Apr 30 13:41:11 machine33 mfsmount[8131]: can't connect to (AC11B00D:9422): ETIMEDOUT (Operation timed out)
Apr 30 13:41:29 machine33 mfsmount[8131]: can't connect to (AC11B00D:9422): ETIMEDOUT (Operation timed out)
