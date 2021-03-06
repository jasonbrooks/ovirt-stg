---
title: Phoenix Lab Ssh Spice Tunnel
category: infra
authors: dcaroest
wiki_category: Infrastructure
wiki_title: Infra/Phoenix Lab Ssh Spice Tunnel
wiki_revision_count: 2
wiki_last_updated: 2015-02-25
---

# Phoenix Lab Ssh Spice Tunnel

Heres a *hacky* way to setup the tunnel for spice to be used when clicking the engine spice button on fedora based machines.

## Requirements

You'll need the following extra packages:

    $ sudo yum install -y tsocks ssh remote-viewer

## Tunnel Configuration

Then you must setup the stunnel configuration like this:

    $ cat /etc/tsocks.conf
    server = 127.0.0.1
    server_port = 8181

## Getting the Engine Certificate

Download the engine ssl certificate:

    $ openssl s_client -connect monitoring.ovirt.org:443 \
          2&gt;/dev/null &lt;/dev/null \
      | openssl x509 &gt; engine.cert

## Replace the remote-viewer

Now replace the remote-viewer binary by the following custom script:

    $ remote_viewer_path=&quot;$(which remote-viewer)&quot;
    $ mv &quot;${remote_viewer_path}&quot;{,.orig}
    $ cat &gt;&gt;&quot;$remote_viewer_path&quot; &lt;&lt;EOS
    #!/bin/bash
    tsocks \
        &quot;${remote_viewer_path}&quot;.orig \
        --spice-ca-file=engine.cert \
        &quot;$@&quot;
    EOS

Make sure that the certificate points to the certificate you downloaded previously.

## Starting the Tunnel

Once done that, you'll have to start the ssh tunnel (you can do it automatically form bashrc or similar):

    $ ssh -fND 8181 youruser@foreman.ovirt.org

That will start the SSH tunnel in the background with a SOCKS proxy listening on 127.0.0.1:8181, where the tsocks connections will connect to.

## Bussines as Usual

So after all this hacky setup, you'll be able to connect to any vm in the phx engine using the spice link in the UI. Hopefully that will not be needed i the future once we have a better solution (vpn?).

<Category:Infrastructure>
