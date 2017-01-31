### Machine issues (VirtualBox)
If you keep experiencing VM creation issue, install the [test VirtualBox software](https://www.virtualbox.org/wiki/Testbuilds), as it has been shown to be a viable solution for some. 

1. Uninstall any existing VirtualBox using the official Uninstall.tool You may need to reboot if certain kext files are not properly unloaded.
2. Run Kitematic [reset script](https://github.com/kitematic/kitematic/blob/master/util/reset)
3. Install [VirtualBox 5.x Testing](https://www.virtualbox.org/wiki/Testbuilds) 
4. Install [Docker Toolbox](https://www.docker.com/toolbox)

If using Docker Toolbox on Windows, check that the account running Kitematic has set `DOCKER_TOOLBOX_INSTALL_PATH = C:\Program Files\Docker Toolbox` (or whatever the actual path is). If you installed as admin, this is likely not set for your user account.

### Kitematic VM Stuck at 99% or Cannot pull images

**Windows Users** Please make sure that your anti-virus/security/firewall allows connection to/from the VM ip 192.168.99.100

For previous windows user, assume 'default' to be 'kitematic'. 

1. Turn off your VM via the following terminal command `docker-machine stop docker-vm` (Docker commands must be installed for this to work)
2. Open your VirtualBox app and go to your Preferences (app preferences, not VM settings)
3. Click on Network, and Host-Only Networks tab
4. You should see a few vboxnet entries, for each one do
  1. Click on vboxnet entry and then click on the screw-driver icon (edit)
  2. Click on DHCP Server tab
  3. On the selected vboxnet, check if `DHCP Server` is checked - if it is, check if the `Server address` is: `192.168.99.1` - If NOT, just click Cancel and move on to the next one.
5. Write down which vboxnet0, vboxnet1, vboxnet2 (however many) has the proper setting of `DHCP server` checked and a `Server address` of `192.168.99.1` and cancel out of the Preferences
6. Select the 'default' VM
7. Click Settings then Network
8. Adapter 2 should be a Host-Only Adapter, and change its name to be the vboxnet you noted above.
9. Click OK and close VirtualBox app
10. In your terminal run: `docker-machine start default` 

If you were stuck at 99% you may need to regenerate the certs:

1. `docker-machine regenerate-certs default`
2.  You can now start Kitematic and see it working :)

_If none of you vboxnet have the proper setup, you can change the one that your VM uses to have the proper server address of `192.168.99.1`_

**SSH Multiplexing**

If you've enabled SSH Multiplexing, it might be the cause of this problem.  As pointed out [in this GitHub issue](https://github.com/kitematic/kitematic/issues/386#issuecomment-130421161) disabling multiplexing for localhost resolved the issue for some people.

### Windows 10

Virtualbox seems to have a bug in Windows 10 and host-only adapter (mentioned above), a fix/patch exists at the following location: [Windows host-only adapter creation fails due to slow background processing](https://www.virtualbox.org/ticket/14040) - The latest Windows 10 build is causing that the Virtualbox Host Only adapter, is not checking the "Virtualbox NDIS6 Bridged Networking Driver" so the default machine cannot start properly.

Another possible fix is to enable Virtualization in the Bios (VT-X) - thanks to @SkiftCreative for the tip:
* In windows 8/10 you can get there by clicking the power menu, holding [SHIFT] then hitting restart. 
* Keeping [SHIFT] held until the screen changes
* Go into advanced settings, then change EUFI settings (were not changing those settings, were just getting into BIOS).
* Once in BIOS enable virtualization, then save and restart.

Make sure that the Hyper-V daemon is **not running**, as it will conflict with Virtualbox installation/setup.

**MS Edge can't see website:**
Go to Start, type "Internet Options" then then the Security Tab, then click Local Intranet, then Sites. Add your Virtual Machine's IP (in this case, the Docker Host) in that list and you're golden.
![](http://www.hanselman.com/blog/content/binary/Windows-Live-Writer/How-to-get-Microsoft-Edge-to-see-your-Vi_FDE5/image_3.png)

_thanks to http://www.hanselman.com/blog/FixedMicrosoftEdgeCantSeeOrOpenVirtualBoxhostedLocalWebSites.aspx_

In certain cases, the network driver can be disabled by some windows updates.
Following the above steps to get to networking and Enabling "**Virtualbox NDIS6 Bridget Networking Driver**" seems to solve most of these issues.
If the networking driver was disabled, VirtualBox should work just fine now (no reboot needed). Otherwise repeat for every network adapter you have (Ethernet, WiFi...) and always uncheck the NDIS6 checkbox -> apply -> check it again -> apply.

TL;DR:

1. Open network center
2. Click "Change adapter settings"
3. Right click your Virtualbox host-only adapter and select Properties
4. Enable "Virtualbox NDIS6 Bridget Networking Driver"


### Exec shell

Kitematic uses sh for the Exec function. This can cause unwanted behavior, such as up arrows being displayed as '^[[A' in Powershell terminal. This is unavoidable, because not all docker containers have a bash shell (or csh, zsh, etc.). One way to use the bash shell, which does not have these limitations, is simply to type "bash" into the terminal each time that a new session is started.

If you want your bash shell to be the default shell for when you 'exec' into the container, then in Settings > Advanced, simply add an entry for SHELL with a value of /bin/bash. [[1]]( https://github.com/docker/kitematic/issues/1022)

Note that Windows containers do not have sh so this. (Open issue: https://github.com/docker/kitematic/issues/2166).

### 403 Errors

If you're a new user, you will need to verify your Dockerhub email address before installing images will work.

See Issue [#789](https://github.com/kitematic/kitematic/issues/789).

### Cannot Download Image/Connect
This could be a simple DNS issue, try the following:

1. Click on the Docker CLI button 

![](https://cloud.githubusercontent.com/assets/3325447/7950182/0ae55b3c-094c-11e5-859b-3acf43df7c34.png)

2. Type in `docker-machine ssh default`
3. From the terminal prompt, type in `echo "nameserver 8.8.8.8" > /etc/resolv.conf`


### IP/Subnet issues

Thanks to @kriscarle for the steps:

1) In a terminal, run `docker-machine rm default` in order to remove the existing VM

2) Go into VirtualBox preferences and remove the host-only network. (steps shown above)

3) In a terminal, run `docker-machine create --driver virtualbox --virtualbox-hostonly-cidr "192.168.59.1/24" default` - This will create the VM on a different subnet (`192.168.59.1/24` is used as an example but any valid subnet will work.)

### Proxy issues:
Move to: [Common Proxy issues & Fixes](https://github.com/kitematic/kitematic/wiki/Common-Proxy-Issues-&-Fixes)


### Running out of space
By default the VM is created with 20GB of disk space. If you would like to create a VM with more disk space, the following command will do just that and create a 100GB VM:

`docker-machine -D create -d virtualbox --virtualbox-disk-size "100000" default`