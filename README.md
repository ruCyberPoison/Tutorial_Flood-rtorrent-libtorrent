# Tutorial_Flood-rtorrent-libtorrent
A tutorial to install flood + rtorrent+libtorrent

Intro: Blue = text to input in command line
#### At firts update and upgrade your Ubuntu server.

```Markdown
´root@seedbox:~#` apt-get update && apt-get upgrade
```

#### Install required packages.

```Markdown
´root@seedbox:~#` apt install git curl build-essential automake pkg-config libtool libcppunit-dev zlib1g-dev libncursesw5-dev libncurses5-dev libssl-dev libcurl4-openssl-dev subversion libsig++ aptitude screen
```

#### Create a directory to put all your future files.

```Markdown
´root@seedbox:~#` cd ~/
`´root@seedbox:~#` mkdir seedbox
`´root@seedbox:~#` cd seedbox/
```

#### Download the xmlrpc-c repo and install.

```Markdown
´root@seedbox:~/seedbox#` git clone https://github.com/mirror/xmlrpc-c.git
`´root@seedbox:~/seedbox#` cd xmlrpc-c/stable
`´root@seedbox:~/seedbox/xmlrpc-c/stable#` ./configure
`´root@seedbox:~/seedbox/xmlrpc-c/stable#` make
`´root@seedbox:~/seedbox/xmlrpc-c/stable#` make install
`´root@seedbox:~/seedbox/xmlrpc-c/stable#` cd ~/seedbox
```

#### Download the libtorrent repo from rakshasa and compile and install.

```Markdown
´root@seedbox:~/seedbox#` git clone https://github.com/rakshasa/libtorrent.git
`´root@seedbox:~/seedbox#` cd libtorrent
`´root@seedbox:~/seedbox/libtorrent#` ./autogen.sh
`´root@seedbox:~/seedbox/libtorrent#` ./configure
`´root@seedbox:~/seedbox/libtorrent#` make
`´root@seedbox:~/seedbox/libtorrent#` make install
`´root@seedbox:~/seedbox/libtorrent#` cd ~/seedbox/
```

#### Download the rtorrent repo from rakshasa and compile and install.

```Markdown
´root@seedbox:~/seedbox#` git clone https://github.com/rakshasa/rtorrent.git
`´root@seedbox:~/seedbox#` cd rtorrent
`´root@seedbox:~/seedbox/rtorrent#` ./autogen.sh
`´root@seedbox:~/seedbox/rtorrent#` ./configure --with-xmlrpc-c
`´root@seedbox:~/seedbox/rtorrent#` make
`´root@seedbox:~/seedbox/rtorrent#` make install
`´root@seedbox:~/seedbox/rtorrent#` cd ~/seedbox/
`´root@seedbox:~/seedbox#` ldconfig
```

#### Configuring the rtorrent.

```Markdown
´root@seedbox:~/seedbox#` adduser --disabled-password rtorrent
`´root@seedbox:~/seedbox#` nano /home/rtorrent/.rtorrent.rc
```

##### Paste this ...

````Codeblock
#############################################################################
# A minimal rTorrent configuration that provides the basic features
# you want to have in addition to the built-in defaults.
#
# See https://github.com/rakshasa/rtorrent/wiki/CONFIG-Template
# for an up-to-date version.
#############################################################################


## Instance layout (base paths)
method.insert = cfg.basedir,  private|const|string, (cat,"/home/rtorrent/")
method.insert = cfg.download, private|const|string, (cat,"/mnt/incomplete/downloads/")
method.insert = cfg.logs,     private|const|string, (cat,(cfg.basedir),"log/")
method.insert = cfg.logfile,  private|const|string, (cat,(cfg.logs),"rtorrent-",(system.time),".log")
method.insert = cfg.session,  private|const|string, (cat,(cfg.basedir),"temp/")
method.insert = cfg.watch,    private|const|string, (cat,(cfg.basedir),"watch/")


## Create instance directories
execute.throw = sh, -c, (cat,\
    "mkdir -p \"",(cfg.download),"\" ",\
    "\"",(cfg.logs),"\" ",\
    "\"",(cfg.session),"\" ",\
    "\"",(cfg.watch),"/load\" ",\
    "\"",(cfg.watch),"/start\" ")


## Listening port for incoming peer traffic (fixed; you can also randomize it)
network.port_range.set = 40000-40000
network.port_random.set = no


## Tracker-less torrent and UDP tracker support
## (conservative settings for 'private' trackers, change for 'public')
dht.mode.set = disable
protocol.pex.set = no

trackers.use_udp.set = yes


## Peer settings
throttle.max_uploads.set = 100
throttle.max_uploads.global.set = 250

throttle.min_peers.normal.set = 20
throttle.max_peers.normal.set = 60
throttle.min_peers.seed.set = 30
throttle.max_peers.seed.set = 80
trackers.numwant.set = 80

protocol.encryption.set = allow_incoming,try_outgoing,enable_retry


## Limits for file handle resources, this is optimized for
## an `ulimit` of 1024 (a common default). You MUST leave
## a ceiling of handles reserved for rTorrent's internal needs!
network.http.max_open.set = 50
network.max_open_files.set = 600
network.max_open_sockets.set = 300


## Memory resource usage (increase if you have a large number of items loaded,
## and/or the available resources to spend)
pieces.memory.max.set = 32000M
network.xmlrpc.size_limit.set = 4M


## Basic operational settings (no need to change these)
session.path.set = (cat, (cfg.session))
directory.default.set = (cat, (cfg.download))
log.execute = (cat, (cfg.logs), "execute.log")
log.xmlrpc = (cat, (cfg.logs), "xmlrpc.log")
execute.nothrow = sh, -c, (cat, "echo >",\
    (session.path), "rtorrent.pid", " ",(system.pid))


## Other operational settings (check & adapt)
encoding.add = utf8
system.umask.set = 0000
system.cwd.set = (directory.default)
network.http.dns_cache_timeout.set = 25
schedule2 = monitor_diskspace, 15, 60, ((close_low_diskspace, 1000M))

pieces.hash.on_completion.set = yes
#view.sort_current = seeding, greater=d.ratio=
#keys.layout.set = qwerty
#network.http.capath.set = "/etc/ssl/certs"
#network.http.ssl_verify_peer.set = 0
#network.http.ssl_verify_host.set = 0


## Some additional values and commands
method.insert = system.startup_time, value|const, (system.time)
method.insert = d.data_path, simple,\
    "if=(d.is_multi_file),\
        (cat, (d.directory), /),\
        (cat, (d.directory), /, (d.name))"
method.insert = d.session_file, simple, "cat=(session.path), (d.hash), .torrent"


## Watch directories (add more as you like, but use unique schedule names)
## Add torrent
schedule2 = watch_load, 11, 10, ((load.verbose, (cat, (cfg.watch), "load/*.torrent")))
## Add & download straight away
schedule2 = watch_start, 10, 10, ((load.start_verbose, (cat, (cfg.watch), "start/*.torrent")))


## Run the rTorrent process as a daemon in the background
## (and control via XMLRPC sockets)
system.daemon.set = true
#network.scgi.open_local = (cat,(session.path),rpc.socket)
#execute.nothrow = chmod,770,(cat,(session.path),rpc.socket)


## Logging:
##   Levels = critical error warn notice info debug
##   Groups = connection_* dht_* peer_* rpc_* storage_* thread_* tracker_* torrent_*
print = (cat, "Logging to ", (cfg.logfile))
log.open_file = "log", (cfg.logfile)
log.add_output = "info", "log"
#log.add_output = "tracker_debug", "log"

###SCGI port for Flood
scgi_port = 0.0.0.0:5000

### END of rtorrent.rc ###

````

*Please edit if required !!**

#### Configuring the rtorrent. 2)

```Markdown
root@seedbox:~/seedbox# mkdir /home/rtorrent/{downloads,.session}
root@seedbox:~/seedbox# chmod 775 -R /home/rtorrent/
root@seedbox:~/seedbox# chown rtorrent:rtorrent -R /home/rtorrent
```

#### Testing the rtorrent. 3)

```Markdown
´root@seedbox:~/seedbox#` su rtorrent
`´rtorrent@seedbox:~/#` cd ~/
`´rtorrent@seedbox:~/#` wget http://releases.ubuntu.com/18.04/ubuntu-18.04.1-desktop-amd64.iso.torrent
`´rtorrent@seedbox:~/#` rtorrent 

`**PRESS ENTER TO OPEN THE PROMPT!**

load.normal>` ~/ubuntu-18.04.1-desktop-amd64.iso.torrent

`**ENTER AGAIN & NOW PRESS THE CTRL+X AND PASTE THIS !**
command>` d.multicall2=,d.start=

`***so now at normally the file is downloading if yes, Great the rtorrent is working. 
*** Now you can check on /home/rtorrent/download if the file is here and you can delete it and also the *.torrent file in /home/rtorrent 

NOW PASTE THIS***

rtorrent@seedbox:~/#` su root
`root@seedbox:/home/rtorrent/#` cd ~/seedbox
`root@seedbox:~/#` nano /etc/systemd/system/rtorrent.service
````

### **Paste my seting and edit if required** 

```Markdown
[Unit]
Description=rTorrent
After=network.target

[Service]
User=rtorrent
Type=forking
KillMode=none
ExecStop=/usr/bin/pkill -e -n -f -x '/usr/bin/rtorrent -n -o import=/home/rtorrent/.rtorrent.rc'
ExecStart=/usr/bin/screen -d -m -fa -S rtorrent-tv /usr/bin/rtorrent -n -o import=/home/rtorrent/.rtorrent.rc
WorkingDirectory=%h

[Install]
WantedBy=default.target
```

**Back to root with CTRL+X and Y and press enter to save** 

```Markdown

´root@seedbox:~/#` systemctl daemon-reload
`´root@seedbox:~/#` systemctl enable rtorrent.service
`´root@seedbox:~/#` systemctl start rtorrent.service
```
**So this is the commands to stop , start  and restart**
```Markdown
´root@seedbox:~/#` systemctl stop rtorrent
`´root@seedbox:~/#` systemctl start rtorrent
`´root@seedbox:~/#` systemctl restart rtorrent
```

### Now it's time to install NodeJS before flood

```Markdown
´root@seedbox:~/#` curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
`´root@seedbox:~/#` sudo apt-get install -y nodejs
`´root@seedbox:~/#` nodejs -v
```
Your NodeJS version must be over or equal than v8.x.x.

### It's time to install Flood.

```Markdown
´root@seedbox:~/#` aptitude install npm
`´root@seedbox:~/#` apt-get install npm
`´root@seedbox:~/#` npm install -g node-gyp
`´root@seedbox:~/#` adduser --disabled-password flood
`´root@seedbox:~/#` cd /home/flood
`´root@seedbox:/home/flood#` git clone https://github.com/jfurrow/flood.git
`´root@seedbox:/home/flood#` cd flood
`´root@seedbox:/home/flood/flood#` cp config.template.js config.js
`´root@seedbox:/home/flood/flood#` nano config.js
```

***Change the value floodServerHost: '127.0.0.1' to floodServerHost: '0.0.0.0' to work outside, well now CTRL+X , Y  and Enter to save*** 

```Markdown
´root@seedbox:/home/flood/flood#` npm install 
`´root@seedbox:/home/flood/flood#` npm run build
`´root@seedbox:/home/flood/flood#` npm start
`´root@seedbox:/home/flood/flood#` cd ~/
`´root@seedbox:~/#` chown flood:flood -R /home/flood

```
