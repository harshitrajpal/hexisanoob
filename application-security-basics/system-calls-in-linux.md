# System Calls in Linux

<figure><img src="../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

**Hard Links**: In the Linux operating system, a hard link is equivalent to a file stored in the hard drive – and it actually references or points to a spot on a hard drive. A hard link is a mirror copy of the original file.\
Every file on the Linux filesystem starts with a single hard link. The _link_ is between the filename and the actual data stored on the filesystem. When changes are made to one filename, the other reflects those changes. The permissions, link count, ownership, timestamps, and file content are the exact same. If the original file is deleted, the data still exists under the secondary hard link. The data is only removed from your drive when all links to the data have been removed. If you find two files with identical properties but are unsure if they are hard-linked, use the `ls -i` command to view the _inode_ number. Files that are hard-linked together share the same inode number.\
\
$ ln link\_test /tmp/link\_new\
\[tcarrigan@server demo]$ ls -l link\_test /tmp/link\_new \
\-rw-rw-r--. 2 tcarrigan tcarrigan 12 Aug 29 14:27 link\_test \
\-rw-rw-r--. 2 tcarrigan tcarrigan 12 Aug 29 14:27 /tmp/link\_new



**Soft Links**: Commonly referred to as _symbolic links_, soft links link together non-regular and regular files. They can also span multiple filesystems. By definition, a soft link is not a standard file, but a special file that points to an existing file.\


ln -s /home/tcarrigan/demo/soft\_link\_test /tmp/soft\_link\_new&#x20;

\[tcarrigan@server demo]$ ls -l soft\_link\_test /tmp/soft\_link\_new \
&#x20;\-rw-rw-r--. 1 tcarrigan tcarrigan 17 Aug 30 11:59 soft\_link\_test \
lrwxrwxrwx. 1 tcarrigan tcarrigan 35 Aug 30 12:09 /tmp/soft\_link\_new -> /home/tcarrigan/demo/soft\_link\_test



**inodes**: On the filesystem, each file is identified by a unique number (an inode number). The link system call just creates another name in the filesystem with the same inode.

When you open a file htrough an application, it points to inode which points to data on the disk and that's how a file is opened and operated upon.



