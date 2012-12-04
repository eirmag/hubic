hubic
=====

Hubic is the french OVH cloud storage solution. They do provide a davfs access to their system, with no official support. There is bunch of scripts (perl, python) which circulate online, but kind of lack of support, as hubic is not yet stable.

Here is my small contributions, reusing services exposed in ws.ovh.com, and javascript API written by OVH.

Credentials
-----------

* getCredentials.html provide a web page that you can download and execute offline. You retrieve from there the mouting point, username and password to use in the davfs.

Mouting
-------

    $ mount -t davfs https://yourid.wd.hubic.ovh.net /media/hubic
    Please enter the username to authenticate with server https://yourid.wd.hubic.ovh.net or hit enter for none.
    Username: {what you get in the html}
    Please enter the password to authenticate user  with server https://yourid.wd.hubic.ovh.net or hit enter for none.
    Password: 
    $ umount /media/hubic

Backup
------
The best backup solution I've seen so far is combination of rsync and gzip:
    rsync -avz --progress --copy-links --keep-dirlinks --exclude-from=logs.exclude.txt /var/log/ /media/hubic.plain/logs

    GZIP="--rsyncable" tar zcvf /tmp/files.tgz /home/gabriel/files/
    rsync /tmp/files.tgz /media/hubic.plain/

The rsyncable option of gzip (here used directly within the tar) is really usefull to reduce transfer size of big zipped files. Check https://beeznest.wordpress.com/2005/02/03/rsyncable-gzip/ for more details.

Encryption
----------

One can use the system to systematically encrypt some part of the remote storage. A convinient solution is to use Encfs. Here are some quick reminders.

    $ encfs /media/hubic/remote/private /media/hubic.plain/
    Creating new encrypted volume.
    Please choose from one of the following options:
    enter "x" for expert configuration mode,
    enter "p" for pre-configured paranoia mode,
    anything else, or an empty line will select standard mode.

    Standard configuration selected.

    Configuration finished.  The filesystem to be created has
    the following properties:
    Filesystem cipher: "ssl/aes", version 3:0:2
    Filename encoding: "nameio/block", version 3:0:1
    Key Size: 192 bits
    Block Size: 1024 bytes
    Each file contains 8 byte header with unique IV data.
    Filenames encoded using IV chaining mode.
    File holes passed through to ciphertext.

    Now you will need to enter a password for your filesystem.
    You will need to remember this password, as there is absolutely
    no recovery mechanism.  However, the password can be changed
    later using encfsctl.

    New Encfs Password: 
    Verify Encfs Password:
    $ sync
    $ fusermount -u /media/hubic.plain/

It binds the local folder '/media/hubic.plain' to the remote folder 'remote/private'. All data written to '/media/hubic.plain' will be transparently encrypted, and you can look at the impact on 'remote/private' folder, or on the hubic interface. Next time you mount the encrypted filesystem again, you will only have to type in the password.

Below, when you want to restore the encrypted content of one file (not all the encrypted content), you can copy the encrypted file in a temporary folder, and apply the encfs configuration located in an xml file (this file is automatically created and located at the root of the encrypted content):

    $ ls -a /media/hubic/remote/private/
    . .. .encfs6.xml hFJxMuTdOVyc7qeuUdtKhxF
    $ ENCFS6_CONFIG=./encfs6.xml encfs /tmp/test/crypt-view /tmp/test/plain-view
 
