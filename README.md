# RailsBridge Virtual Machine

If you just want the VM image, **downloads are here**: http://downloads.railsbridge.org/

To workshop participants: this is the "behind the scenes" stuff that instructors use to create the virtual machine that you will use. You should install the virtual machine image file, not this code. Please follow [the instructions on the RailsBridge Boston site](http://docs.railsbridgeboston.org/installfest/) to set up your virtual machine.

To instructors and TAs: if you're interested in helping to maintain the VM, keep reading.

## Setup of the generated VM

The base box is Ubuntu 14.04 LTS; Ubuntu distributes version 4.3 of the VirtualBox guest additions. We should have students install the same version of the VirtualBox host.

The target versions are Ruby 2.0 and Rails 4.0 (the latest patchlevels available at build time). These are set in the `versions.sh`, which is used by the provisioning scripts.

If you want to run a workshop with Rails 3, there is a `rails3` branch (Ruby 1.9.3, Rails 3.2). It may be less up to date so check if there have been any security updates to Rails 3 before building it.

We use `chruby` to build/install Ruby, and invoke it in the user's `.bash_profile` to set their `PATH`. It provides a version of `gem` that defaults to user installs.

## Building a fresh image

The Vagrantfile will allow you to rebuild the RailsBridge VM from scratch. Run:

    vagrant up

Don't worry about the red text (Vagrant automatically colors all text printed to standard error). The base box will be downloaded directly from Ubuntu if it hasn't already been added.

Building Ruby takes a while, so get some coffee.

Then, to create an image file to distribute:

    rake package

This will create an image file name `railsbridgevm-version.box`, where the `version` is based on the tag you have checked out.

When you are ready to make the "gold master" version that we will ask students to download, create (and push to GitHub) a tag with the year and month of the workshop (e.g. `2014-01`) before running `rake package`. During the process of testing an image, you can run `rake package` with an untagged commit; the version will then include its SHA and how many commits it is ahead of the last tag.

## What to edit in this repo

To keep things simple and easy for everyone to modify, we use a shell script provisioner. There are three scripts:

* `provision-root-install.sh` runs as root (installs packages/Ruby system-wide)
* `provision-user-install.sh` runs as the `vagrant` user (installs gems to home directory)
* `provision-root-cleanup.sh` runs as root (removes files and zeroes out disk)

We also run the Heroku Toolbelt install script directly as root.

Files are copied into the VM from these directories:

* `binfiles` to `/usr/local/bin` (as root)
* `etcfiles` to `/etc` (as root)
* `dotfiles` to `/home/vagrant` (as the user)

## Rebuilding changes

If you make changes to the provisioning scripts, you should rebuild the VM from a clean image to ensure that everything is reproducible and that no manual changes have snuck in. To do this, run `vagrant destroy` and then `vagrant up` again.

It will re-run everything again, so grab some more coffee.

## Using the VM

There are a few extra scripts in the VM that you can use during the workshop:

* `railsbridge-update-dotfiles` will download and re-install all dotfiles from this repo (useful if they are mistakenly edited or deleted).
* `railsbridge-color off` will disable coloring of shell and IRB prompts (log out and back in to see the change). To re-enable color, run `railsbridge-color on`.
* `showargs` can be used to explain how command-line arguments work.
* `vagrant` prints a message explaining that you're in the VM and should exit if you want to run a Vagrant command.

## Tests

TODO: There should be some kind of automated test for the output (i.e. can you start the VM, log in, clone a test Rails app, and run it).
