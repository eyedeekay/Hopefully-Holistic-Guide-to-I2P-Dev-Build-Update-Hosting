# Hopefully Holistic Guide to I2P Dev Build Update Hosting

The process of setting up an I2P update server, complete with I2P torrents
for updates, is surprisingly straightforward, but there isn't really a document
where every step is laid out from end-to-end. Having done it a couple times, I
suppose I'm the apt person to write such a document, with the proposed target
being a dev build that you host yourself or for a small audience of testers.

## What you need to get started:

- An I2P router installed on your system.
- `python3`, `pip3` and `bash` for generating the news feeds
- Virtualenv or Docker to isolate the feed generator dependencies(optional but
 a good idea)
- `git` for obtaining software from the internet in source form

If you're on Debian or Ubuntu linux, this will satisfy the dependencies:

```sh
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
./i2pk -p $HOME/i2p-dev-keys \
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

1. To get started, clone the news server software.

```sh
git clone https://github.com/i2p/i2p.newsxml
```

2. Next, you'll need to set up the news server software. If you got all the dependencies
from "What you need to get started" you should be able to run the following commands:

#### If you want to use `virtualenv`

Using `virtualenv` is recommended for anyone who wants to build newsfeeds on their
system.

```sh
./setup_venv.sh
. env/bin/activate
pip install .
```

#### If you want to use `docker`

Docker is also capable of isolating these dependencies and can be used instead of
`virtualenv`.

```sh
You're done, actually. The container has all the dependencies managed automatically.
```

#### If you want to use neither

As long as you aren't using the `newsfeed` package from python anywhere else,
you should also be able to just `pip install` the dependencies. This is the least
stable way to do things and only recommended if you host your newsfeed in a dedicated
VM.

```sh
pip install .
```

3. After you've done that, you're adequately configured

### Setting up a Download Server

### Signing your Update

### Setting up a Bittorrent Tracker
