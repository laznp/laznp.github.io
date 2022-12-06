---
permalink: /install-bspwm-archlinux
---

# **Install BSPWM on Arch Linux**

## Update Arch Linux

Before you begin, it is important to update your system before you install bspwm. To do this, run the following command in your terminal:

```sh
sudo pacman -Syu
```

## Install Dependencies

In order to install bspwm, you will need to install a few dependencies. Run the following command in your terminal to install them:

```sh
sudo pacman -S xorg xorg-xinit xorg-xbacklight
```

## Install BSPWM and Polybar

Now that you have all the necessary dependencies, you can now install bspwm. Also Bspwm does not come with a built-in status bar. Therefore, you will need to install a status bar that is compatible with Bspwm. A popular choice is [Polybar](https://github.com/jaagr/polybar). To do this, run the following command in your terminal:

```sh
sudo pacman -S bspwm
sudo pacman -S polybar
```

Once the installation is complete, you will need to create a configuration file for Polybar. To do this, run the following command:

```sh
polybar example
```

This will create a configuration file named `example.config` in your current directory. You can modify this file to customize your Polybar installation.


## Configure BSPWM

Now that bspwm is installed, you will need to configure it. To do this, you need to create a configuration file for bspwm. Create a file called `.bspwmrc` in your home directory with the following content:

```sh
# bspwmrc

# set the border size
bspc config border_width 2

# set the window gap size
bspc config window_gap 12

# set the window gap size
bspc config split_ratio 0.52

# set the window gap size
bspc config borderless_monocle true

# set the window gap size
bspc config gapless_monocle true
```

## Start BSPWM and Polybar

Now that you have everything configured, you can now start bspwm. To do this, you will need to create an xinitrc file. Create a file called `.xinitrc` in your home directory with the following content:

```sh
# xinitrc

# start bspwm
exec bspwm
exec polybar
```

## Log Out and Log In

Now that you have everything set up, you can now log out of your current session and log in using bspwm. 

Congratulations, you have successfully installed bspwm on your Arch Linux system!