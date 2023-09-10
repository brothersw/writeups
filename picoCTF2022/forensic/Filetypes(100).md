**Description**

Author: Geoffrey Njogu

This file was found among some files marked confidential but my pdf reader cannot read it, maybe yours can. You can download the file from here.

Hints
1) Remember that some file types can contain and nest other files

**Solution**

The file extension listed on a file may not be the actual nature of that file, you could easily rename a .jpg to a .exe file. It wouldn't work as the type of file it is renamed as, so we need to find what type of file it actually is.

[File signiatures](https://en.wikipedia.org/wiki/List_of_file_signatures), or magic bytes are a way that programs list what type of file they. They often reserve the first few spots in the program to designate what they are. Not all programs do this however, so it will not always work.

Some documents are in plain text, and decoding them as ASCII like any text editor would do is a way to find out the file as well.

In this case, the file was a plain text file and the contents below can be opened in your favorite text editior.
This is the first few lines, as it is all we need:

```sh
#!/bin/sh
# This is a shell archive (produced by GNU sharutils 4.15.2).
# To extract the files from this archive, save it to some FILE, remove
# everything before the '#!/bin/sh' line above, then type 'sh FILE'.
#
lock_dir=_sh00048
# Made on 2022-03-15 06:50 UTC by <root@ffe9b79d238c>.
# Source directory was '/app'.
#
# Existing files will *not* be overwritten, unless '-c' is specified.
#
# This shar contains:
# length mode       name
# ------ ---------- ------------------------------------------
#   1092 -rw-r--r-- flag
#
```

This is a shell archive which is a type of linux command file. We can execute this in the terminal after using ```chmod +x Flag.pdf``` to give us executable permissions on this file. I suggest doing this in an empty directory with only Flag.pdf

It creates another archive. From now on, you can use the magic bits of the file, or use the file tool to determine what type of archive is extracted. 

i.e ```file flag``` will give you the type of file

I heavily recommend using [7zip](https://7zip.info/) to extract these files as it is a great all in one extraction tool that can cover many protocols.

```file flag``` lets us know that this is a ar archive, which we can extract with

```7z e flag```

This command creates a new file also named flag, which is a cpio archive. Now that we know the drill, repeat ```file <filename>``` and ```7z e <filename>``` to extract.

Here is the list of files made

flag: cpio archive

flag: bzip2 compressed data, block size = 900k

flag~: gzip compressed data

flag: lzip compressed data, version: 1


Lzip cannot be extracted with 7zip, instead use the lzip tool

```lzip -d flag```

flag.out: LZ4 compressed data (v1.4+)

LZ4 canot be extracted with 7zip, instead use the lz4 tool

```lz4 -d flag.out flag```

flag: LZMA compressed data

flag~: lzop compressed data


lzop cannot be extracted with 7zip, instead use lzop tool

```lzop -x flag~```

flag: lzip compressed data


Lzip cannot be extracted with 7zip, instead use the lzip tool

```lzip -d flag```

flag.out: XZ compressed data

flag: ASCII text


Finally after that long line of layered compression algorithms, we have our flag
...Almost

```
7069636f4354467b66316c656e406d335f6d406e3170756c407431306e5f
6630725f3062326375723137795f33343765616536357d0a
```

This hex translates to our flag!
It is: ```picoCTF{f1len@m3_m@n1pul@t10n_f0r_0b2cur17y_347eae65}```
