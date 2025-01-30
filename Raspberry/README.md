# Table of content
- [Table of content](#table-of-content)
- [General note](#general-note)
- [Hardware requirements](#hardware-requirements)
- [Install on SSD](#install-on-ssd)
- [Install on SD card](#install-on-sd-card)

# General note
The starting point is obviously to have an OS up and running on your server. We'll work here on a `Raspberry Pi 5 Model B`. Here, I explain 2 ways to install the OS: one on the SD card plugged in the Raspberry Pi, which is the easiest installation but also the less reliable, the other on an SSD which is much more robust in the long run and more performant, but also requires a bit more setup and expenses. I've used `Ubuntu Server 24.04.1 LTS`. The rest of this tutorial was done on this peculiar OS and raspberry model, but you can choose any hardware and OS you like, just keep in mind that you'll need to adapt this tutorial to your peculiar setup.

# Hardware requirements
In all cases, you'll need a Raspberry Pi with its power supply, an RJ45 cable and an SD card. It's recommended adding a [cooling system](https://www.raspberrypi.com/products/active-cooler/). There also are protecting [cases](https://www.raspberrypi.com/products/raspberry-pi-5-case/).

If you're installating your OS on an SSD, you'll need an SSD HAT and a SSD compatible with your Raspberry Pi. In my case I used the [M.2 HAT+](https://www.kubii.com/fr/raspberry-pi-5/4114-m2-hat-plus-5056561803463.html) with the [SSD NVMe M.2 2280/2242](https://www.kubii.com/fr/support-de-stockage/4260-disque-dur-ssd-nvme-m2-2280-makerdisk-3272496317512.html), but you can choose what the best feats your needs. Be aware if you use a protecting case that it's compatible with your HAT and your cooling system.

# Install on SSD
1. Assemble all the elements of your Raspberry Pi as explained in [the other section](#install-on-sd-card). You should also add your SSH HAT and plug the SSD on the HAT. Plugging the SSD HAT PCIE cable is done by lifting the protection. If you prefer a graphical interface for this installation step, which is more beginner-friendly, you'll install Ubuntu Desktop on the SD Card and plug, for the first steps of the installation only, a keyboard, a mouse and a monitor. Else, you can install Ubuntu Server on the SD Card and do everything from command line, which means you'll need to write the OS image on the SSD from the command line instead of using Raspberry Pi Imager (I haven't tested it myself, but you can find good tutorials about it on the web).
2. Install Ubuntu Desktop or Server on the SD Card as described in [the other section](#install-on-sd-card).
3. When your Raspberry Pi is running Ubuntu Desktop correctly from the SD Card, you can install [Raspberry Pi Imager](https://www.raspberrypi.com/software/) to install Ubuntu Server on the SSD card as we've done before for the SD Card. If you've choosen to install Ubuntu Server on the SD Card, you'll need to install Ubunut Sever on the SSD from the command line following.
4. Reboot your Raspberry Pi and remove the SD Card; that should lead to a boot sequence on the SSD that'll launch the Ubuntu Server installation. Wait until Ubuntu Server installation is done and ready.
5. You should be able to log in with SSH from a client computer as usudo as we've done in [the other section](#install-on-sd-card).
6. Run an OS update with `sudo apt update && sudo apt upgrade`

Congratulation, you have a running Ubuntu Server on your Raspberry Pi accessible in SSH for any computer in the same local network.

# Install on SD card
There is an [official tutorial](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview) about this installation, and I'll explain it in my terms here.

1. Assemble all the elements of your Raspberry Pi: plug the SD Card in it, add your cooling system if you have one and plug an RJ45 cable. Don't plug the power cable for now.
2. On your personnal computer, plug your SD Card to your computer. Then install [Raspberry Pi Imager](https://www.raspberrypi.com/software/) and launch it, select your SD Card in the Imager interface and select the OS you want, here we'll use a `Other general purpose OS` / `Ubuntu Server 24.04.1 LTS`. Don't forget to `Edit settings`, that will give you advanced options we'll use: override the `hostname` to your liking, check the `Enable SSH` with `Use password authentication` and provide as `username` "usudo" and the strong password you want. Finally, write the OS on the SD Card.
3. When finished, unplug the SD Card from your personnal computer and plug it into the Raspberry Pi. Be sure the RJ45 cable is plugged and power on the raspberry.
4. Wait until Ubuntu Server installation is done and ready.
5. You should be able to log in with SSH from a client computer as usudo. For this, use a client computer connected in the same local network as your Raspberry Pi, search for the Raspberry Pi local IP address (which should be given by your router administrator interface) and run from your client computer this: `ssh usudo@myLocalIP`. You'll be prompted to enter the usudo user password, and you should be logged in.
6. Run an OS update with `sudo apt update && sudo apt upgrade`

Congratulation, you have a running Ubuntu Server on your Raspberry Pi accessible in SSH for any computer in the same local network.
