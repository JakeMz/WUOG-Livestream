# WUOG-Livestream Maintenance
Instructions on how to reset the WUOG livestream

This explanation is going to have commands you need to run. The things you should type into the digital ocean console are highlighted in red.
Example:
```
$ echo hello
```
Only “echo hello” should be typed into the console, the “$” is there for reference because it is always the last character in our linux-based terminal displayed before user input.

# Regular Maintenance
For restarting the livestream, you must first access our Digital Ocean Droplet via the console and log in as the user "icecaster". Instructions on how to do this can be found in the ops transition doc. Currently there are two custom scripts in the droplet that make starting and restarting both icecast and nginx easy.

1. Enter the digital ocean droplet console as icecaster
2. This script starts both programs

```
$ sudo ./start.sh
```
3. This script restarts both programs
```
$ sudo ./restart.sh
```

# Rebuilding The Droplet
This stuff is very advanced and must only be done if absolutely nessecary because it wipes the current setup and it is up to you to rebuild it from scratch.

[Digital Ocean's Guide](https://docs.digitalocean.com/products/droplets/how-to/rebuild/)

Make a snapshot of the droplet in case something goes wrong and you have to revert it. Rebuild the droplet to wipe all files and programs from it. The droplet is currently running Ubuntu 20.04, so rebuild it from a base image of Ubuntu 20.04.

## Re-establishing ssh connection
After you rebuild the droplet, you won't be able to access the console from digital ocean until you re-establish a ssh connection to it on the ops computer.

1. Find the IP address of stream.wuog.org in digital ocean. It's going to be a number that looks something like this -> ###.###.###.###
2. Open windows powershell on the ops computer. Replace the parentheses with the IP you found and run the following command.
```
> ssh root@(IP of stream.wuog.org)
```

This establishes your connection to the console of our livestream droplet.

## Creating user
[Digital Ocean's Guide](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-ubuntu-20-04)

Rebuilding the droplet deletes all users, so you’ll need to make an icecaster user again before you can build the stream.

1. Enter the droplet console as the root user from digital ocean.
2. `$ adduser icecaster`
   
   After this command, follow the instructions to set the user’s passwords, they current one can be found in the transition doc. Hit enter to continue
4. `$ usermod -aG sudo icecaster`

   This gives icecaster sudo privileges

After these steps, leave the console and re-enter it from digital ocean as icecaster.

## Installing Emacs
Emacs is a command line text editor that we will use to edit files. When a file is open in emacs, you can use the arrow keys to navigate and must right click to paste text. To save a file, use ctrl+x then ctrl+s, and to close a file, use ctrl+x then ctrl+c. If you mess up either of these shortcuts, use ctrl+g to clear the shortcut input.

1. `$ sudo apt install -y emacs`

Emacs is very old and some people prefer to use nano, if thats you then then use nano instread of emacs for the rest of this tutorial. The commands to save a file open in nano is ctrl+o, to close a file is ctrl+x. Pasting and file navigation are the same as emacs.

## Installing Nginx
[Guide](https://docs.vultr.com/how-to-install-nginx-web-server-on-ubuntu-24-04)

Nginx is the reverse proxy that uses our ssl certificate to make our livestream secure. Install this before installing icecast. The instructions below are for installing Nginx on Ubuntu 24.04, but it should still work on 20.04.

Enter the console to the livestream droplet through digital ocean (be sure to log in as icecaster) and enter the following commands:

1. `$ sudo apt update`
2. `sudo apt install nginx -y`
3. `sudo nginx -v`

   If successful, this command should output some text with the version number.

4. `$ sudo systemctl enable nginx`

## Installing Icecast2
[Guide](https://docs.vultr.com/install-icecast-on-ubuntu-20-04)

1. `$ sudo apt update`
2. `$ sudo apt upgrade`
3. `$ sudo apt install icecast2`

   This command will require you to confirm and set passwords. Keep localhost as the server hostname.

4. `$ sudo systemctl enable icecast2`

## Editing Config File And Setting Up SSL

1. `$ sudo emacs /etc/icecast2/icecast.xml`

   This opens icecast’s configuration file

2. Scroll down to the listen-socket section of the config file and create another <listen-socket> entry. You can copy the two from below, but the 8000 socket entry should already be in there. 8000 is the default port, keep this entry if you also want to stream without ssl. The 8000 socket will stream to stream.wuog.org:8000. The 443 socket will stream to stream.wuog.org. icecast.xml should now contain the two socket entries listed below.
   
```
<listen-socket>
    <port>8000</port>
    <!-- <bind-address>127.0.0.1</bind-address> -->
    <!-- <shoutcast-mount>/stream</shoutcast-mount> -->
</listen-socket>
<listen-socket>
    <port>443</port>
    <ssl>1</ssl>
</listen-socket>
```

3. `$ sudo systemctl restart icecast2`

   Restart Icecast

4. `$ sudo emacs /etc/nginx/conf.d/icecast.conf`

   Create a configuration file for nginx

5. Paste this into the configuration file and save the file. Some of the entries in this file may be changed or added to automatically by certbot later.

```
server {
    listen 80;
    listen [::]:80;
    server_name stream.wuog.org;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;

    location / {
        proxy_set_header Accept-Encoding "";
        proxy_pass http://127.0.0.1:8000/;
        sub_filter_types application/xspf+xml audio/x-mpegurl audio/x-vclt text/css text/xml;
        sub_filter ':8000/' '/';
        sub_filter '@localhost' '@stream.wuog.org';
        sub_filter 'localhost' $host;
        sub_filter 'Mount Point ' $host;
    }
}

```

6. `$ sudo nginx -t`

   Makes sure that the config file is set correctly

7. `$ sudo systemctl restart nginx`

   Restart nginx.

8. `$ sudo ufw allow 'nginx full'`
9. `$ sudo ufw allow 8000/tcp`
10. `$ sudo ufw reload`
11. `$ sudo apt install certbot python3-certbot-nginx`
12. `$ sudo certbot -d stream.wuog.org --agree-tos`
13. `$ sudo systemctl restart nginx`

## Creating Start/Restart Scripts
To make restarting the stream easier, you can create a script in icecaster's home directory.

1. `$ pwd`

   This should return “/home/icecaster”. If it does not, then leave the console and log back in as icecaster.

2. `$ sudo emacs start.sh`
3. Into this file, write and save:

```
echo sudo systemctl start icecast2
sudo systemctl start icecast2
echo sudo systemctl start nginx
sudo systemctl start nginx
```

4. `$ sudo emacs restart.sh`
5. Into this file, write and save:

```
echo sudo systemctl restart icecast2
sudo systemctl restart icecast2
echo sudo systemctl restart nginx
sudo systemctl restart nginx
```

6. `$ sudo chmod -rwx start.sh`
7. `$ sudo chmod -rwx restart.sh`


