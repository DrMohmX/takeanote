---
layout: post
title: Chapter 1.1 - Storage Management
subtitle: 1/2 - Storage Devices and Partitions.
tags: [test, rhel, exam, book]
gh-badge: [star, fork, follow]
comments: true
---
Hello, dear readers.

In this post I will try to explain the **storage devices**:
1. manage (create partitions, extend existing ones),
2. interaction
3. monitor

# Intro
First let's start from data partitions. Data is stored on disks that are logically divided into partitions. Partitions themselves can be stored on one or multiple disks. Partitions are like pieces of a cake (disk) that are dividied to work as smaller "disks". It helps you to save money and sure, so called "zero-block elimination"
> Zero-block elimination is also called "thin provisioning" and I will cover it later

Partition information ("partition table information", sometimes) is stored at special disk locations that the system references at a boot time. **There are plenty of tools** RHEL offers you in order to manage partitions.

One of the key structure of all modern storage system tools is ***thin provisioning***. 2 reasons why "thin provisioning" is good and need to be used: it allocates only what **needed** and *storing* data at **adjacent** locations.
## Storage Management
Several partitions can exist in a single disk. The information about these several partitions is stored on the disk (booable disk, 446 bytes (Bootloader - Grub2) + 64 bytes (MBR/GPT Part Table info) + 2 bytes (Signature) = 512 bytes)
