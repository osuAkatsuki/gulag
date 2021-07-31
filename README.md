[![Discord](https://discordapp.com/api/guilds/748687781605408908/widget.png?style=shield)](https://discord.gg/ShEQgUx)

gulag is my implementation of an osu! server's backend (bancho protocol, /web/
endpoints, avatars, static assets, and a developer rest api). it's designed as
a modern substitution for existing osu! server projects who've become inactive
or who've gone closed-source. i aim to make this project the ideal choice for
running osu! private servers, and to fully support osu!'s protocol, while
abusing it a little to get as much as we safely can out of it. :)

i'll set the stage with a brief introduction; i've been playing actively on osu!
private servers since early 2016, and founded [akatsuki](https://akatsuki.pw) in
late 2017 using [ripple](https://github.com/osuripple)'s source with no prior
development experience. i spent nearly all of my spare time learning more about
the wonderful world of (osu!) programming by working on their codebase, until
i had eventually learned enough to try writing a server from the ground up.

gulag was my third attempt to write an osu! server, and based on my previous
failures i didn't believe i had what it took to write a competitive server; but
after months of development and gradual progress it's seemed more and more
likely that it'd really become the replacement for akatsuki's existing stack.

in my opinion, i think it's become the most maintainable, thoughtfully efficient
and well suited for the operation of an osu! private server of anything i've seen.
it's certainly not the fastest, but for the level of abstraction it's written at
it's certainly much more efficient than any alternative options. there are some
other great projects out there (especially some of the more modern ones), but
frankly i've spent countless hours thinking about how i can improve the server
and make it superior in a wide variety of different ways, not being scared to
tear down & refactor large structures in the codebase as i learn more.
i really think it's paid off, and i can say without a doubt that i plan to
use this software for akatsuki. of course everyone has their biases and values
so i recommend you take a good look at all the options and come to an informed
decision if you're planning on running a server!

the project has also seemingly increased activity in the osu! private server
development community over the last year and many other open source osu! server
projects have popped up, such as [peace](https://github.com/Pure-Peace/Peace),
[kuriso](https://github.com/osukurikku/kuriso), and many other attempts have
been made by less experienced developers to write servers as a means to learn
more about programming as a whole, which has thankfully become more possible
due to the increased amount of documentation & examples (such as gulag) available.

at the moment, gulag's still in development and if you're running a serious
instance of a private server in production, i wouldn't yet recommend switching
your stack as there is still work to be done; but take a look around and see
what you think. it's certainly fine for little friends-only servers or if
you're just looking to play around; if you're running one of these, please
consider joining our discord, we really need all the testing we can get. :D

contributions are welcome but please don't feel like they're required; gulag's
mostly my baby and i really want to get the master implementation as close to
perfection as i can. if you're a dev and want to contribute, i'd strongly
recommend forking the repository and playing around on a server of your own,
this is how all previous high quality contributions have come into fruition.

there is currently no official frontend project for gulag. the most serious
attempt to date is [gulag-web](https://github.com/Varkaria/gulag-web), though
there is a pretty decent chance that [varkaria](https://github.com/Varkaria) and
i will be doing some major refactoring in the future before moving akatsuki onto
the new stack; the code will need to be of comparable quality to what you see here.


# Installation Guide

**Important notes:**
- gulag currently does not work on most other distros besides ubuntu 18.04 and 20.04 due to an issue with the liboppai library.
- osu uses the old deprecated TLSv1.0 so you will need to make sure your reverse proxy supports TLSv1.0. The included nginx configuration should enable TLSv1.0 on 20.04.

## Install requirements
Ubuntu 20.04 has python3.9 so you can simply do:
```sh
sudo apt install python3.9 python3-pip build-essential mysql-server nginx python3-certbot python3-certbot-nginx
```
For 18.04 you'll need to use the ppa:
```sh
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.9 python3.9-dev python3.9-distutils build-essential mysql-server nginx python3-certbot python3-certbot-nginx
wget https://bootstrap.pypa.io/get-pip.py
python3.9 get-pip.py && rm get-pip.py
```
Clone the repository and install pip requirements:
```sh
git clone https://github.com/cmyui/gulag.git --recursive && cd gulag
sudo python3.9 -m pip install -r ext/requirements.txt
```

## Set up mysql
Create an empty database for gulag: run `sudo mysql` to open a mysql shell and run these commands:
```sql
CREATE DATABASE gulag;
CREATE USER 'gulag' IDENTIFIED BY 'make_a_random_password_here';
GRANT ALL PRIVILEGES ON gulag.* TO 'gulag';
FLUSH PRIVILEGES;
exit
```
Import gulag's sql structure:
```sh
mysql -p -u gulag gulag < ext/db.sql
```

## Set up nginx and certificates
Create a TLS certificate for your domain:
```sh
D=your.domain certbot certonly --nginx -d osu.$D,a.$D,b.$D,c.$D,c4.$D,c5.$D,c6.$D,ce.$D,assets.$D
```
Copy and edit the nginx configuration with your domain:
```sh
sudo cp ext/nginx.conf /etc/nginx/conf.d/gulag.conf
sudo nano /etc/nginx/conf.d/gulag.conf
sudo nginx -s reload
```

## Configure and run gulag
Copy and edit gulag's config file:
```sh
cp ext/config.sample.py config.py
nano config.py
```
Test the server:
```sh
./main.py
```

## Install gulag as a daemon for production
Move gulag to a system-wide location and create a system user for it (optional, recommended for security but may make things harder to work with)
```sh
sudo mv gulag /srv/gulag # don't forget to update paths in nginx
sudo useradd -r gulag
sudo chown -R gulag:gulag /srv/gulag
sudo chmod 640 /srv/gulag/config.py # protect credentials
```
Copy and edit the systemd service file:
```sh
sudo cp ext/gulag.service /etc/systemd/system/
sudo nano /etc/systemd/system/gulag.service
sudo systemctl daemon-reload
```
Make sure gulag isn't running elsewhere. Enable and start the service:
```sh
sudo systemctl enable --now gulag
```
You can view log output with `tail -f /var/log/gulag.log`.


# Directory Structure

    .
    ├── constants  # code representing gamemodes, mods, privileges, and other constants.
    ├── ext        # external files from gulag's primary operation.
    ├── objects    # code for representing players, scores, maps, and more.
    ├── utils      # utility functions used throughout the codebase for general purposes.
    └── domains    # the route-containing domains accessible to the public web.
        ├── cho    # (ce|c4|c5|c6).ppy.sh/* routes (bancho connections)
        ├── osu    # osu.ppy.sh/* routes (mainly /web/ & /api/)
        └── ava    # a.ppy.sh/* routes (avatars)
