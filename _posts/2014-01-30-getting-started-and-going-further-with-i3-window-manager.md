---
layout: post
title: Getting started and going further with i3 window manager
category: tech
---
If you're new to i3 window manager (i3wm) you probably aren't aware of the customization capabilities it has. I present here a few configuration customizations that I've accumulated on my own machine that I encourage you to make use of for your own.

###Adding custom i3 key bindings
After you install i3 you should have a configuration file located at `~/.i3/config`. This config file contains all the key bindings for i3. The default configuration is pretty good but you may desire to add your own key bindings for custom actions. If you look through the file, you can see that it's filled with `bindsym` and `bindcode` lines followed by key combinations and commands. It's fairly easy to add your own. 

The simplest way to add a new binding is using `xev` to find the appropriate key code and then using `bindcode` in your configuration file. I wanted to bind my laptops native Volume+ and Volume- icons on my keyboard. Adding the custom binding was easy:

1. Opened up the config file (`~/.i3/config`) with my favorite text editor (Sublime Text in my case)
2. Switched to another workspace and launched a terminal (Modifier + Enter)
3. Ran the command `xev` (to open up the 'Event Tester' application)
4. Focused the 'Event Tester' window by mousing over it
5. Pressed the key I wanted to bind
6. Noted the keycodes that were printed in the terminal window (68 and 69 came out in the following format: `state 0x0, keycode 69 (keysym 0xffc0, F3), same_screen YES,`)
7. Added a line in my configuration file structured like the following: `bindcode $mod+68 exec amixer set Master 5%-`
8. Saved the file
9. Restarted i3 in place (Modifier + Shift + R)

In my case, `amixer` was a good way to control the volume on my system but you might have a different command you wish to use. I also use `xbacklight` to bind "brightness" keys on my keyboard to control the backlight. Here are some examples of custom key bindings I set up:

{% highlight sh %}
# volume control (Alt + F2 or Alt + F3)
bindcode $mod+68 exec amixer set Master 5%-
bindcode $mod+69 exec amixer set Master 5%+

# brightness control (Alt + F11 or Alt + F12)
bindcode $mod+95 exec xbacklight - 10
bindcode $mod+96 exec xbacklight + 10

# lock computer (Alt + Delete)
bindcode $mod+119 exec i3lock -c 000000

# suspend computer
bindsym $mod+Shift+s exec dbus-send --system --print-reply --dest="org.freedesktop.UPower" /org/freedesktop/UPower org.freedesktop.UPower.Suspend
{% endhighlight %}

###List of common commands for i3 actions
If we use i3wm, we need to find commands to perform common actions. Below are some common terminal commands and applications I use to do things on my laptop. These can be used with `exec` in your config file to bind keys to functions.

<table class="table table-striped">
    <tr>
        <th>Function</th>
        <th>Command</th>
    </tr>
    <tr>
        <td>Run/Start i3 from command line</td>
        <td><code>startx i3</code></td>
    </tr>
    <tr>
        <td>i3 logout command</td>
        <td><code>i3-msg exit</code></td>
    </tr>
    <tr>
        <td>i3 restart command (in place)</td>
        <td><code>i3-msg restart</code></td>
    </tr>
    <tr>
        <td>i3 volume control</td>
        <td><code>amixer set Master &lt;amount&gt;</code><br><small>e.g. &lt;amount&gt; = 5+</small></td>
    </tr>
    <tr>
        <td>i3 volume mute toggle</td>
        <td><code>amixer -D pulse set Master 1+ toggle</code></td>
    </tr>
    <tr>
        <td>i3 brightness control</td>
        <td><code>xbacklight &lt;+/-&gt; 10</code><br><small>may need to install <code>xbacklight</code></small></td>
    </tr>
    <tr>
        <td>i3 lock computer</td>
        <td><code>i3lock --image=&lt;bg image path&gt;</code></td>
    </tr>
    <tr>
        <td>i3 set wallpaper</td>
        <td><code>feh --bg-scale &lt;path-to-image&gt;</code><br><small>may need to install <code>feh</code></small></td>
    </tr>
</table>

What applications do you use alongside i3? [I'd like to know.](mailto:brent@walther.io "Email me.")
