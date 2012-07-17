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

