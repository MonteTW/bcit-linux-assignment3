This is an instruction to help you who has a droplet with Arch Linux operating system to implement the modules `nginx` as the server, and `ufw` as the firewall on your droplet with a script that will automatically generate a html file that displays some basic information of the droplet's system and hold it on the IP address of your droplet.

## Prerequisites
1. Update your operating system by running
```bash
sudo pacman -Syu
```

2. Install `nginx` module
```bash
sudo pacman -S nginx
```

3. Install `ufw` module
```bash
sudo pacman -S ufw
```

## Step 1: Create a System User

Create a user `webgen` with the command:
```bash
sudo useradd -r -m -d /var/lib/webgen -s /usr/bin/nologin webgen
```
- `-s /usr/bin/nologin` is a non-login user
- `-m -d /var/lib/webgen` is making a home directory at the path of `/var/lib/webgen`

**why?**

## Step 2: Create Certain Directories 
Use `mkdir` to create the directories `bin` and `HTML` inside `/var/lib/webgen` to make the tree as below:
```
/var/lib/webgen/
├── bin
└── HTML
```

## Step 3: `git clone` the Files from This Repository 
Clone the files from this repository to `/var/lib/webgen/bin`
```bash
cd /var/lib/webgen/bin
git clone <Repository URL>
```

## Step 4: Make the Files Executable and Change Ownership
Make the files in `/var/lib/webgen/bin` executable
```bash
sudo chmod 755 *
```

Change the ownership of `/var/lib/webgen` and all the subdirectories and files to user webgen
```bash
sudo chown -R webgen:webgen /var/lib/webgen
```

## Step 4: .service File
Copy the service file to `/etc/systemd/system/`
```bash
sudo cp /var/lib/webgen/bin/generate-index.service /etc/systemd/system/
```

Run the file and test it
```bash
sudo systemctl start generate-index
```

Check the status of generate-index
```bash
systemctl status generate-index
```

Check if there's an `index.html` generated inside `/var/lib/webgen/HTML`

## Step 5: .timer File
Copy the service file to `/etc/systemd/system/`
```bash
sudo cp /var/lib/webgen/bin/generate-index.timer /etc/systemd/system/
```

Reload the unix file
```bash
 sudo systemctl daemon-reload
```

Run and enable the .timer file
```bash
sudo systemctl start generate-index.timer
sudo systemctl enable generate-index.timer
```

Check the status of `generate-index.timer`
```bash
systemctl status generate-index.timer
```

Check the timer is counting at the background, and check the time left from now to 5 am is correct
```bash
systemctl list-timers
```

## Step 6: `nginx` Configuration

Replace the `/etc/nginx/nginx.conf` file by `nginx.conf` file
```bash
sudo rm /etc/nginx/nginx.conf
sudo cp /var/lib/webgen/bin/nginx.conf /etc/nginx/
```

Create two specific directories for server block
```bash
sudo mkdir /etc/nginx/sites-available
sudo mkdir /etc/nginx/sites-enabled
```

Copy `index.conf` to `/etc/nginx/sites-available`
```bash
sudo cp /var/lib/webgen/bin/index.conf /etc/nginx/sites-available/
```

Test the config file
```bash
sudo nginx -t
```

Start and Enable
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

Check the status of `nginx`
```bash
systemctl status nginx
```

>**Why is it important to use a separate server block file instead of modifying the main `nginx.conf` file directly?**
>
>By putting different `server` blocks in different files. This allows you to easily enable or disable certain sites.
>
>we can disable a site by unlink the active symbolic link:
>`unlink /etc/nginx/sites-enabled/index.conf`

Check the server is working as you want by put the IP address of your droplet to a browser to make sure the page that display your system information is implemented

## Step 8: `ufw` configuration

Enable and start the service of `ufw`
```bash
sudo systemctl enable --now ufw.service
```

Allowing `ssh` and `http`
```bash
sudo ufw allow ssh
sudo ufw allow http
```

Turn on the Firewall
```bash
sudo ufw enable
```

Check the firewall is on and working
```bash
sudo ufw status verbose
```

## Extra: Script Enhancement
