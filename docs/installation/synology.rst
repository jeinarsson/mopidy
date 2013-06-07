.. _raspberrypi-installation:

****************************
Installation on Synology NAS
****************************


What, why?
==========

A NAS server is (usually) an embedded Linux computer, with lots of hard drives.
As such, it can also run mopidy, provided that mopidy is compiled for the 
relevant architecture.

If the NAS has USB ports, it is possible that you can use an USB DAC (Sound card)
to output sound directly from your NAS to an amplifier.

If you don't know why you would like to do this, probably you don't want to
go through the trouble detailed below.

This document provides at least basic information on how to achieve a working
installation of mopidy on a Synology NAS.

This guide is tested on
=======================
* Synology DS413j (ARMv5, mv6282)

To see what CPU your NAS has, visit
http://forum.synology.com/wiki/index.php/What_kind_of_CPU_does_my_NAS_have

How to
============================

#. Enable the command line interface in the DSM Control Panel

    This enables you to SSH into the NAS, login as root / theadminpassword
    Caution: As root on your NAS you can do serious harm. Do not attempt any
    experimental procedures (such as the one described here) on a system where
    you cannot afford a crash.

#. Install Python in the DSM Package Manager

#. Install the IPKG package manager

    * Easiest way

      * Add http://packages.quadrat4.de/ to your package repositories in the DSM Package Manager,
      * Install the newly found IPKG Bootstrap in the DSM Package Manager

    * Other way:
    
      * Please see
        http://forum.synology.com/wiki/index.php/Overview_on_modifying_the_Synology_Server,_bootstrap,_ipkg_etc#How_to_install_ipkg

#. Get toolchain with ipkg

  * Get the following packages with ipkg
  
    .. code-block:: bash

      ipkg install opt-devel gcc git alsa-lib alsa-utils flac
      ipkg install wavpack ffmpeg 

    The ALSA libs in ipkg are not the latest, however, it seems to
    work and it's one less thing to compile..

  * Make and go to a temporary directory, I use /opt/tmp
  
    .. code-block:: bash

      mkdir /opt/tmp
      cd /opt/tmp

#. Compile and install GStreamer
  
    First we need the prerequisites. The glib and lots of other stuff
    in ipkg is too old, so we have to download and compile most things.
    First need its prerequisites, libffi, gettext and pkg-config.
    
    .. code-block:: bash

      wget http://ftp.gnome.org/pub/gnome/sources/glib/2.34/glib-2.34.3.tar.xz
      wget http://pkgconfig.freedesktop.org/releases/pkg-config-0.28.tar.gz
      wget ftp://sourceware.org/pub/libffi/libffi-3.0.13.tar.gz
      wget http://ftp.gnu.org/pub/gnu/gettext/gettext-0.18.2.tar.gz
      tar -xvf glib-2.34.3.tar.xz
      tar -xvf pkg-config-0.28.tar.gz
      tar -xvf libffi-3.0.13.tar.gz
      tar -xvf gettext-0.18.2.tar.gz

    Now, before compiling, we need to work around what seems to be a bug
    in the pthreads installation: http://forum.synology.com/enu/viewtopic.php?f=90&t=30132

    .. code-block:: ssh
    
      mkdir /opt/arm-none-linux-gnueabi/lib_disabled
      mv /opt/arm-none-linux-gnueabi/lib/libpthread* /opt/arm-none-linux-gnueabi/lib_disabled
      cp /lib/libpthread.so.0 /opt/arm-none-linux-gnueabi/lib/
      cd /opt/arm-none-linux-gnueabi/lib/
      ln -s libpthread.so.0 libpthread.so
      ln -s libpthread.so.0 libpthread-2.5.so

    And there are two tools, perl and m4, which some packages will look for in the wrong
    location. We work around this by simply making a copy of the binaries.

    .. code-block:: bash

      cp /opt/bin/perl* /usr/bin
      cp /opt/bin/m4 /usr/bin

    When making, use prefix=/opt to install in the correct folder, like so:

    .. code-block:: bash

      cd pkg-config-0.28
      ./configure --prefix=/opt
      make
      make install
      cd ..

      cd libffi-3.0.13
      ./configure --prefix=/opt
      make
      make install
      cd ..

      cd gettext-0.18.2
      ./configure --prefix=/opt
      make
      make install
      cd ..    

      cd glib-2.34.3
      ./configure --prefix=/opt --disable-gtk-doc --disable-man --disable-gcov
      make
      make install
      cd ..

    Finally we can build the actual gstreamer package. We are going to compile
    gstreamer, and the base + good + ugly plugins. URLs taken from here:
    http://gstreamer.freedesktop.org/modules/
    It seems there are newer versions, but I happened to take 1.0.6.

    .. code-block: bash

      wget http://gstreamer.freedesktop.org/src/gstreamer/gstreamer-1.0.6.tar.xz
      wget http://gstreamer.freedesktop.org/src/gst-plugins-base/gst-plugins-base-1.0.6.tar.xz
      wget http://gstreamer.freedesktop.org/src/gst-plugins-ugly/gst-plugins-ugly-1.0.6.tar.xz
      wget http://gstreamer.freedesktop.org/src/gst-plugins-good/gst-plugins-good-1.0.6.tar.xz
      tar -xvf gstreamer-1.0.6.tar.xz
      tar -xvf gst-plugins-base-1.0.6.tar.xz
      tar -xvf gst-plugins-ugly-1.0.6.tar.xz
      tar -xvf gst-plugins-good-1.0.6.tar.xz    

    And start the compilation/installation fest

    .. code-block: bash

        cd gstreamer-1.0.6
        ./configure --prefix=/opt
        make
        make install
        cd ..

        cd gst-plugins-base
        ./configure --prefix=/opt
        make
        make install
        cd ..

        cd gst-plugins-ugly
        ./configure --prefix=/opt
        make
        make install
        cd ..

        cd gst-plugins-good
        ./configure --prefix=/opt
        make
        make install
        cd ..


#. Profit
