---
title: Install PyQt5 Inside a Virtual Environment
author: Cody
published: 2017-08-17T00:00:00+00:00
_hash: 36a472517171a3c311a61be5632759159f7dba5b3e165aaeac6b08a4c0f45e5f
updated: 2026-03-20T20:19:42.953403+00:00
...

A <a href="https://docs.python.org/3/library/venv.html" target="_blank">virtual environment</a>
allows you to install specific versions of Python distribution packages in a contained environment without contaminating the system Python.
This allows you to have multiple versions of PyQt5 installed on the same system
so you can work on different projects that use different versions of PyQt5.

Download the open source online installer from https://www.qt.io/ or use the maintenance tool if you already have Qt installed.

```shell
wget http://mirror.csclub.uwaterloo.ca/qtproject/archive/online_installers/3.0/qt-unified-linux-x64-3.0.0-online.run
chmod u+x qt-unified-linux-x64-3.0.0-online.run
./qt-unified-linux-x64-3.0.0-online.run
```

Navigate the installation wizard. Skip the signup and install version Qt 5.9.1.

<h2 id="download-sip">Download SIP source code</h2>

https://www.riverbankcomputing.com/software/sip/download

<h2 id="download-pyqt5">Download PyQt5 source code</h2>

https://www.riverbankcomputing.com/software/pyqt/download5

<h2 id="venv">Create a virtualenv for PyQt5</h2>

```shell
sudo apt-get install python3-venv
python3 -m venv ~/.virtualenvs/pyqt5
source ~/.virtualenvs/pyqt5/bin/activate
# It is better to source the venv instead of just using the full path to the venv python
# because it will use that python for the paths in qmldir for the pyqt5qmlplugin example
# Yes the examples in the download actually get changed when you build pyqt5.
```

<h2 id="install-sip">Install SIP</h2>

```shell
cd ~/Downloads/sip-4.19.4.dev1708131720/
python configure.py
make
make install
```

Now the `sip` distribution package is installed in that virtualenv.

```shell
~/.virtualenvs/pyqt5/bin/python
```

```
Python 3.5.2 (default, Sep 14 2017, 22:51:06)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sip
>>> # Works!
```

<h2 id="install-pyqt5">Install PyQt5</h2>

In order to build `QML` support I had to install `libgl1-mesa-dev` on Ubuntu 16.04.
See <a href="#bonus-2">Bonus #2</a> for how I figured out which package to install.

```shell
sudo apt-get install libgl1-mesa-dev
```

```shell
cd ~/Downloads/PyQt5_gpl-5.9.1.dev1707250927/
python configure.py --qmake ~/Qt/5.9.1/gcc_64/bin/qmake --sip ~/.virtualenvs/pyqt5/bin/sip
make
make install
```

<h3 id="bonus-1">Bonus #1 Add to pip freeze</h3>

If you pip freeze you won't see `sip` and `PyQt5` because they don't have a `.dist-info` directory.

```shell
~/.virtualenvs/pyqt5/bin/pip freeze
```

```
pkg-resources==0.0.0
```

You can however manually create them.

```shell
mkdir ~/.virtualenvs/pyqt5/lib/python3.5/site-packages/sip-4.19.4.dev1708131720.dist-info
touch ~/.virtualenvs/pyqt5/lib/python3.5/site-packages/sip-4.19.4.dev1708131720.dist-info/INSTALLER
mkdir ~/.virtualenvs/pyqt5/lib/python3.5/site-packages/PyQt5-5.9.1.dev1707250927.dist-info
touch ~/.virtualenvs/pyqt5/lib/python3.5/site-packages/PyQt5-5.9.1.dev1707250927.dist-info/INSTALLER
```

```shell
~/.virtualenvs/pyqt5/bin/pip freeze
```

```
pkg-resources==0.0.0
PyQt5==5.9.1.dev1707250927
sip==4.19.4.dev1708131720
```

<h3 id="bonus-2">Bonus #2 How I figured out I needed that Ubuntu package</h3>

When I first ran `configure.py` for PyQt5 it didn't say it would build QtQml.

```
Checking to see if the QtQml module should be built...
# ...
These PyQt5 modules will be built: QtCore, QtNetwork, QtXml, QtXmlPatterns,
QtDBus, QtWebSockets, QtWebChannel, QtNfc.
```

So I ran it again with `--verbose` and saw there was an error for `QtQml`.

```
/usr/bin/ld: cannot find -lGL
collect2: error: ld returned 1 exit status
cfgtest_QtQml.mk:242: recipe for target 'cfgtest_QtQml' failed
```

`-lGL` means that `ld` couldn't find `libGL.so`, that's how `ld` looks for libraries.

Then I searched the Ubuntu packages for the file `libGL.so`.

```shell
dpkg -S libGL.so
```

```
libgl1-mesa-glx:i386: /usr/lib/i386-linux-gnu/mesa/libGL.so.1
libgl1-mesa-dev:amd64: /usr/lib/x86_64-linux-gnu/mesa/libGL.so
libgl1-mesa-glx:amd64: /usr/lib/x86_64-linux-gnu/mesa/libGL.so.1
libgl1-mesa-glx:amd64: /usr/lib/x86_64-linux-gnu/mesa/libGL.so.1.2.0
libgl1-mesa-glx:i386: /usr/lib/i386-linux-gnu/mesa/libGL.so.1.2.0
libgl1-mesa-dev:amd64: /usr/lib/x86_64-linux-gnu/libGL.so
```
