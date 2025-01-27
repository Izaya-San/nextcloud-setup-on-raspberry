For a basic install, you can use your main user with the sudo permissions to do everything (here `usudo`), and it'll work. If you choose to do that, simply ignore this part en always use this user for the next tutorials when a user is specified.

# Quick permissions explanations
By default, Linux goes with a superuser with all powers named `root`, that you don't have access to, and it's a good thing for security. The only way you have to execute commands as a root user is to user the `sudo` command, which basically says "I would like to execute my command as if I'm root". This `sudo` command can be given or not to the other users, effectively restricting a user power. When executing a command with `sudo`, you'll be asked the current user's password. The use created by default with your Ubuntu Server always have this power.
Another useful command is `su` which allows switching from your current user to another one by giving this user's password. You can remove access to this command to specific users, but it's a bit of a risky task, so I won't do it here.

# Users created
I wanted to improve the permissions seggragtion by creating several users:
- ussh: user used to log in on our sever via ssh. It'll be our entrance door. It won't have `sudo` permission
- usudo: the existing user created on Ubuntu Server by default that have `sudo` permission
- unextcloud: optional user I'll use for nextcloud directory creation, without `sudo` permission. I considered it a bit better to have it, but you can really jus use the usudo to do everything instead

# Create users
To run as usudo
```sh
# If needed, change the current user password
passwd

# Create other users
sudo adduser ussh
sudo adduser unextcloud
```

# Restrict sudo permission
To run as usudo
```sh
# As usudo, check the list of users with sudo permission, it should only returns the user usudo
getent group sudo
# If ussh or unextcloud have the sudo permission, remove it with:
sudo deluser ussh sudo
```
