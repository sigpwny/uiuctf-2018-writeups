In this challenge, we're given an image and an archive.  

When we try to extract the archive using 7z, we see that is protected by a password. 

Let's see if the image can help us.  

In images, there is metadata in the Exif standard. We can look at this data using a tool called exiftool. There might be information that can help.

```
exiftool cute_dino.jpg 
ExifTool Version Number         : 10.80
File Name                       : cute_dino.jpg
...
Document ID                     : xmp.did:788A2A26D02511E7A0AEC879B62BAB1D
Derived From Instance ID        : xmp.iid:788A2A23D02511E7A0AEC879B62BAB1D
Derived From Document ID        : xmp.did:788A2A24D02511E7A0AEC879B62BAB1D
Comment                         : the password is dinosaursarecool
Image Width                     : 900
Image Height                    : 560
...
Megapixels                      : 0.504
```

Sure enough, there is a comment containing the password! We can use this password to extract the archive. 

```7z e secure_disk.7z -ochallenge```

The command above will extract the archive to the "challenge" directory after we type in the password. 

We now have a copy of the raspberry pi image. 

```
file rsp.img 
rsp.img: DOS/MBR boot sector; partition 1 : ID=0xc, start-CHS (0x0,130,3), end-CHS (0x6,4,22), startsector 8192, 88472 sectors; partition 2 : ID=0x83, start-CHS (0x6,30,25), end-CHS (0xe2,104,6), startsector 98304, 3538944 sectors
```
We would like to mount this image so we can explore the files inside. 

If we look at the image with a utility called fdisk, we can see what's inside the image in a nice format (some of this information is already included in the file command seen above).  
```
fdisk -l rsp.img 
Disk rsp.img: 1.8 GiB, 1862270976 bytes, 3637248 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4d3ee428

Device     Boot Start     End Sectors  Size Id Type
rsp.img1         8192   96663   88472 43.2M  c W95 FAT32 (LBA)
rsp.img2        98304 3637247 3538944  1.7G 83 Linux
```
The second partition looks promising! Let's go ahead and mount that partition with the mount command. 

We need to do a few things before we can mount this. First, let's create a directory as a mount point. 

```mkdir pi_mount```

Now, we need to calculate the offset for mount. For the second partition, we can see that it starts at the value 98304 from the fdisk output.  
However, if we look at what it says above, the units are in sectors. Mount takes the offset in terms of bytes, so we need to figure out how many bytes there are until the partition starts.  
The tool shows us that each sector is 512 bytes, so we simply multiply 98304 by 512 to get the value we want, 50331648.  

Finally, we need to know the filesystem type for mount. The output from fdisk says Linux and that means the filesystem type would likely be ext4 since this is a modern Linux system.

We can use cfdisk to double-check.  
 ```cfdisk rsp.img```  
If you go down to rsp.img2, you will see the filesystem type is ext4. 

We also want to use a loop device since this is in a file.  

Our final command to mount the image is:  
```sudo mount -t ext4 -o offset=50331648,loop rsp.img pi_mount```

Now that we have the image mounted, let's go exploring. 

Since we know we're looking for the information for a user, a reasonable area to start would be in the /home directory. Inside, we see the pi directory. Let's see what's in there.  

```
ls -lsa
total 32
4 drwxr-xr-x 3 pi pi 4096 Sep  6 17:26 .
4 drwxr-xr-x 3 root   root   4096 Jun 26 19:17 ..
4 -rw-r--r-- 1 pi pi  220 Jun 26 19:17 .bash_logout
4 -rw-r--r-- 1 pi pi 3523 Jun 26 19:17 .bashrc
4 -rw-r--r-- 1 pi pi   86 Jun 26 19:17 myfavoritepasswords
4 -rw-r--r-- 1 pi pi   35 Jun 26 19:17 myfavoriteservers
4 -rw-r--r-- 1 pi pi  675 Jun 26 19:17 .profile
4 drwx------ 2 pi pi 4096 Jun 26 19:17 .ssh
```

Wow, it looks like we hit the jackpot!  

```
cat myfavoritepasswords myfavoriteservers 
I love to use the password ilovedinosaurs because I love dinos! I mean, who doesn't?!
I love to SSH into x.x.x.x [IP redacted]
```

Now we have the login server and a password. This seems promising! Let's look at everything else in the directory. We see a .ssh directory. This might contain some keys our target user uses to login.  

```
ls -lsa .ssh
total 16
4 drwx------ 2 pi pi 4096 Jun 26 19:17 .
4 drwxr-xr-x 3 pi pi 4096 Sep  6 17:26 ..
4 -rw------- 1 pi pi 3326 Jun 26 19:17 id_rsa
4 -rw-r--r-- 1 pi pi  735 Jun 26 19:17 id_rsa.pub
```
```
cat id_rsa id_rsa.pub 
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,D66DF062580250CED25078FC4C42BE42

2m[REDACTED]TEaR
-----END RSA PRIVATE KEY-----
ssh-rsa AA[TRUNCATED]nQ== dinolover
```

Sure enough, we have the keys we need to login to the server. We can use the comment at the end of the public key as the username and the password in the *myfavoritepasswords* file as the passphrase for the private key. 

With this information, our command becomes:
```
ssh -i id_rsa dinolover@x.x.x.x
```

After entering our passphrase, we get a shell on the server!

We can view what's in the directory:
```
dinolover@ctfd:~$ ls
flag
```
And finally print out the goods!

```
dinolover@ctfd:~$ cat flag
dino{maybe_i_should_stick_to_paleontology}
```

Overall, this was a pretty simple forensics challenge that required getting a password from the Exif data of an image, extracting an archive, mounting an image, exploring a filesystem, and using information found on that filesystem to login to a remote server. 
