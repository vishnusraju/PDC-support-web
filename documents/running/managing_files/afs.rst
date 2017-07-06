.. index:: AFS File System
.. _afs:

Guide: AFS
==========

The `afsws` module
^^^^^^^^^^^^^^^^^^

On the PDC systems that use modules, the AFS commands are in the afsws (AFS WorkStation) module.

.. code-block:: bash
		
   > module add afsws

 
Viewing and modifing access (Access Control List)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   

Every directory in AFS is controlled by the Access Control List (ACL) describing different user rights in that directory. To see this list, use the command ``fs listacl`` or ``fs la``. This command entered by the user ``svensson`` might result in something like this:

.. code-block:: bash
	
  > fs la
    Access list for . is
    Normal rights:
    svensson rlidwka
    system:administrators rlidwka
    system:anyuser rl
    andersson rlidwk

Before going into the details of this list, we should take a look at what permissions you can set. Each action you can perform in a directory has its own letter:

  +---------------------+---------------------------------------------+--------------------------------------------------+
  |    Notation         |  File action                                |  Description                                     |
  +=====================+=============================================+==================================================+
  |   ``r``             |     read                                    | read the files in the directory                  |
  +---------------------+---------------------------------------------+--------------------------------------------------+
  |   ``l``             |     list                                    | list the files in the directory                  |
  +---------------------+---------------------------------------------+--------------------------------------------------+
  |   ``i``             |     insert                                  | create new files                                 |
  +---------------------+---------------------------------------------+--------------------------------------------------+
  |   ``d``             |     delete                                  | delete files                                     |
  +---------------------+---------------------------------------------+--------------------------------------------------+
  |   ``w``             |     write                                   | modify files                                     |
  +---------------------+---------------------------------------------+--------------------------------------------------+
  |   ``k``             |     lock                                    | lock files in the directory                      |
  +---------------------+---------------------------------------------+--------------------------------------------------+
  |   ``a``             |     administer                              | change the ACL for the directory                 |
  +---------------------+---------------------------------------------+--------------------------------------------------+


In the example earlier, we see that the user ``svensson`` has all the rights to his directory, so does the group ``system:administrators``. The group ``system:anyuser`` which contains all the users of AFS in the whole world may read and list the files in this directory. Finally ``svensson``'s friend ``andersson`` has all the rights except the right to change the ACL. To alter the ACL you use ``fs setacl`` or ``fs sa`` like:

.. code-block:: bash

   > fs sa [directory] user rights

Assuming that you are in your home directory and that you want to give the user ``mysister`` some rights in this directory you could write

.. code-block:: bash

   > fs sa . mysister rliw

This would make it possible for ``mysister`` to read, list, create new files, and to modify existing files. By default ``fs sa`` adds to or alters the contents of the ACL instead of replacing it. To revoke the rights given to a user you must use t he the following command:

.. code-block:: bash

   > fs sa [directory] user none

Finally, to see all the available commands with ``fs`` use ``fs help``.

Protecting Your ``init`` files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: This has already been set up if you got a standard PDC account, check with ``ls -la ~``

Since the file protection is on the directory rather than file level, you cannot not have different levels of rights on the files in directory. Normally, this is not a problem, you just put the files you want to keep secret in a directory (often called ``Private``) and public files in another directory (called ``Public``). However this may pose a problem in your home directory which contains files that should be public readable such as ``.login``, ``.forward``, ``.tcshrc``, and others. If these files are not public readable, programs like rlogin will not function properly. You have also files that you don't want other to read, like the file ``mbox`` where your email is stored.

The trick to solve this is to make a public readable subdirectory containing the files. In your home directory you then create symbolic links to the se files. The links will allow you to read the files which now appear as public readable. You should not make your home directory public readable. One example to clarify the method;

Change to your home directory:

.. code-block:: bash

   > cd

Create a subdirectory:

.. code-block:: bash

   > mkdir Public

Make this subdirectory readable by any user:

.. code-block:: bash

   > fs setacl Public system:anyuser read

Move all the files that should be public readable to this directory:

.. code-block:: bash

   > mv .login .cshrc .tcshrc .forward Public

Create the links:

.. code-block:: bash

   > ln -s Public/.login .
   > ln -s Public/.cshrc .

and so on...

Creating and managing groups
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Every user in the AFS system can create groups of users. All the members can then be given the same access rights by adding the group to an ACL. This is a very convenient way of giving the same rights to a group.

In the ACL, you recognise groups if they are in a format ``owner:groupname``, in the example earlier in this document we see the group ``system:anyuse``. This is one of the systems groups of which the most important are:

* ``system:anyuser`` This is all the users of AFS all over the world.
* ``system:authuser`` This is all the local users of AFS.
* ``system:administrators`` This is the group of systems administrators, they have all the rights to all your directories, regardless what you define in your ACL.

To create your own groups, use the command ``pts`` as follows:

* Create a new group with ``creategroup`` or ``cg``, owner should be your username

.. code-block:: bash

   > pts creategroup owner:groupname

* Add a user to a group with ``adduser`` or ``ad``

.. code-block:: bash
   
   > pts adduser user owner:groupname

* Deletes a group with ``delete`` or ``del``

.. code-block:: bash
   
   > pts delete owner:groupname

Removes one user from the group with ``removeuser`` or ``rem``

.. code-block:: bash
   
   > pts removeuser user owner:groupname

Lists the members in a group with ``membership`` or ``m``.

.. code-block:: bash
   
   > pts membership owner:groupname

List all commands to ``pts`` with ``help``

.. code-block:: bash
   
   > pts help

.. rubric:: Example
   
Here is an example, assume that you have two friends svensson and andersson. You want to give them certain rights in a directory called my_secrets. Yor own username is me. First in your home directory, you create the group friends:

.. code-block:: bash
   
   > cd
   > pts creategroup me:friends

Then you should add the users to the group

.. code-block:: bash
   
   > pts adduser svensson me:friends
   > pts adduser andersson me:friends

All we have to do now is to add this group to the ACL for the directory my_secrets. Assuming that my_secrets are a subdirectory under your home dire ctory you would type:

.. code-block:: bash

   > fs setacl my_secrets me:friends rlidw

which would let members of the group friends read, list, insert, delete and write files in your directory. You use fs setacl in the same way for users and groups, just remember that a group is written as owner:groupname.

Then you should add the users to the group

.. code-block:: bash
   
   > pts adduser svensson me:friends
   > pts adduser andersson me:friends

All we have to do now is to add this group to the ACL for the directory my_secrets. Assuming that my_secrets are a subdirectory under your home dire ctory you would type:

.. code-block:: bash
   
   > fs setacl my_secrets me:friends rlidw

which would let members of the group friends read, list, insert, delete and write files in your directory. You use fs setacl in the same way for us ers and groups, just remember that a group is written as owner:groupname.


Accessing other cells
^^^^^^^^^^^^^^^^^^^^^

If you want to access files that are located somewhere else, e.g. your home directory at another institution that uses AFS, you need to acquire tokens for that cell (unless the files you want are readable by anyone, in which case you don't have to do anything special). This is done by first getting Kerberos tickets for the corresponding realm and then getting tokens from those tickets using the command afslog.

As an example, assume that you have an account ``user@PHYSTO.SE`` with the home directory ``/afs/physto.se/home/u/user``. First you need to get Kerberos tickets:

.. code-block:: bash
   
   > kauth user@PHYSTO.SE

Then you need to acquire tokens:

.. code-block:: bash
   
   > afslog -c physto.se

You should now be able to read and write the files in ``/afs/physto.se/home/u/user``.

Disk usage and quota
^^^^^^^^^^^^^^^^^^^^

How much space do you have in your home directory? And how much space is already used? You can find out in the following ways:
	    
To see the size of single files (NOT directories in AFS):

.. code-block:: bash
   
   > ls -lh

Check your current overall usage:

.. code-block:: bash
   
   > du -hs ~/*

and WAIT! It will take some time to get the total size of each folder in your home directory.

.. code-block:: bash
   
   > fs lq directory_name

will list the quota of for the given directory. For example:

.. code-block:: bash
   
   > fs lq ~

In AFS there are two aspects of your storage that are limited - KB of disk space and the number of files you can create in a certain folder.


Maximum number of files
^^^^^^^^^^^^^^^^^^^^^^^

The maximum number of files in an AFS directory is 64435 (if the file names are short, otherwise the number is less). If you try to create one more file than that, you will get an error message.

.. code-block:: bash
   
   File too large

OpenAFS has a very slow algorithm for accessing files in a directory with many files. So it's not practical having more than a few thousand files in a directory. Recommended is instead to group the filenames in different directories or create larger files.

Check the status of an AFS server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
	    
If you are suspecting that the AFS server you are using is overloaded you can check this.

You can check if an AFS file server is overloaded. First find out on what file server your directory is located:

.. code-block:: bash
   
   > module add afsws
   > fs whereis ~

This will return a host name for your home directory, ~, for instance sculpin.pdc.kth.se. Now, get some information from that host:

.. code-block:: bash
   
   > rxdebug sculpin.pdc.kth.se | head -5 | tail -2

An output might be:

.. code-block:: bash
   
   > 0 calls waiting for a thread
   > 122 threads are idle

Those values corresponds to the normal healthy condition of an AFS file server with not so high load. But if you on the other hand would see:

.. code-block:: bash
   
   > 500 calls waiting for a thread
   > 2 threads are idle

then the AFS server is on a high load which will make everything go very slow. 

.. seealso::
   
 `Official OpenAFS user guide <http://docs.openafs.org/UserGuide/>`_