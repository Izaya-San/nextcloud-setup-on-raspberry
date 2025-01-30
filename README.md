# Nextcloud setup on Raspberry
## Table of content
- [Nextcloud setup on Raspberry](#nextcloud-setup-on-raspberry)
  - [Table of content](#table-of-content)
  - [Description](#description)
  - [Disclamer](#disclamer)
  - [Configuration](#configuration)
  - [Setup](#setup)
  - [Other documentations](#other-documentations)
  - [Links](#links)

## Description
This repository describes a step-by-step installation of Nextcloud on a Raspberry Pi and a RAID 5 with Ubuntu Server, Apache, PHP and MySQL. It also explains important concepts to understand the commands and covers everything from the physical installation of the OS on the Raspberry Pi, the linux users setup, the SSH certificat setup, a full RAID 5 setup for the data storage, the Apache configuration, etc. You'll also find explanations on how to back up your data, how to restore your nextcloud server from backups, how to upgrade your Nextcloud version, troubleshooting notes, etc.

## Disclamer
This content is my own understanding from the experience I have. You should always refer to the official documentation of the tools you use. I just want to give peace of the knowledge I've gathered during the past years on this topic, but I'm not responsible for what you do, and you should always understand what you're doing.

## Configuration
Hardware:
- Raspberry Pi 5 Model B
  - 8 GB of RAM
  - Plugged to a home router by RJ45 cable
  - Disk: SSD disk (can be an SD card, but I discourage it because it's not reliable and will easily break, corrupt your data, etc.)
- Data storage:
  - You can store your data anywhere, even on a single disk or directly on your SSD.
  - For a minimum of data safety I've opted for a RAID 5 with 3 identical disks plugged on the Raspberry. Note that if you want to use hard drive disks, keep in mind that these disks needs to be powered.
- Optional:
  - APC back-UPS powering the disks and the Raspberry. The data cable is also plugged to the Raspberry. This is completly optional, I'm using it because the power supply on my network is sometimes unstable, so it's nice to have it protection against power fluctuations and a battery to keep the server online during a power outage.

Softwares & OS:
- Ubuntu server 24.04.1 LTS
- Nextcloud 29.0.10
- Apache 2.4.58
- PHP 8.1.31
- MySQL 8.0.40

Other:
- Your Domain Name and access to its configuration
- Access to your router configuration

## Setup
1. OS install on the Raspberry Pi. [See](./Raspberry/README.md)
2. Linux users. [See](./Linux/Users.md)
3. SSH configuration and certificats. [See](./Linux/SSH.md)
4. RAID 5 setup, if you have one. [See](./RAID5/README.md)
5. Apache setup. [See](./Apache/README.md)
6. Nextcloud setup. [See](./Nextcloud/README.md)
7. APC backup-UPS setup, if you have one. [See](./APC%20backup-UPS/README.md)

## Other documentations
- Backup and Restore your Nextcloud server. [See](./Nextcloud/Backup.md)
- Upgrade Nextcloud. [See](./Nextcloud/Upgrade.md)
- Nextcloud server troubleshooting. [See](./Nextcloud/Troubleshooting.md)

## Links
- [Nextcloud official latest documentation](https://docs.nextcloud.com/server/latest/admin_manual/contents.html). To get an older version of the documentation, change the `latest` in the URL by your Nextcloud version number, like `29`.
- [Ubuntu server documentation](https://ubuntu.com/server/docs)
- [Raspberry Pi documentation](https://www.raspberrypi.com/documentation/computers/getting-started.html)
