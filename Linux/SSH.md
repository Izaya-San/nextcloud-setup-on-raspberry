# Table of content
- [Table of content](#table-of-content)
- [SSH setup](#ssh-setup)
  - [Instal SSH](#instal-ssh)
  - [Access your server from outside](#access-your-server-from-outside)
- [Change SSH port and restrict access](#change-ssh-port-and-restrict-access)
  - [Server setup](#server-setup)
  - [Access your server from outside](#access-your-server-from-outside-1)
- [Ease your SSH log in](#ease-your-ssh-log-in)
  - [SSH authorized keys](#ssh-authorized-keys)
  - [Client custom config](#client-custom-config)

# SSH setup
## Instal SSH
```sh
# Check there is not SSH server setup
sudo systemctl status ssh

# If not installed, install SSH
sudo apt update
sudo apt install openssh-server

# Check the openssh server is running
sudo systemctl status ssh
```

## Access your server from outside
From here, you should be able to log in from a client via SSH onto your server with any linux user we've created. We'll firstly try to log in from a client computer inside your local network.

On the server as usudo:
```sh
sudo apt install net-tools
# Run this to find the server local Ip which is the Ip given by your router to your server and which is typically something like 192.168.1.x. It's the adress of your server inside the local network
ifconfig
```

On your client computer connected to the same router as your server:
```sh
# Typical SSH login command from the client, change myLocalIP with your sevrer local ip we've just found
ssh usudo@myLocalIP

# Logout and check it with ussh
ssh ussh@myLocalIP
  # And try to change your current user when you're logged in
  su usudo
```

Then, open your router ports and try to access it from a client outside your local network. Your local network, typically your home network, has a public IP which identifies it from the outside. When a request is sent to this public IP, your router will receive it and choose what to do with it. In our case, we'll need to ask our router to send all TCP requests sent on the port 80 (HTTP), 443 (HTTPS) and 22 (SSH) to be redirected to our server. The process depends on your router.

When done, re-do the ssh log in commands you've done in local but from a computer outside your network (typically using your phone as a Wi-Fi hotspot in 5G) and by chaging myLocalIP with your public ip.

# Change SSH port and restrict access
## Server setup
To improve the security, you can change the SSH port from 22 to any other number between 1024 and 65536. This way, hackers which are used to trigger the port 22 to access your server will fail. Let say we're chaging it to 1224.

We'll also restrict SSH login to ussh only. Hence, to access the user with sudo powers you'll need to log in as ussh and then user the `su` command to switch to usudo. This further increase security.

```sh
# Edit the ssh config
sudo nano /etc/ssh/sshd_config
  # Change the value: "Port 22" by
  Port 1224
  # Add these lines to restrict ssh log in access to only ussh
  PermitRootLogin no
  AllowUsers ussh

# Restart SSH
sudo systemctl restart ssh

# On Ubuntu, you'll also need to
sudo systemctl edit ssh.socket
  # Change "ListenStream=22" by
  ListenStream=1224

# Restart SSH Socket
sudo systemctl restart ssh.socket

# Ask for the SSH status, it should say we're using the port 1224
sudo systemctl status ssh
```

## Access your server from outside
As we've done previously, we'll try to log in to our server from a client computer outside our local network.

On your client computer:
```sh
# Log in as ussh and switch to usudo
ssh ussh@myPublicIP -p 1224
  su usudo
# Logout

# Try again with usudo, it should fail
ssh usudo@myPublicIP -p 1224
```

# Ease your SSH log in
Useful on your personnal computer if you are logging in on your server regularly. This computer should be a trusted computer as it easy the access to your server.

The following explanations are for Linux computers, but you can find similar ways on other OS.

## SSH authorized keys
For now, we've logged in via username/password. Instead, you can switch to a certificate authantication.

On your client computer, you'll need a key pair for your authentication.
```sh
# Check you have one
cat ~/.ssh/id_rsa.pub

# If not, generate it with this (you can provide empty answers to each question and this will generate a key pair in ~/.ssh in the files id_rsa and id_rsa.pub)
ssh-keygen -t rsa -b 4096
```
Note: if you've chosen a passphrase, when running the log in command it will ask you to enter this passphrase. The passphrase decrypt your key before using it to log in. There is an SSH Agent that will ease the process by asking the passphrase once, and if you use your key another time the same client session, it won't ask for the passphrase again.

On your server as ussh:
```sh
cd
# Create the ssh directory and provide the right permissions
mkdir .ssh
chmod 700 .ssh
# Create the file wich contains authorized keys
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
# Add to this file a line with your public key (the one in your client computer at ~/.ssh/id_rsa.pub)
nano .ssh/authorized_keys
```

You should be abled to log in from your local computer without the need of entering the ussh password:
```sh
ssh ussh@myPublicIP -p 1224
```

## Client custom config
Instead of running the full command `ssh ussh@myPublicIP -p 1224` where we need to know the ip and the port, we can create a custom SSH configuration in which we'll save all this information.

On your client computer:
```sh
nano ~/.ssh/config
```
Write this:
```
Host rpi
    HostName myPublicIp
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
    Port 1224
    User ussh

Host *
    Port 22
```
It says, if I ask for `rpi` host, you'll use the port 1224, the username ussh, the ip is myPublicIp (to be changed with yours which looks like xxx.xx.xx.xx) and my credential file. If you use ssh for any other host, it'll use the port 22.

Then you'll be able to log in by simply writing:
```sh
ssh rpi
```
