---
title: Repository mirrors
category: infra
authors: dcaroest
wiki_category: Infrastructure
wiki_title: Repository mirrors
wiki_revision_count: 8
wiki_last_updated: 2014-06-27
---

# Repository mirrors

## Mirroring oVirt repositories

Do you want to become a mirror? Do you have some bandwidth to spare?

Drop us a line at [infa](mailto:infra@ovirt.org) and tell us about yourself! You just need to pass us a public ssh key, and once set up you'll be able to rsync from our site with the following command:

       rsync -rltHvvzP mirror@resources.ovirt.org:/var/www/html destination/dir

After some validation we will add your site to this page and to the mirrorlist!

**Thanks a lot for your bandwidth!**

## For admins

We have a simple setup to allow mirroring of the oVirt repositories. Right now the mirroring is done through ssh with rsync. You'll find in **resources.ovirt.org** a user named **mirror**, that's the user that the mirrors should use, you'll see that it has a lot of entries under *~/.ssh/authorized_keys*, there each entry is restricted to run only a specific command that allows them to rsync the repo directory.

### Adding a mirror

To add a mirror you just need to add it's public ssh key in *~mirror/.ssh/authorized_keys*, with the command restriction as the other entries. Then when the mirror is confirmed you can add it to the mirrorlist (*resources.ovirt.org:/var/www/html/pub/yum-repo/mirrorlist*) and to this wiki.

## Current mirrors

| Name                                                                  | Location                       | URLs                                                                                                      | Contact                         | Other                                                                           |
|-----------------------------------------------------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------------|---------------------------------|---------------------------------------------------------------------------------|
| [NLUUG](http://www.nluug.nl)                                          | The Netherlands (Amsterdam)    | [http](http://ftp.nluug.nl/os/Linux/virtual/ovirt/) [ftp](ftp://ftp.nluug.nl/pub/os/Linux/virtual/ovirt/) | ftpmirror-beheer_at_nluug.nl  | Syncing on all odd hours, bandwidth is currently 4 Gb/s and we do IPV4 and IPV6 |
| [AARNet Pty Ltd](http://www.aarnet.edu.au/)                           | Australia                      |                                                                                                           | alex.dodson_at_aarnet.edu.au  |                                                                                 |
| [Studenten Net Twente (SNT)](http://www.snt.utwente.nl/)              | The Netherlands                | [http](http://ftp.snt.utwente.nl/pub/software/ovirt/) [ftp](ftp://ftp.snt.utwente.nl/pub/software/ovirt/) | ftpcom at snt.utwente.nl        |                                                                                 |
| [Georgia Tech Software Library (GTlib)](http://www.gtlib.gatech.edu/) | United States (Georgia)        | [http](http://www.gtlib.gatech.edu/pub/oVirt/pub/) [ftp](ftp://www.gtlib.gatech.edu/pub/oVirt/pub/)       | neil.bright_at_oit.gatech.edu |                                                                                 |
| [ibiblio](http://www.ibiblio.org/)                                    | United States (North Carolina) | [http](http://mirrors.ibiblio.org/ovirt/pub/) [ftp](ftp://mirrors.ibiblio.org/ovirt/pub/)                 | cmpalmer_at_ibiblio.org       |                                                                                 |
| [Rochester Institute of Technology](http://www.rit.edu)               | United States (New York)       | [http](http://mirrors.rit.edu/ovirt/pub/) [ftp](ftp://mirrors.rit.edu/ovirt/pub/)                         | pfmeec_at_rit.edu             |                                                                                 |
| [Duke University](http://duke.edu)                                    | United States (North Carolina) | [http](http://mirror.linux.duke.edu/ovirt/pub/) [ftp](ftp://mirror.linux.duke.edu/ovirt/pub/)             | drew.stinnett_at_duke.edu     |                                                                                 |

<Category:Infrastructure>
