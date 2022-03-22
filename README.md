# Hopefully Holistic Guide to I2P Dev Build Update Hosting

The process of setting up an I2P update server, complete with I2P torrents
for updates, is surprisingly straightforward, but there isn't really a document
where every step is laid out from end-to-end. Having done it a couple times, I
suppose I'm the apt person to write such a document, with the proposed target
being a dev build that you host yourself or for a small audience of testers.

## What you need to get started

- An I2P router installed on your system.
- `python3`, `pip3` and `bash` for generating the news feeds
- `virtualenv` or `docker` to isolate the feed generator dependencies(optional but
 a good idea)
- `git` for obtaining software from the internet in source form

```sh
# If you're on Debian or Ubuntu linux, this will satisfy the dependencies:
sudo apt-get install python \
    python-dev \
    python-virtualenv \
    libxml2-dev \
    libxslt1-dev
```

In other systems, you'll need to determine the similar steps to install this
software.

### Setting up your Signing Keys

In order to distribute updates, you should set up signing keys in order to
authenticate the downloads. It's also important in order to carry out the whole
process, end-to-end, in order to understand how it all works. In order to
generate your plugin signing keys, use the following steps:

1. Download the helper script. This wraps around some of the key generation
options to make them easier to use. The helper script is available
[at this link](https://raw.githubusercontent.com/eyedeekay/Generate-Plugin-Signing-Keys/master/i2pk)
and can be viewed [on my github](https://github.com/eyedeekay/Generate-Plugin-Signing-Keys/).
2. Generate your newsfeed signing keys by running the following command from the directory where
you downloaded `i2pk`.

```sh
./i2pk -p $HOME/.i2p-plugin-keys \
    -s you@mail.i2p \
    -t RSA_SHA512_4096 \
    -n newssigner \
    -c router generate_keys
cp ~/i2p/certificates/router/you_at_mail.i2p ~/i2p/certificates/news/you_at_mail.i2p
```

And you're all done. Your new signing keys are ready to use for both `news` and `router`
updates on your own system. These updates will not work for people who do not choose to
trust your keys.

### Setting up a News Server

Next, you'll need to set up a News Server. In I2P, a News Server is a hidden server which
serves up XML files in signed, zipped bundles. You sign these bundles with the keys we
generated in the previous step.

#### `docker` and `virtualenv`

It is highly recommended that you isolate the python dependencies you need using either
`docker` or `virtualenv`, depending on your preference.

1. To get started, clone the news server software.

```sh
git clone https://github.com/i2p/i2p.newsxml
```

2. Next, you'll need to set up the news server software. If you got all the dependencies
from "What you need to get started" you should be able to run the following commands:

##### If you want to use `virtualenv`

Using `virtualenv` is recommended for anyone who wants to build newsfeeds on their
system.

```sh
# This is the "official" way
./setup_venv.sh
. env/bin/activate
pip install .
```

##### If you want to use `docker`

`docker` is also capable of isolating these dependencies and can be used instead of
`virtualenv`.

```sh
# Docker handles things automatically by wrapping up the official steps in a container
The container has all the dependencies managed automatically, you do not need to do anything.
```

##### If you want to use neither

As long as you aren't using the `newsfeed` package from python anywhere else,
you should also be able to just `pip install` the dependencies. This is the least
stable way to do things and only recommended if you host your newsfeed in a dedicated
VM.

```sh
# This is usually not a very good idea
pip install .
```

3. After you've done that, you're adequately configured to build news feeds. However,
you still aren't set up to serve the news feeds to people. Since we're not ready to
do that for real yet, to practice, let's build the existing feeds, and copy them over
to your web server.

#### Configuring the newsfeed generator

The newsfeed generator can use a configuration file, called `etc/su3.vars.custom`, which
you should edit.

```sh
# Install dir
I2P=/home/you/i2p
# private key keystore
KS=/home/you/.i2p-plugin-keys/newssigner-su3-keystore.ks
# signer
SIGNER=you@mail.i2p
```

If you're using `docker`, you should use `etc/su3.vars.custom.docker` instead, and omit
the `$HOME` directory from the config lines.

```sh
# Install dir
I2P=/i2p
# private key keystore
KS=/.i2p-plugin-keys/newssigner-su3-keystore.ks
# signer
SIGNER=you@mail.i2p
```

##### If you used `pip` or `virtualenv` in the last step

If you used `pip` or `virtualenv` to set up your feed generator, then you need to
generate the feeds and `.su3` bundles first, then manually copy them to the web server
you want to use. In this case we're going to assume the one that comes with I2P, but
basically any web server will works. The first step will generate the `./build` directory and
all the files that you need to serve. After it runs, take a moment to examine the contents of
`./build`. After the second command runs, the feeds can be previewed at `http://localhost:7658`.

1. Start by building the feeds. Make sure you're in your `virtualenv` if you're using it,
and run the command:

```sh
./new.sh
```

2. If you used `pip` or `virtualenv` then we will also be using the web server that is
built into I2P. When this step is complete, the feeds can be previewed at
`http://localhost:7658`.

```sh
# Use this command if you're on Debian or Ubuntu a service
sudo -u i2psvc cp -r ./build /var/lib/i2p/i2p-config/eepsite/docroot/news
```

```sh
# Use this command if you're using a "User" install or the `.jar` package
cp -r ./build $HOME/.i2p/eepsite/docroot/news
```

##### If you used `docker` in the last step

All you need to do is the following two commands. The first one will generate the
`./build` directory and all the files that you need to serve. After it runs, take
a moment to examine the contents of `./build`. The second command takes the build
directory and places it in the document root of a light web server, which will serve
the built feeds on `http://localhost:3000`.

1. The first command generates the feeds and turns them into signed bundles. This will
prompt you for a password.

```sh
./docker-news.sh
```

2. The second command runs the web server and serves the feeds on `localhost:3000`

```sh
./docker-newsxml.sh
```

Now you're ready to host your news feeds.

### Signing your Update

Now you're prepared to build and sign newsfeeds, however, you still need to build
and sign the updates themselves. How you build the updates is going to depend on
how you build your app, but for the first example we're going to assume you're building
an I2P router from the `i2p.i2p` source. In `i2p.i2p` there is an `ant` target that
runs the signing command, you can use it with:

```sh
ant signed-updater200
```

1. When you run this command, you'll be prompted to enter your signing key details. When you
see:

```make
-sign-update:
    [input] Enter su3 private signing key store:
```

2. When you see:

```make
[input] Enter su3 key name (you@mail.i2p):
```

3. Finally, when you see:

```make
[input] Enter su3 key password for you@mail.i2p:
```

Enter the password for your keystore. You will end up with a signed update file named
`i2pupdate.su3` in your directory. That is your signed update file, which you will place
in your update server in the next step.

There are different kinds of I2P updates now, registered via the `UpdatePostProcessor`
system. You can, for example, make an "Executable" `UpdatePostProcessor` or a "DMG"
`UpdatePostProcessor` instead. When you generate an `.su3` for such a file, you may
once again use `i2pk` to make the process easier. For example, to build an Executable
based updater, use:

```sh
./i2pk -p $HOME/.i2p-plugin-keys \
    -s you@mail.i2p \
    -t RSA_SHA512_4096 \
    -n newssigner \
    -f EXE \
    -c router sign i2pupdate.exe
```

### Setting up a Download Server

Now you're able to notify your users that there is an update, but you need somewhere
to host the actual download. For the purposes of our tutorial, we'll assume you're
using the I2P site included with the Java I2P router, but any web server will do. This
will serve as a reliable backup distribution point for your I2P updates. Copy
`i2pupdate.su3` to the document root.

```sh
# Use this command if you're on Debian or Ubuntu a service
sudo -u i2psvc mkdir -p /var/lib/i2p/i2p-config/eepsite/docroot/files
sudo -u i2psvc cp ./i2pupdate.su3 /var/lib/i2p/i2p-config/eepsite/docroot/files
```

```sh
# Use this command if you're using a "User" install or the `.jar` package
mkdir -p $HOME/.i2p/eepsite/docroot/files
cp ./i2pupdate.su3 $HOME/.i2p/eepsite/docroot/files
```

If you don't want to use that, any static file server with an I2P tunnel pointed at
it will do. For example, you can use this:

```sh
# This is a suggestion for people who want to use docker, for instance
mkdir -p "$HOME/update-server"
cp ./i2pupdate.su3 "$HOME/update-server"
docker run -d \
    -v "$HOME/update-server":/web \
    -p 8080:8080 \
    halverneus/static-file-server:latest
```

or any other web server you want.

### Setting up a Bittorrent Tracker

Strictly speaking you don't need to set up your own tracker, but I think you should.
It's easy and reliable to host one using the `zzzot` plugin on a Java I2P router.
```zzzot``` is hosted at [http://stats.i2p/](http://stats.i2p) can be installed by
pasting the URL below into the **`Installation from URL`** field on
[the Router Plugins page](http://127.0.0.1:7657/configplugins):

```
http://stats.i2p/i2p/plugins/zzzot.su3
```

and clicking the **`Install plugin`** button.

### Self-Publishing your update

Finally, now that you've got the news server, the news feed generator, the download
server, and an updater published, you're ready to create the last thing you need, your
custom newsfeed. This will **replace** the example newsfeed we built earlier. To do
this we'll need to generate a torrent of our router update using our open tracker,
generate a `releases.json` file describing where to find the torrent and the update
packages, and write a news entry for your router update.

1. Using the `.su3` package you generated in the "Signing your Update" step, generate
a .torrent of your update and convert it to a magnet link. Every torrent generator I have
tried is capable of generating an I2P torrent, they aren't substantially different from
regular torrents for the most part. I often use `mktorrent` because most package managers
have it and because it's pretty easy to automate.

```sh
# Using mktorrent
mktorrent --announce="http://yourzzzotopentrackerhasabase32urlthatisprettylong.b32.i2p/a" \
   --web-seed="http://yoursite.i2p/files/i2pupdate.su3" \
   i2pupdate.su3
```

2. Load the file named `i2pupdate.su3.torrent` into I2PSnark without starting it. Copy the
`i2pupdate.su3` file into the `i2psnark` directory, then start the torrent from the I2PSnark
webUI. The torrent will appear as fully downloaded, and you will be ready to seed. As a last
step, copy the "Magnet Link" for your `.su3.torrent` from the WebUI.

```sh
# Use this command if you're on Debian or Ubuntu a service
sudo -u i2psvc cp ./i2pupdate.su3 /var/lib/i2p/i2p-config/i2psnark
```

```sh
# Use this command if you're using a "User" install or the `.jar` package
cp ./i2pupdate.su3 $HOME/.i2p/i2psnark
```

3. Next we need to create the `releases.json` file which will be used to generate part of our
newsfeed that informs I2P routers that they need to update, and where to get those updates
from. You will need to add this file to your `i2p.newsxml` checkout. A `releases.json` file
contains: the date the torrent was created, the minimum version required to apply the automatic
update, the version that I2P will be updated to, and the download locations. This is a recent
example of an updates.json file:

```json
[
  {
    "date": "2022-02-21",
    "version": "1.7.0",
    "minVersion": "0.9.9",
    "minJavaVersion": "1.8",
    "updates": {
      "su3": {
        "torrent": "magnet:?xt=urn:btih:f49db26dd75f61d8d2b4a187bc3d61d71c6dee09&dn=i2pupdate-1.7.0.su3&tr=http://tracker2.postman.i2p/announce.php",
        "url": [
          "http://stats.i2p/i2p/1.7.0/i2pupdate.su3",
          "http://mgp6yzdxeoqds3wucnbhfrdgpjjyqbiqjdwcfezpul3or7bzm4ga.b32.i2p/releases/1.7.0/i2pupdate.su3"
        ]
      }
    }
  }
]
```

Note the `"torrent:"` field, where you should place the generated magnet link you copied from the I2PSnark
Web UI, and the `"url"`
