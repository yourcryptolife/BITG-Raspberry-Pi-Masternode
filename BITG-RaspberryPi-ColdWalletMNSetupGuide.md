## Raspberry PI Bitcoin Green Cold Wallet Masternode Setup Guide ##

**DISCLAIMER: This is not financial advice. The author accepts NO RESPONSIBILITY for ANY LOSS OF DATA or CAPITAL that may occur as a result of following and implementing the information contained in this guide. By reading and/or implementing the information contained herein, you agree to accept full responsibility for your actions, and shall assume the full risk of any liability arising from your own conduct. USE AT YOUR OWN RISK!**

This guide is a work in progress. Feedback to help improve it and donations towards its upkeep are all welcome :)

> $BITG Bitcoin Green Donation Address: **GXtS1QCPsMANpRDQa5xFwogfbxT42dy5uV**

> $BTC Bitcoin Donation Address: **1AxzU81tw8rBL9vRyGdhWL4s4C8BusffP7**


## Overview Of The Masternode Setup ##

This is a guide for setting up a Bitcoin Green Cold VPS Masternode using a Raspberry Pi3 to host the controller wallet & a Vultr Ubuntu 16.04 VPS to act as the public facing MN server. 

I decided to try this approach as I wanted to be able to keep my controller wallet open 24/7 so that it could earn staking rewards on top of the masternode rewards. 

To do this on my personal laptop wouldn't work, and as there was a Pi wallet released for BITG, I decided to see if it would work.

The setup builds on top of the Bitcoin Green wallet setup guide & masternode setup script (https://goo.gl/S7fKdP) provided in the Bitcoin Green Discord channel (https://discord.gg/g3CFth) by XeZZoR (thanks mate).

Essentially I have just setup the controller wallet on the Pi instead of my personal laptop. The VPS masternode server setup is exactly the same as XeZZoR's guide and uses his script, so if you are familiar with that, this should be quite easy to follow.

## Dangers to be aware of with the Pi ##

> NOTE: I'll add to this list as I discover any more shortcomings ;)

Using the Pi for crypto minting does come with some added things to take care of to make sure your coins are safe.

Most importantly, the Pi runs off an SD card, and these are known to fail more regularly than your standard HDD, so regular backups of your wallet.dat file to another drive are VERY IMPORTANT to prevent any loss of your $BITG coins!

We also need to do some basic security to the Pi. Mine run behind my home routers firewall, but I do secure them further by running some standard security basics.

This setup has 2 benefits with regards to running the masternode:

1. **Coin Safety**. Because it stores your coins locally (on the Pi), it protects your masternode investment from being stolen by hackers should they manage to hack the masternodes public facing IP address (the VPS server). There is no direct connection between the 2 machines.

> NOTE: this also assumes that you ensure the Pi has at least some basic intrusion protection by following the instructions in this guide. It also assumes that you follow the backup instructions. I have included a backup script for backing up the wallet.dat file from the masternode wallet on the Pi. These are written to a seperate USB thumb drive.

2. **Optimised ROI** In addition to earning masternode rewards, this setup allows you to earn staking rewards by leaving the Pi online 24/7, and keeping the wallet on the Pi open all the time - encrypted and locked, but open for staking.

This allows you to optimise the return of your masternode as you will earn both masternode rewards and staking rewards.

**Before we get started, you will need the following items:**

1. Raspberry Pi 3 to host your controller wallet and coins
2. At least an 8GB micro SD Card with Noobs installed
3. A Pi power pack, a monitor, USB keyboard and USB mouse. If you have a VGA monitor, you'll need a VGA to HDMI converter as the Pi only has a HDMI slot. The keyboard, mouse and monitor are needed for the initial setup of the Pi. Once the Pi is configured and online, you can access it remotely via TeamViewer. 
4. A USB drive to hold backups of your wallet.dat file.

> NOTE: For the purposes of this guide I have named the USB volume BITGBackup as we will setup a shell script & CRON job to manage this automatically later on.

5. Heat sinks for the Pi to keep its chips as cool as possible - we're mining cryptos remember ;)

![Raspberry Pi Masternode Kit](../assets/pi3-kit.png?raw=true)

6. A Linux VPS - in this guide we will use a $5 Vultr Ubuntu 16.04 Linux VPS server. Don't have one yet? No worries - you can get your own Vultr VPS server here (https://www.vultr.com/?ref=7352503)

7. At least 2501 BITG (2500 collateral for the masternode, the balance to cover any transaction fees). 

You can buy this on any of the following exchanges:

Cryptopia: https://goo.gl/J64np1

Coinexchange: https://goo.gl/LS2WV2

CryptoBridge: https://goo.gl/7nZGdk


## SETUP YOUR PI ##

First we will setup & configure the Pi so that it is properly secure and can run backups smoothly. Remember that this is where our coins will be kept, so making sure this is secure and properly set to backup regularly is critical.

> NOTE: For ease of use, I recommend using Noobs to install Rasbian. For more info on how to get Noobs onto the SD card, and do the Raspbian install, follow the steps here: https://www.raspberrypi.org/learning/software-guide/

Do the following to get the Pi online.

1. Connect your Pi to the monitor, USB keyboard and USB mouse.
2. Insert the SD card with Noobs & power up the Pi.
3. Follow the on screen prompts to install Raspbian using Noobs
4. Connect your PI to the internet

Once Raspbian has been installed and you have setup the wifi connection to the internet, you must then do some basic security setups on the Pi. All of this step will be in done using the terminal:

> NOTE: If this is on a default install, you are currently working as the default user Pi

1. First we will make sure we are working as root. Type the following and press enter

``sudo su``

Enter your root password you chose during the setup process. 

You should now see the terminal entry line ends with a ``#``. This shows you are now working as root.

![Working as Root](../assets/pisecurity1.png?raw=true)

2. Now we will change the default Pi user password (don't delete the Pi user – you will need it later on to install Teamviewer)

In your terminal type

``sudo passwd``

Enter the current password when asked.
Enter the new password when asked.
Confirm the new password when asked.

![Password changed](../assets/pisecurity2.png?raw=true)

3. Now create a new superuser and password (If you are new to linux I recommend creating superuser 'bitg' to make this tutorial easier to follow and to allow the backup script to run without you having to make any edits). 

At the prompt type

``sudo adduser bitg``

Supply a password for this user when prompted. When asked for the details such as Name etc, just press enter until you get to the line asking you if the information is correct, at which point you just type `y` and press enter.

![User bitg added](../assets/pisecurity3.png?raw=true)

Now we must upgrade the bitg user to a superuser. At the prompt type

``sudo adduser bitg sudo `` 

User bitg will now be added to the superuser group.

![user upgraded to superuser](../assets/pisecurity4.png?raw=true)

4. Force sudo to require a password

At the prompt type

``sudo nano /etc/sudoers.d/010_pi-nopasswd``

![open nano](../assets/pisecurity5.png?raw=true)

Now change the pi entry (or whichever usernames have superuser rights) to:

``pi ALL=(ALL) PASSWD: ALL``

eg.: ``bitg ALL=(ALL) PASSWD: ALL``

Press ``CTRL + X`` to exit.

Press ``y`` to save the file before closing it.

![force sudo password](../assets/pisecurity6.png?raw=true)

5. Update Raspbian to the latest version 

First, to update your package list type

``sudo apt-get update`` and press enter

![get filesystem updates](../assets/pisecurity7.png?raw=true)

Next to rrun the upgrades, type 

``sudo apt-get dist-upgrade`` and press enter

![start upgrade](../assets/pisecurity8.png?raw=true)

You will be shown a list of new installs and upgrades - type `y` annd press enter to complete the upgrade process.

![confirm update](../assets/pisecurity9.png?raw=true)

Now remove unnecessary files after the upgrade to maximise space on the SD card. 

At the prompt enter each line, and press enter at the end of each line

```
sudo apt-get clean
sudo apt-get autoremove --purge
```

![remove unnecessary files](../assets/pisecurity10.png?raw=true)

You can save this page as a reference https://www.raspberrypi.org/documentation/raspbian/updating.md

6. Install ufw firewall and enable it by typing each of the following in succession (press enter after each line),

```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw logging on
sudo ufw enable
```

![install and configure firewall](../assets/pisecurity11.png?raw=true)

Now type 

``sudo ufw status``

The firewall should say that status is active.

![check firewall status](../assets/pisecurity12.png?raw=true)

7. Now we will install fail2ban. What this app does is ban people that keep entering the wrong password when trying to login via ssh, i.e brute force attacks. Remember to press enter after each line

``sudo apt-get install fail2ban`` and enter `y` when prompted

![install fail2ban](../assets/pisecurity13.png?raw=true)

Now type the following

```
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```
![start fail2ban](../assets/pisecurity14.png?raw=true)

After you have performed the above steps, go to Shutdown > Reboot

![Reboot](../assets/pisecurity15.png?raw=true)

**A Note on SSH**

SSH is typically used for remote access (we'll use it a little later to access our VPS from our Pi). However you need a static IP in order to use it if you want to access the Pi via SSH.

My Pi's run on my home network behind my router and so don't have a static external IP. For remote access to the Pi's I just run Teamviewer for Rasbian (instructions on how to do this at the end of the guide).

If you want to activate SSH on your Pi so that you can access it via SSH, you can look at the instructions on this page - https://www.raspberrypi.org/documentation/configuration/security.md. 


## SETUP THE CONTROLLER WALLET ##

After the Pi has rebooted, login as the new superuser you created (if you followed instructions, this will be user 'bitg' and you will be prompted for a password to login)

Now its time to setup the controller wallet. This is the wallet that will hold the collateral for the masternode.

Before we do so, we will need to download the correct wallet version for the Pi, as well as the BITG Bootstrap files to speed up the wallet synchronisation process. 

First lets create a folder for all the Bitcoin Green files we will need to download.

1. Open File Manager. 
2. Click File > Create Folder
3. Name the folder 'Bitcoingreen Files'
4. Click OK

![create folder Bitcoin Green](../assets/wallet1.png?raw=true)

Check that you now have a folder called 'Bitcoingreen Files' under home/bitg (bitg is the superuser you created earlier)

1. Open your browser on the Pi, and download the Pi BITG wallet from Github - https://github.com/bitcoingreen/bitcoingreen/releases/download/1.1.0/bitcoingreen-1.1.0-arm-linux-gnueabihf.tar.gz - save this file to the Bitcoingreen Files folder you just created.
2. Next point your browser to https://www.amazon.de/clouddrive/share/FYC5wP8e282To2XxUkzxqzUNjzHQB8zrMIsGmT2KJXT to download the latest BITG Bootstrap files (this link is also available in the Links section of the Bitcoin Green Discord Channel - if you haven't already joined, you can do so here https://discord.gg/g3CFth)
3. Save this to the Bitcoingreen Files folder as well.

![download wallet and bootstrap files](../assets/wallet2.png?raw=true)

4. Extract the wallet zip file (right click on it, then select Extract Here). This should create a folder named bitcoingreen-1.1.0
5. Open this folder, and then open the 'bin' folder. Inside the bin folder you will see 6 files. 
6. Double click on the file called bitcoingreen-qt to start the wallet install. 

![start the wallet](../assets/wallet3.png?raw=true)

7. Press the 'Execute' button when prompted. 

![press execute](../assets/wallet4.png?raw=true)

8. When prompted, select 'Use default data directory' (the path shown in grey should be /home/bitg/.bitcoingreen) – this is important for the backup script to work without having to edit it

![choose default data directory](../assets/wallet5.png?raw=true)

9. Click OK - the wallet will open, and you will see a message that says it is out of synch. 

![wallet out of synch](../assets/wallet6.png?raw=true)

10. Without waiting for the wallet to synchronise, close the wallet down as we will now use the bootstrap to fast track the synchronisation process.

To do this, we need to access the bootstrap files we downloaded earlier.

1. First extract the bootstrap files we downloaded earlier (right click on the bootstrap zip file and select Extract Here)

![extract bootstrap](../assets/wallet7.png?raw=true)

2. Double click the folder BITG_Bootstrap
3. Click anywhere inside the folder, and press CTRL+A to select all files and folders, then CTRL+C to copy them

![copy bootstrap](../assets/wallet8.png?raw=true)

4. Navigate to the /home/bitg/ folder. Click anywhere in the folder and press ``CTRL + H`` This will reveal the hidden folders. 

> NOTE: Any folder with a '.' in front of it is hidden by default in Linux

5. Open the (now unhidden) .bitcoingreen folder, click anywhere inside it and then press CTRL+V to paste them into this folder 

> NOTE: this is the default data directory for the wallet that we chose when we installed it. It is also where your wallet.dat (ie. your COINS!) file lives.

6. You will see a warning popup that says you are about to overwrite files. Make sure you check the 'Apply this option to all existing files' and then click the Overwrite button

![paste bootstrap files into .bitcoin directory](../assets/wallet9.png?raw=true)

7. Go back to the home>bitg>Bitcoin Green>bitcoingreen-1.0.0>bin folder, and double click the bitcoingreen-qt file again to launch the wallet. Remember to press the 'Execute' button when prompted.

8. The wallet will now launch and complete the synchronisation process.


## SECURE THE CONTROLLER WALLET ##

Once the wallet has synchronised with the blockchain, we need to do a little tightening up on securing the wallet itself.

First we will encrypt the wallet.

1. Click Settings > Encrypt Wallet
2. Enter an 8 word or more secure passphrase and click OK

![encrypt wallet](../assets/wallet10.png?raw=true)

> **THIS IS VERY IMPORTANT!!!**
> **Make sure that you save this passphrase somewhere safe - without it you will never be able to get your coins back if you need to restore the wallet at some point in the future. I personally have a printout that I keep in my safe, as well as a copy on a usb pendrive that I can access whenever I need it.**

Next we need to configure the wallet. 

Go to Settings > Options

1. Under the Main Tab, make sure that the Start Bitcoin Green on system login is ticked

![start bitcoin green on startup](../assets/wallet11.png?raw=true)

2. Click the Wallet Tab
3. Tick Enable coin control features
4. Tick Show Masternodes tab

![enable coin control and masternodes](../assets/wallet12.png?raw=true)

5. Click OK

Next we need to lock the wallet

1. Go to Settings > Lock Wallet
3. Click Ok

Finally we will unlock it for staking only

1. Go to Settings > Unlock Wallet
2. Enter your passphrase
3. Tick Unlock for staking only
3. Click Ok

![unlock for staking](../assets/wallet13.png?raw=true)

Now backup the wallet.dat file.

> NOTE: Make sure that your USB drive called BITGBackup is inserted into the Pi.

To do a manual backup of the wallet.dat file, do the following:

1. Go to File > Backup Wallet
2. Select a destination (recommend the USB drive)
3. Name the file
4. Click OK

![manual backup](../assets/wallet14.png?raw=true)

To keep the Pi running smoothly however, we want to automate this process, and ensure that the backup file is kept on the USB drive in case the SD card ever fails. 

To do this we must create a shell script that can save a compressed and timestamped copy of our wallet.dat file twice a day.

1. Open your terminal on the Pi

2. Type

``sudo nano bitgbackup.sh``

3. enter your password and press enter

![auto backup-step1](../assets/backup1.png?raw=true)

4. Nano will now open

5. Copy and paste (right click > paste) the following into nano:

```
#!/bin/bash`
####################################
# Backup BITG Wallet.dat file to USB
####################################
# What to backup
#backup_files="home/YOUR_USER/.bitcoingreen/wallet.dat"
backup_files="/home/bitg/.bitcoingreen/wallet.dat"
# Where to backup to
# NOTE: make sure to name the USB drive 'BITGBackup'
# path to the USB Drive for backups will be dest="/media/YOUR_USER/BITGBackup"
dest="/media/bitg/BITGBACKUP"
# Create archive filename.
day=$(date +"%Y%m%d%H%M")
archive_file="BITG-WalletBackup-$day.tgz"
# Print start status message.
echo "Backing up $backup_files to $dest/$archive_file"
date
echo
# Backup the files using tar.
tar czf $dest/$archive_file $backup_files
# Print end status message.
echo
echo "BITG wallet.dat file backup finished"
date
# Long listing of files in $dest to check file sizes.
ls -lh $dest
```
![auto backup-step2](../assets/backup2.png?raw=true)

6. Check the line:

``backup_files="/home/bitg/.bitcoingreen/wallet.dat" ``

to make sure it is pointing to wallet.dat on your Pi.

![auto backup-step3](../assets/backup3.png?raw=true)

> NOTE: if you had created the superuser bitg earlier, then this line should be correct already and you should not have to edit it.

7. Check the line:

``dest="/media/bitg/BITGBackup" ``

to see that the backup file is being writtten to the USB. 

![auto backup-step4](../assets/backup4.png?raw=true)

> NOTE: If you are following this guide, and named your USB volume BITGBackup, then this line will also be correct. 

8. Press ``CTRL + X``
9. Press ``Y`` and then press Enter
10. type

``sudo chmod u+x bitgbackup.sh``

11. Press enter

![auto backup-step5](../assets/backup5.png?raw=true)

12. now type

``sudo ./bitgbackup.sh``

13. Press enter

You should see the script push out some output as it backs up to the USB drive. This should also show you the directory listing of the USB so you can see if your backup file exists.

![auto backup-step6](../assets/backup6.png?raw=true)

Open the USB drive in file explorer and confirm that the script has generated the backup file by opening the backup zip file to see that the wallet .dat file has been saved. 

You should see a file that is called something like BITG-WalletBackup-201805081029.tgz – the number at the end is the date and time of the backup so that you can keep track of them.

To automate this script, we must now setup a CRON job to manage it.

1. Open the terminal and type the following

``sudo crontab -e``

2. Press enter

![auto backup-step7](../assets/backup7.png?raw=true)

3. If its the first time you are doing this, you must now choose a text editor - I suggest choose 2 (nano) and press enter
4. Navigate to the bottom of the file using your arrow keys
5. ype the following, each on a new line
```
59 11 * * * /home/bitg/bitgbackup.sh
59 23 * * * /home/bitg/bitgbackup.sh
```
6. Press ``CTRL + X``
7. Type ``Y`` and press enter

![auto backup-step8](../assets/backup8.png?raw=true)

The above will now run the backup shell script every day at 11:59 and again at 23:59 and will save a zipped copy of the wallet.dat file to the USB drive. Make sure to check on this regularly and make additional backups off this USB drive.


## SETUP THE MASTERNODE - THE CONTROLLER WALLET ##

Now its time to start setting up the Masternode.

First we must create the masternode wallet address(es) to send your masternode coins to.

1. Go to file > receiving addresses
2. Click +New button
3. In Label field type the name of your masternode (eg. MN1 or Masternode1, MN2 or Masternode2, etc.)
4. Click OK
5. Repeat this for each MN you want to add.
6. You should now see your new masternode address(es) in the receiving addresses list.
7. Send exactly 2500 coins to (each of) your masternode address(es). The recommended method is to first transfer all the coins from the exchange where you purchased them to your wallet, and then move exactly 2500 coins to each of the MN address(es).

Before we can continue, we must create the VPS so that we can have the VPS IP number as we will need this to create the masternode configuration file on the local wallet.

To create the VPS, ![open your account on Vultr](https://www.vultr.com/?ref=7352503) and do the following steps

1. Click the Servers tab on the left
2. Click the blue + sign on the right to deploy a new server

![](../assets/vultr1.png?raw=true)

3. Select a location (anywhere is fine)

![](../assets/vultr2.png?raw=true)

4. Choose a server type – this must be Ubuntu 16.04

![](../assets/vultr3.png?raw=true)

5. Choose a server size – we recommend the $5/month server

![](../assets/vultr4.png?raw=true)

6. Scroll down to step 7 and give the server a hostname (like BITG or Bitcoin Green) and label (bitg masternode server)

![](../assets/vultr5.png?raw=true)

7. Check that quantity is 1 and then click the Deploy Now button

![](../assets/vultr6.png?raw=true)

Vultr will now create your VPS server. Give it a few minutes to do so, and then click the Servers tab on the left menu again to see your new server in the list.

![](../assets/vultrserverlist.png?raw=true)

Click on the server to see the details screen where you will see the IP number assigned to this server. Make a note of this as we will need it for the next step.

![](../assets/vultripaddress.png?raw=true)

Now we need to setup the masternode configuration file on the Pi wallet. Before we do so we need the following set of info. 

> NOTE: you need a seperate dataset for EACH MN you setup, and you MUST have already transferred your MN collateral to the MN address so that we have a transaction hash value.

For each MN we need:

1. masternode private key 
2. transaction hash
3. output index 
4. your VPS servers IP number and the port for your masternode (its recommended that you start with port 9333 (the bitg default port) and work upwards from that for each seperate node, ie 9334, 9335 etc.

Go to the bitg wallet on the Pi.

1. To generate the masternode private key, go to Tools > debug console and type 

``masternode genkey ``

2. Press enter
3. Copy and paste the result to a text editor

> NOTE: You must repeat this for each masternode you want to setup.

4. To generate the txid and output index, type 

``masternode outputs``

5. Press enter

This will generate a set of txid & output index for each MN address that you sent coins to.

6. Copy and paste the result to a text editor

7. Now open the masternode configuration file, and for each masternode you want to setup, enter all this information on a new line as follows

MasternodeAlias(eg. MN1) ipnumber:port(eg.125.254.124.251:9333) MasternodePrivateKey(eg.93HaYBVUCYjEMeeH1Y4sBGLALQZE1Yc1K64xiqgX37tGBDQL8Xg) TransactionID(eg.2bcd3c84c84f87eaa86e4e56834c92927a07f9e18718810b92e0d0324456a67c) Index(eg.1)

eg.
```
MN1 234.178.115.231:9333 93HaYBVUCYjEMeeH1Y4sBGLALQZE1Yc1K64xiqgX37tGBDQL8Xg 2bcd3c84c84f87eaa86e4e56834c92927a07f9e18718810b92e0d0324456a67c 1
MN2 234.178.115.231:9334 76HGT56432aashjhYT65435jGHHJGkjhhi887JHKkh908KJkjhh 98dsd664463008097fdsfsdf90sssa198df77987896fs6676686sdd85675477j 1
```

> NOTE: keep all the info for each MN on its own line

8.Save and close the config file.

9. Close the wallet.

10. Restart the wallet.

OK so now we need to move across to setting up the VPS on Vultr.

## SETUP THE MASTERNODE - ON THE VPS ##

> NOTE: This section is based on the excellent cold wallet setup guide & script provided by XeZZoR which you can get here https://goo.gl/S7fKdP  

First we need to connect to the VPS server you created at Vultr.

1. On your Pi, open the console and type 

``sudo su``

2. Enter your password

You should now be root user (console should show a '#' instead of a '$')

3. Now type 

`` ssh your_server_ip`` (eg. ssh 192.168.0.1) 

and press enter

4. Enter the password for the server (you can copy this from the Vultr dashboard – then use your mouse to right click and paste it into the terminal, and then press enter)

![](../assets/vultrpword.png?raw=true)

5. Once you are logged in to the server via SSH, type/copy & paste the following commands into your terminal -  this will fetch and run XeZZoR's automatic server installation & setup script.

``wget https://raw.githubusercontent.com/XeZZoR/scripts/master/BITG/setup.sh``

press enter then at the next prompt type

``chmod 755 setup.sh``

press enter then at the next prompt type

``./setup.sh``

press enter

This will start the BITG masternode setup script. 

>NOTE: If its the first time you are using it, you will need to install all needed dependencies – in which case type `y` and press enter.
> If you have already installed these, type `n` and press enter.

![](../assets/xezz1.png?raw=true)

> NOTE: Installing the dependencies can take a while. If you are prompted to proceed with operation type `y` and press enter.

![](../assets/xezz2.png?raw=true)

The script will get to a point where you must start configuring the masternodes. 

![](../assets/xezz3.png?raw=true)

This is where you need the masternode alias, VPS server IP and port, & masternode private key information that we used earlier.

1. Enter the number of masternodes you want to setup, and then enter the relevant info for each node when prompted.

![](../assets/xezz4.png?raw=true)

2. For the RPC ports, you can enter any valid free ports – the script recommends you start at 17100 and work upwards from there for each masternode.

![](../assets/xezz5.png?raw=true)

Once the script has finished running, you will see a message that says 

``Bitcoin Green server starting.``

If at any time you made an error just enter CTRL + C and then you can restart the script by typing

``./setup.sh``

During setup, each masternode is given an alias and has its own control script in the ~/bin folder. To check on the status of each masternode, type

``cd bin``

then type 

``ls``

You will see the files available in the /bin folder. 

You want to run one that looks like ``bitcoingreen-cli_mn1.sh``. If you see this file, then the script has run successfully.

3. Exit the terminal on your Pi.

Now go to your Vultr dashboard, and restart the VPS. This is required after installing all the server dependencies during setup.

![](../assets/vultrserverrestart.png?raw=true)

To do so, click the Servers tab on the left of your Vultr dashboard. Then click on the 3 dots ... to the right of the text that says 'Running' and then click Server Restart.

Onnce the VPS is back up and running, you need to SSH into the VPS again from the terminal on your Pi.

1. Once you are back in the VPS, type 

``sudo su``

then enter your password

2. Now that you are the root user, type

``cd bin``

3. then type

``bitcoingreend_mn1.sh -reindex``

This will restart the masternode server on the VPS.

4. You can now check on the masternode synchronisation status by typing

``bitcoingreen-cli_mn1.sh mnsync status``

![](../assets/xezz6.png?raw=true)

5. Run this until you see it say ``“IsBlockChainSynced”: true`` then type

``bitcoingreen-cli_mn1.sh masternode status``

6. Run this until you see ``“message”:Masternode successfully started”``.

Now we must go back to the Pi to complete the final steps of switching on your masternode.


## START THE MASTERNODE - ON YOUR PI ##

1. On the Pi, open the wallet and let it synchronise
2. Unlock the wallet by going to Settings > Unlock wallet – enter your passphrase

Make sure that Unlock for staking only is UNTICKED!

![Unlock wallet](../assets/wallet16.png?raw=true)

2. Go to the Masternodes tab
3. You should see your Masternode(s) in the list
4. Right click on it and click Start Alias
5. You should see Protocol field show 70912 and Status field should show ENABLED

![Start Masternode Alias](../assets/wallet15.png?raw=true)

6. Wait ~48 hrs for the node to start earning rewards

Now the final step is to lock the wallet and then unlock the wallet for staking only

1. Go to Settings > Lock wallet
2. Go to Settings > Unlock wallet
3. Enter your passphrase and tick “Unlock for staking only”

![Unlock wallet](../assets/wallet13.png?raw=true)

4. Click OK

# Congratulations! #

You are now setup to earn masternode rewards and staking rewards to your Raspberry Pi :)

## Additional Steps ##
### HIGHLY Recommended but not required to earn rewards :) ###

**Backup the SD Card**

A good thing to do now is to make a regular disk image backup of the SD card, so that if it fails, you can easily restore it.

You can do so by using the raspbiBackup script. Get instructions on how to use it here: https://www.linux-tips-and-tricks.de/en/quickstart-rbk

> NOTE: I am currently experimenting with this. I will update this guide with the automation script once I have it figured out.

**Setup Remote Access to the Pi with TeamViewer**

Setup Remote access so that you can manage your Pi from anywhere using Teamviewer. I do this as I can then control my cold wallet from my phone using the Teamviewer app.

It also means you can easily manage multiple Pi's as your Pi mining setup grows :)

Follow this tutorial to set TeamViewer up on Raspbian: https://m.youtube.com/watch?v=vr3Gf8vnKAg

Download TeamViewer for Raspbian from this link: https://www.teamviewer.com/iotcontest/

Thats IT!

## Donations ##

I hope you found this guide useful. If you did, any donations will be much appreciated :)

> Donate Bitcoin Green (BITG) **GXtS1QCPsMANpRDQa5xFwogfbxT42dy5uV**

> Donate Bitcoin (BTC) **1AxzU81tw8rBL9vRyGdhWL4s4C8BusffP7**
