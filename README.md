mtd-blockrom-ftl
================

The blockrom MTD driver provides bad block free 
read access to MTDs via /dev/blockromX device files. 
Bad blocks are automatically skipped upon read, 
and reading continues with the next good block.

This allows for read-only filesystems to exist in MTDs with bad blocks:
usually the baddies are skipped when writing the filesystem into MTD by
the tool performing the write (e.g. nandwrite). To successfully mount
these filesystems, however, you will need a translation layer that skips
bad blocks upon read as well. blockrom is such a translation layer.

Working with /dev/blockromX devices
===================================

Blockrom will create one device node per MTD - or, if MTD partitioning support
was selected as well, it will create one node per MTD partition.

You cannot write to these devices. So in order to put something there your
kernel will also require to support some means to write to MTDs; I recommend
using the MTD character access driver for this.

Installing a file system
------------------------
- First, erase the corresponding MTD (mtd3 in this example). The `flash_erase` 
  tool is from the `mtd-utils` package of the linux-mtd project.

    flash_erase /dev/mtd3 0 0

- Write the file system image (squashfs in this example) to the mtd char 
  device, skipping bad blocks as you write. Since this example writes to NAND
  flash it uses `nandwrite`, which also is from `mtd-utils` of linux-mtd.

    nandwrite -m -p /dev/mtd3 fs-image.squashfs

Mouning blockrom
-----------------
- You can now mount the corresponding blockrom block device. This example
  assumes that you wrote a squashfs to mtd3 (like in the example above):

    mount -t squashfs /dev/blockrom3 /mnt/mysquash

Installation
============
- copy the driver source into your kernel sources:

    cp mtd-blockrom-ftl/src/drivers/mtd/* <where-your-kernel-is>/drivers/mtd

- apply the glue logic patch to your kernel

    cd <where-your-kernel-is>
    patch -p1 < <where-you-downloaded-blockrom>/mtd-blockrom-glue.patch

- run `make menuconfig` or similar and select the blockrom ftl

    make menuconfig
    drivers->
           mtd->
              [*] Bad-block-skipiping readonly 'blockrom' access to MTD devices

- compile, deploy your kernel

- done

Todo
====
blockrom.c lacks of a handler for "corrected read errors". These happen e.g. if 
a one-bit CRC error occurred and was corrected. According to some posts at the
linux-mtd mailing list thes errors may occur even if a flash device is only
ever read from, never written to.
Upon encountering such an error the driver is supposed to "bit scrub" the
corresponding erase block. This basically means moving the EBs data some place
else, and erasing the erroneous block. In order to safely support such an 
operation the blockrom driver needs to apply some kind of translation table
when looking up block addresses and offsets.

There's a stub in the driver source where a bit scrubbing routine would go in,
as well as a kernel config option for enabling it (if you enable it right now
your kernel build would fail, though, since it's not yet implmented).


Tests
=====
The `tests/` sub directory has unit tests for blockrom.c.  You don't need a
kernel for building or running the tests, they're self-contained.

You _do_ need `test_harness.h`, a tiny collection of mocking tools for writing
C unit tests, though. You can obtain it from https://github.com/t-lo/test_harness.
Just put the .h file into the `tests/` sub directory and you should be good to
go. In order to build and run the tests just issue `make` in the `tests/` sub
directory.



