# Running a laptop as a server

*TL;DR: I hoard electronics. Laptops as servers need configuration to avoid death-by-lid.*

Every once in a while I buy my wife the latest, greatest laptop device for her needs
(which for the last decade has basically meant getting her a nice solid Chromebook every 3 or so years).
In trade, I usually keep her old laptop, wipe it clean and try to eventually find creative new uses for it.

## The case of the Dell Inspiron 1521

In 2007, we had just moved to the U.S. but my wife was still pretty far from her family, so a laptop was still a necessity for her to stay in touch. So for Christmas, she got a Dell Inspiron 1521.

In 2010, the Inspiron 1521 laptop was already showing its age, so for Christmas, my wife got a first-of-its-kind Cr-48 Chromebook and the old laptop was dismissed from service.

I reinstalled the Windows OS but a 4GB RAM, dual-core laptop has its limits and it just wasn't useful - so it sat on the shelf...

Recently, though, I picked it up again to give the laptop a new purpose ... as a linux server for mild computing applications.
I'm still not sure what I'll run on it, but I like having linux servers with which to tinker, so here we are.

First things first: The old spindle HDD had to go - I replaced it with a Corsair MX500 SSD for which I found a deal. This gives me plenty of storage for VMs, containers, etc.

Everything else is stock, though, due to budget and practicality constraints.

As for OS: I went the Debian route; I had recently set up some Debian VMs on my main box and part of that journey was setting up `apt-cacher-ng`, so the trail was already blazed for me.

### Servers don't have lids

Repurposing a laptop as a server means dealing with some features that typical servers don't have. The main one so far was the lid: by default, the OS triggers a hibernate when the laptop lid gets closed. Death-by-lid!
I really want to avoid having a giant open *(red)* clamshell as an eyesore in my server hub.

#### Disable hibernate and turn the screen off/on when the lid closes/opens

After the initial installation and driver fixes, I confronted my death-by-lid situation. Disabling hibernate-by-lid was straightforward. However, the intermediate result caused the console display to remain on even when the lid was closed; this is not energy-efficient and will eventually burn out the screen.

I eventually arrived at the following solution:

```shell
# Disable hibernate
systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
# Install power and display management (vbetool since we're don't have X installed)
apt install -y acpid vbetool
# Configure ACPId lid action
mkdir -p /etc/acpi/actions/
cat <<EOF >/etc/acpi/actions/lid.sh
grep -q closed /proc/acpi/button/lid/LID/state && (vbetool dpms off)
grep -q open /proc/acpi/button/lid/LID/state && (vbetool dpms on)
EOF
chmod a+x /etc/acpi/actions/lid.sh
# Configure ACPId lid event to trigger action
cat <<EOF >/etc/acpi/events/lid
event=button/lid
action=/etc/acpi/actions/lid.sh %e
EOF
# Restart ACPId
service acpid restart
```

That's it - now we don't have to worry about burning in the LCD display and the server is a little more energy efficient.
