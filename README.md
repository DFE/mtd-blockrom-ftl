mtd-blockrom-ftl
================

The blockrom MTD driver provides bad block free read access to MTDs via /dev/blockromX device files. Bad blocks are automatically skipped upon read, and reading continues with the next good block.