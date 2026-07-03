# My Homelab Doc

> I think the best way to learn something is by building your own

---

# About this Repository

This repository documents my journey of building a complete and ideal homelab from scratch.

The goal is not only to self-host useful services but also to understand how modern infrastructure works behind the scenes plus to give convinience to my family and friends if possible

Instead of following tutorials blindly, I want to build every layer involved with ofcourse help of internet

By the end of this project I want to confidently build Dexter`s lab(jk)

---

# Why I'm Doing This

When i own something i want to own it completely not rent it not partially own it, i would want it to be mine completely.

A homelab provides a safe environment where I can:

- do and alter things to my convinience
- break things
- fix them
- experiment
- automate
- monitor
- attack
- defend

without risking real systems.

This repository is my documentation of everything I learn.

---

# Hardware (CURRENT)

Laptop 1

- Intel Core i5
- 8 GB RAM
- Internal SSD/HDD 1TB

Laptop 2

- AMD Ryzen7
- 16 GB RAM
- Internal SSD/HDD 1TB

RaspberryPI Zero W

---

# Current Architecture
**Current Architecture is on current-architecure.md**

---

# Why Proxmox?

Watched lots of yt before starting this project and that installing every service directly onto Linux eventually becomes messy.

Essentially it becomes the operating system that manages all other operating systems.

---

# Why OpenMediaVault?

Currently using OMV because it is lightweight and easy but i will be migrating to trueNAS once hardware allows it but for the time being it is an easy tester

---

# Current Progress

## Phase 1 -- Beginning

### Installing Proxmox

Installed Proxmox with USB stick.

Problems encountered:

- Not really a problem but you have to flash the iso with DD mode for proxmox and when its done the usb stick becomes unrecognizable for windows and i thought it broke completely so i spent a good 30 minutes just figuring out why is that and learned it was completely fine

---

### Installing openmediavault

It was fairly simple you just have to learn the proxmox UI. For example to upload the iso image from your pc and unmount the CD/DVD.

Problems encountered:

- Again not really a problem but because i didnt know you have to unmount the disk to prevent installing itself again and again

### Configuring OMV 

- Configured OMV to have SMB and SSH connections, storage, file system and a user


# Next Step:

## Phase 2 -- Containers


### Goal:

Build apps

- media Server
- cloud Server
- password manager (probably) 

---

# Troubleshooting Journal

Every time something breaks I document:

- Problem

- Cause

- Solution

- What I learned

---

# Main Goal

This homelab is more than a collection of self-hosted services.

It is my personal learning environment where I can safely explore virtualization, Linux, networking, cybersecurity, automation and cloud technologies.

The objective is not to build the biggest server, but to understand every component well enough that I can design, deploy and troubleshoot similar systems in real-world environments.