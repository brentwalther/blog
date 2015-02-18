---
layout: post
title: How to replace Unity with i3 window manager on Ubuntu 12.04
category: tech
---
So you've decided that you don't like Unity and you'd like to replace it. [i3 window manager](http://i3wm.org/ "http://i3wm.org/") is a popular tiling window manager that is easy to learn and can greatly improve productivity. Once you've gotten the hang of all the hotkeys, you'll find that you rarely need a mouse or touchpad anymore. I personally became much more productive when I installed it, but I had some trouble finding good information on how to completely replace Unity a still retain the  default Ubuntu login manager called lightdm.
###Installing i3
In order to install the stable version of i3 on Ubuntu, open up your terminal using `Ctrl + Alt + T` and run the following commands under root (preferably using `sudo`):

{% highlight sh %}
# login to the root shell for one 'echo' command
sudo -i
echo "deb http://debian.sur5r.net/i3/ $(lsb_release -c -s) universe" >> /etc/apt/sources.list
logout
# we've returned to our user account
sudo apt-get update
sudo apt-get --allow-unauthenticated install sur5r-keyring
sudo apt-get update
sudo apt-get install i3
{% endhighlight %}

At this point, you'll be able to use i3 and make sure that it works well on your system. To use i3:

* Logout of Unity
* From the login screen, click the circle icon above your user account
* Select "i3"
* Login with your password
* i3 will offer to create a config file. Press enter to create it
* i3 will ask you to choose a modifier key. Choose the key you'd like by pressing it on the keyboard and then press enter

You'll find yourself at a black screen. Welcome to i3! Assuming you're not familiar with the hotkeys, you can use `Modifier + D` to open the application launcher or `Modifier + Enter` to open a terminal. To logout of i3, open a terminal and run the command `i3-msg exit`

###Removing Unity
If you're satisfied with i3, and you'd like to use it along with the login screen `lightdm`, we've got the option of removing Unity and Compiz. To do that, run the following commands:

{% highlight sh %}
sudo apt-get autoremove --purge compiz compiz-gnome compiz-plugins-default libcompizconfig0

sudo apt-get autoremove --purge unity unity-common unity-services unity-lens-* unity-scope-* libunity-core-6* libunity-misc4 appmenu-gtk appmenu-gtk3 appmenu-qt* overlay-scrollbar* activity-log-manager-control-center firefox-globalmenu thunderbird-globalmenu
# the following command will disable the desktop (we won't need it with i3!)
gsettings set org.gnome.desktop.background show-desktop-icons false
{% endhighlight %}

###Configuring `lightdm`
Since we removed Unity, we need to reconfigure the login manager `lightdm` to login with i3 by default. This can be accomplished with a few simple steps:

1. Run the command `sudo nano ~/.xinitrc` 
2. Type into the file `exec i3`
3. Save the file using `Ctrl + O`
4. Exit the file using `Ctrl + X`
5. Run the command `sudo nano /etc/lightdm/lightdm.conf`
6. Change the line with `user-session` to read `user-session=i3`
7. Save the file using `Ctrl + O`
8. Exit the file using `Ctrl + X`
9. Restart your computer with the command `sudo init 6`

**You're done!** Enjoy your new window manager!

Want to keep going/learn more? Check out my [customization guide/cheat sheet](http://brentwalther.net/blog/getting-started-and-going-further-with-i3-window-manager "Customization Guide / Cheat Sheet")
