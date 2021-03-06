# statnot
statnot is a [notification-daemon](http://www.galago-project.org/news/index.php) replacement for lightweight window managers like [dwm](http://dwm.suckless.org) and [wmii](http://wmii.suckless.org). It receives and displays notifications from the widely used [Desktop Notifications](http://www.galago-project.org/specs/notification/0.9/index.html) speficiation.

* [latest release](http://www.k2h.se/code/dl/statnot-latest.tar.gz)
* [source repository](http://github.com/halhen/statnot/tree/master)

Distribution specific links:

* [archlinux AUR package](http://aur.archlinux.org/packages.php?ID=25528)

## Background
In some lightweight window managers, the text in the status bar is fed from an external process. For example dwm (version 5.4 and above) reads the status message from the X root window name, set by `xsetroot -name <text>`. A user typically enters a loop like below in .xinitrc to keep the status bar updating.

    while true
    do
        xsetroot -name "$(date +"%F %R")"
        sleep 30s # update every thirty seconds
    done &

This solution works well for status messages that are managed from a single point, for example when printing the same information every time. statnot lets you combine regular status messages with Desktop Notifications in a straightforward way. 

### Desktop Notifications
If you have used a "regular" window manager like KDE or Gnome, you have probably come across notifications. The are typically small windows with text messages and sometimes an icon that shows for a couple of seconds before they fade out. They are for example used to let the user know that a new instant message has arrived or that the battery is running low. Desktop Notifications is a specification created for freedesktop.org that many applications use. For example Pidgin and Evolution can be configured to notify for new messages using Desktop Notifications.

## Installation
*Note: if you install statnot through a package manager, some of these steps have been taken care for you. You probably need to edit the .xinitrc yourself, see below.*

To install statnot, first install the required dependencies:

* [python 2.5+](http://www.python.org)
* [dbus-python](http://dbus.freedesktop.org/releases/dbus-python/)
* [pygtk](http://www.pygtk.org/) - (not for GUI support, but the dbus-python library requires it)

Next, adjust the target directories in the `config.mk` file to fit your setup. 

To install, run as root:

    # make install

Finally, statnot needs to start with the window manager. You can for example add the following to .xinitrc:

    killall notification-daemon &> /dev/null
    statnot & 

Note that the statnot needs to be the only notification tool running. The example above makes sure that `notification-daemon` is not running.

## Configuration
The major, likely only, part you want to configure in statnot is what the status message should look like. During installation, a file called `.statusline.sh` is created in `$HOME/`. This gets called with regular intervals to retrieve the text that should be printed on the status line. statnot reads the first line of STDOUT, so a simple `echo <text>` is a good way to return the text.

During normal status updates, .statusline.sh is called without parameters. Here, you typically fetch and `echo` information about the computers performance, battery level or current time. 

Any pending notification is passed as the first argument to .statusline.sh. This way you can include additional information to the actual notification. 

Below is an example of .statusline.sh. It prints something like `[load 0.12 0.10 0.7]   11:42` in the status bar. When there is a pending notification, it prints `NOTIFICATION: <notification text>`.

    if [ $# -eq 0 ]; then
        loadavg="`cat /proc/loadavg | awk '{print $1, $2, $3}'`";
        echo "[load ${loadavg}]   `date +'%R'`";
    else
        echo "NOTIFICATION: $1";
    fi

### Advanced configuration
*Note: statnot is in an early version. Configuration is crudely performed in the Python program itself. To adjust these variables, use `which statnot` to figure out where statnot is located and open it in an editor as root. The configuration section starts some twenty lines from the top. This will change in future, more mature, versions.*

* `DEFAULT_NOTIFY_TIMEOUT`: number of milliseconds a notify should show unless it requests otherwise. Default: 3000.
* `MAX_NOTIFY_TIMEOUT`: maximum number of milliseconds a notification is allowed to show. Default: 5000.
* `NOTIFICATION_MAX_LENGTH`: maximum length in number of charaters that are shown from a notification. The message is truncated if it is longer. Default: 100.
* `STATUS_UPDATE_INTERVAL`: how often the status message is updated in seconds. Default: 2.0.
* `STATUS_COMMAND`: Speficies which command to fetch the status message from. Default: `/bin/sh $HOME/.statusline.sh`.

statnot calls the `update_text(text)` function to update the actual status message. The default implementation updates the status bar in the dwm way, using `xsetroot -name`. You can write your own `update_text` function if this doesn't suite your needs.

## Possible errors
If no status message shows, verify that statnot is running. Also make sure your $HOME/.statusline.sh works and prints properly.

If notifications are not shown, make sure that no other notification-daemon is running. `killall notification-daemon` is a good command to try. Restart statnot if there was another daemon running. Also make sure that .statusline.sh takes care of and prints the $1 parameter (see section Configuration).

## Supported software
More and more applications use Desktop Notifications. Use [Google](http://www.google.com) to find solutions for your applications. `libnotify` is a good term to search, since it is a common library used by many applications.

You can also send your own notifications to statnot. This is easily done with the `notify-send` command. For example, `notify-send "Hello World"` will print `Hello World` in the status bar according to your speficiation. This is useful to notify that a long running task like a download or software build has finished.

notify-send can also be used for other, more direct messages. For exampe, I call a script called `dwm-volume` when my volume media buttons on the keyboard are pressed. This script adjusts the volume and sends a notification containing e.g. `vol [52%] [on]`. 

    #!/bin/sh
    if [ $# -eq 1 ]; then
        amixer -q set Master $1
    fi
    notify-send -t 0 "`amixer get Master | awk 'NR==5 {print "vol " $4, $6}'`"

As you can see, I use the option `-t 0` to notify-send, i.e. I request that the notification should show for zero milliseconds. For statnot, this means that the message should show for a regular status tick, by default two seconds, but if other notifications arrive, like a second press on the volume button, it goes away. This setup allows my audio volume to show only when I change it, while it updates instantly when I press the media buttons.

## Final notes
I'm sure there are other ways to use statnot. For example, one can create an update_text() that sends notifications as e-mail or instant messages, or that stores them to a log file. If you create any cool applications with statnot, I'd be happy to hear about them.

If you are interested in more examples, my [dotfiles, including .statusline.sh](http://github.com/halhen/dotfiles/tree/master) and [dwm configuration](http://github.com/halhen/dwm/tree/master) are available on [github](http://github.com/halhen).

Written by Henrik Hallberg (<halhen@k2h.se>) and released under the GPL. Please report any bugs or feature requests by email. Also, please drop me a line to let me know you like and use this software.

<http://code.k2h.se>


