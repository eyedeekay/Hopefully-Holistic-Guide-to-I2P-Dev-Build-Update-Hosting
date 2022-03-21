# Hopefully Holistic Guide to I2P Dev Build Update Hosting

The process of setting up an I2P update server, complete with I2P torrents
for updates, is surprisingly straightforward, but there isn't really a document
where every step is laid out from end-to-end. Having done it a couple times, I
suppose I'm the apt person to write such a document, with the proposed target
being a dev build that you host yourself or for a small audience of testers.

## What you need to get started:

- An I2P router installed on your system.
- `python3` and `bash` for generating the news feeds
- Virtualenv or Docker to isolate the feed generator dependencies(optional but
 a good idea)

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

Next

### Setting up a Download Server

### Setting up a Bittorrent Tracker
