# Harry's quest to leanify the raspberry pi (model 3b)

### Fact: Processes, Packages, Services, and more all run/exist within the base ubuntu-server image.
### Hypothesis: If I trim said fats, I should be able to observe a measurable (*top) and noticeable (*app) decrease in resource usage and increase in performance headroom.
### tldr: Can I optimize the rpi? Let's find out...

Context: Feb-02-2020 - ubuntu-19.10.1-preinstalled-server-armhf+raspi3.img

## Step 1. Annoyances

#### Waiting for cloud-init to do its thing and waiting for apt upgrade on rpi is equivalent to skeleton_help_desk.gif (https://imgur.com/DNsXXq9)

Disable apt update/upgrade on boot:

    sudo su
    
    system-ctl stop apt-daily.service
    system-ctl disable apt-daily.service
    system-ctl stop apt-daily-upgrade.service
    system-ctl disable apt-daily-upgrade.service

    system-ctl stop apt-daily.timer
    system-ctl disable apt-daily.timer
    system-ctl stop apt-daily-upgrade.timer
    system-ctl disable apt-daily-upgrade.timer

Remove cloud-init:

    apt purge cloud-init

    rm -rf /etc/cloud/
    rm -rf /var/lib/cloud/

## Step 2. Feel the Situation

Run htop for a moment and get a feel for the current usage off of a freshly booted image:

(Note down any big hitters for resource consumption if found)

    htop
    # At this point, I notice 131M RAM and multiple processes running and consuming CPU.
    # Sorting by memory usage, I notice 13 processes exist for snapd.
    # I also see dbus, wpa_supplicant, and many systemd.

## Step 3. Start Digging

Sites I will reference for different components to disable/remove:

[1] https://wiki.linuxaudio.org/wiki/raspberrypi

[2] https://plone.lucidsolutions.co.nz/hardware/raspberry-pi/3/disable-unwanted-raspbian-services

[3] https://askubuntu.com/a/1114686

Let's begin with the low hanging fruits.

Since I do not need snapd, let's get rid of it and see how it affects things.

(Note: I keep htop open within tmux through ssh, leading to a few megabytes of overhead in measurements.)

    # ref:[3]

    sudo rm -rf /var/cache/snapd/

    sudo apt autoremove --purge snapd

    rm -fr ~/snap

After performing this action, RAM usage drops to 127M as snap processes are killed in leu of removal.

Cool! 4megabytes is 4megabytes.

Since I do not plan on running wifi or bluetooth for my pi, let's disable each and see what happens.

    # ref:[2]

    vim /etc/modprobe.d/raspi-blacklist.conf
    # pasting in blacklists for wifi and bluetooth from page

    # the disabling of all mentioned services in the page did not exist,
    # so I proceeded to reboot the host to see if blacklisting helps resource usage.

    shutdown -r now

...

After rebooting and letting the dust settle, I opened up htop to find the RAM usage had dropped once more to a whopping 111M! Woohoo, that's another 16megabytes of RAM restored! (give or take 3M due to tmux)

Now, this may not seem much in the grand scheme of things, however it is important to understand the context of our situation; the RAM we restore to being free is a good 10% of the existing usage, as well as the fact that we do not have much to work with in the first place (853M).

I am also interested in seeing what happens when we reduce GPU usage in raspi-config since we are only interested in running through tty/cli.

Let's get back into the groove of things with ref:[1].

Going through the list of services to disable, I will list the ones that worked for me here and the 'M' dropped and resulting total.

    service dbus stop
    # -2M (111M)
    killall dbus-daemon
    # -9M (102M)

Running these commands also resulted in less processe being shown in htop and pstree,
therefore CPU should be experiencing a lighter load across the board.

The rest did not seem to do anything, so I went ahead and moved onto the next big hitter.

As mentioned before, I plan on running this for cli only purposes; no GPU, no DE.

Therefore, I find it fitting to utilize config.txt for reducing GPU memory share.

    vim /boot/firmware/config.txt

    # head to the bottom of the file and insert
    gpu_mem=16

According to documentation, this is the lowest allowed value.
Let's see what happens after a reboot.

...

Lo' and behold!
The memory usage does not drop!
In fact, it increased back to 111M!

Well, let's not panic just yet.
Remember the last two commands we just ran?

Those two do not survive reboots, so it is imperative that we have those commands be sticked onto a boot command through systemd, initd, or etc. (your preference).

[4] http://raspberrywebserver.com/serveradmin/run-a-script-on-start-up.html

Running those two again bring us back down to our holy and peaceful 102M.

I will deal with automating those commands on a later date.

What I did not mention, and saved for last, is what setting the gpu_mem actually it.
It did do something, believe it or not, and what it did was raise the headroom of our RAM from 853M all the way up to 902M!

Woohoo, a whole 'nother whopping 9megabytes of RAM to add to the stack!

But the journey isn't over yet!
We have all this RAM available to us now, and all we have to do is find a way to use it.
You could use ramfs to store data into the newfound RAM space and avoid SD r/w.
You could also run webservers or minecraft servers a little more happily with the extra room.
And with less processes, there are less interrupts to bother your important application runtimes.

## Step 4. 'Kowalski, Analysis.'

To answer the original hypothesis, after making much observation with the many actions taken herein,
I can safely declare,

### It is true that removing packages, services, and processes did give a more measurable headroom in CPU and RAM, and it is a safe claim to say that unused resources benefits the runtime of other programs who are more resource hungry in a noticeable way (I am looking at you Java).

This is a 'ymmv' for every user's use case.
I simply came here today to see how much I could really squeeze out of a Raspberry.

Take what you will from this, and I hope it can help you in your usages in the long run!

Until next-time, Peace Out!
- Harrison