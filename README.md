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

[Digital Ocean's guide](https://docs.digitalocean.com/products/droplets/how-to/rebuild/)

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
[Digital Ocean's Guild](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-ubuntu-20-04)

