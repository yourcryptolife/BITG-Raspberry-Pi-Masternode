## Raspberry PI Bitcoin Green Cold Wallet Masternode Setup Guide ##

This is a guide for setting up a Bitcoin Green Cold VPS Masternode using a Raspberry Pi3 to host the cold wallet & a Vultr Ubuntu 16.04 VPS to act as the MN server.

**Use at your own risk!**

You can see the current performance of a BITG Masternode here: https://masternodes.online/currencies/BITG/

This setup has 2 main benefits with regards to running the node:

1. Coin safety. Because it stores your coins locally (on the Pi), it protects your masternode investment from being stolen by hackers should they manage to hack the VPS which is the masternodes public facing IP address.

> NOTE: this also assumes that you ensure the Pi has at least some basic protection by following the instructions in this guide. 

2. In addition to earning masternode rewards, this setup allows you to earn staking rewards by leaving the Pi online 24/7, and keeping the wallet on the Pi open all the time - encrypted and locked, but open for staking.

This allows you to maximise the return of your masternode as you will earn both masternode rewards as well as staking rewards.

**Before we get started, you will need the following items:**

1. Raspberry Pi 3 to host your cold wallet and coins
2. At least an 8GB micro SD Card with Noobs installed
3. A Pi power pack, a monitor, USB keyboard and USB mouse. If you have a VGA monitor, you'll need a VGA to HDMI converter as the Pi only has a HDMI slot.
4. A USB drive to hold backups of your wallet.dat & a backup image of the Pi's SD card. I recommend at least a 16GB USB drive for this. 
> NOTE: For the purposes of this guide please name the USB volume BITGBackup as we will setup a small shell script & CRON job to manage this automatically later on.
5. A Linux VPS - in this guide we will use a $5 Vultr Ubuntu Linux server - ![get your Vultr server here](https://www.vultr.com/?ref=7352503) https://goo.gl/MYCGgv
6. At least 2501 BITG (2500 collateral for the masternode, the balance to cover any transaction fees). 

You can buy this on any of the following exchanges:

Cryptopia: https://goo.gl/J64np1

Coinexchange: https://goo.gl/LS2WV2

CryptoBridge: https://goo.gl/7nZGdk

OK, so let's get started. 

First we will setup & configure the Pi.

## ON YOUR PI ##

1. Connect your Pi to the monitor, USB keyboard and USB mouse.
2. Insert the SD card with Noobs & power up the Pi.
3. Install Raspbian using Noobs
4. Connect your PI to the internet
For more info on how to get Noobs onto the SD card, and do the Raspbian install, go to https://www.raspberrypi.org/learning/software-guide/

Once Raspbian has been installed and you have setup the wifi connection to the internet, you must then secure the Pi.

Follow the instructions on this page - https://www.raspberrypi.org/documentation/configuration/security.md, **paying specific attention to:**

1. change the default PI user password (don't delete the Pi user – we will need it later on – its a Pi thing ...)
2. create a new superuser and password (NOTE: I recommend creating superuser 'bitg' to make this tutorial easier to follow and to allow the backup script to run without you having to make any edits)
3. make sudo require a password
4. update Raspbian to the latest version (follow the instructions here https://www.raspberrypi.org/documentation/raspbian/updating.md)
5. install ufw firewall and enable it

After you have performed the above steps, go to Shutdown > Reboot

Now login as the new superuser you created (if you followed instructions, this will be user 'bitg' and you will be prompted for a password to login)

Now its time to setup the cold wallet. Before we do so, we will need to download the correct wallet version for the Pi, as well as the BITG Bootstrap files to speed up the wallet synchronisation process. 

First lets create a folder for all the Bitcoin Green files we will need to download. 

1. Open File Manager. 
2. Click File > Create Folder
3. Name the folder 'Bitcoingreen Files'
4. Click OK

Check that you now have a folder called 'Bitcoingreen Files' under home/bitg (bitg is the superuser you created earlier)

1. Open your browser on the Pi, and download the Pi BITG wallet from Github - https://github.com/bitcoingreen/bitcoingreen/releases/download/1.1.0/bitcoingreen-1.1.0-arm-linux-gnueabihf.tar.gz - save this file to the Bitcoingreen Files folder you just created.
2. Next point your browser to https://www.amazon.de/clouddrive/share/FYC5wP8e282To2XxUkzxqzUNjzHQB8zrMIsGmT2KJXT to download the latest BITG Bootstrap files (this link is also available in the Links section of the Bitcoin Green Discord Channel - if you haven't already joined, you can do so here https://discord.gg/g3CFth)
3. Save this to the Bitcoingreen Files folder as well.
4. Extract the wallet zip file (right click on it, then select Extract Here). This should create a folder named bitcoingreen-1.1.0
5. Open this folder, and then open the 'bin' folder. Inside the bin folder you will see 6 files. 
6. Double click on the file called bitcoingreen-qt to start the wallet install. Press the 'Execute' button when prompted. 
7. When prompted, select 'Use default data directory' (the path shown in grey should be /home/bitg/.bitcoingreen) – this is important for the backup script to work
8. Click OK - the wallet will open, and you will see a message that says it is out of synch. 
9. Without waiting for the wallet to synchronise, close the wallet down as we will now use the bootstrap to fast track the synchronisation process.

To do this, we need to access the bootstrap files we downloaded earlier.

1. first extract the bootstrap files we downloaded earlier (right click on the bootstrap zip file and select Extract Here)
2. double click the folder BITG_Bootstrap
3. Click anywhere inside the folder, and press CTRL+A to select all files and folders, then CTRL+C to copy them
4. Navigate to the /home/bitg/.bitcoingreen folder and press CTRL+V to paste them into this folder (NOTE the . before bitcongreen – this is the default data directory for the wallet)
5. You will see a warning popup that says you are about to overwrite files. Make sure you check the 'Apply this option to all existing files' and then click the Overwrite button
6. Go back to the home>bitg>Bitcoin Green>bitcoingreen-1.0.0>bin folder, and double click the bitcoingreen-qt file again to launch the wallet. Remember to press the 'Execute' button when prompted.
7. The wallet will now launch and complete the synchronisation process.

Once the wallet has synchronised with the blockchain, we need to do a little tightening up on securing the wallet itself.

First we will encrypt the wallet.

1. Click Settings > Encrypt Wallet
2. Enter an 8 word or more secure passphrase and click OK
> **VERY NB!!! Make sure that you save this passphrase somewhere safe - without it you will never be able to get your coins back if you need to restore the wallet at some point in the future. I personally have a printout that I keep in my safe, as well as a copy on a usb pendrive that I can access whenever I need it.

Next we need to configure the wallet. 

Go to Settings > Options

1. Under the Main Tab, make sure that the Start Bitcoin Green on system login is ticked
2. Click the Wallet Tab
3. Tick Enable coin control features
4. Tick Show Masternodes tab
5. Click OK

Next we need to lock the wallet

1. Go on Settings > Lock Wallet
2. Enter your passphrase
3. Click Ok

Now backup the wallet.dat file.

> NOTE: Make sure that your USB drive called BITGBackup is inserted into the Pi.

To do a manual backup of the wallet.dat file, do the following:

1. Go to File > Backup Wallet
2. Select a destination (recommend the USB drive)
3. Name the file
4. Click OK

To keep the Pi running smoothly however, we want to automate this process, and ensure that the backup file is kept on the USB drive in case the SD card ever fails (which is common on the Pi). To do this we must create a shell script that can save a compressed and timestamped copy of our wallet.dat file twice a day.

1. Open your terminal on the Pi
2. type

> sudo nano bitgbackup.sh

3. enter your password and press enter
4. nano will now open.
5. Copy and paste the following (everything BETWEEN –-start--- and –-end---) into nano:

---start---

```
#!/bin/bash`
####################################
# Backup BITG Wallet.dat file to USB
####################################
# What to backup
#backup_files="home/YOUR_USER/.bitcoingreen/wallet.dat"
backup_files="/home/bitgpi/.bitcoingreen/wallet.dat"
# Where to backup to
# NOTE: make sure to name the USB drive 'BITGBackup'
# path to the USB Drive for backups will be dest="/media/YOUR_USER/BITGBackup"
dest="/media/bitgpi/BITGBACKUP"
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

--end--

6. Check the line:

> backup_files="/home/bitg/.bitcoingreen/wallet.dat" 

to make sure it is pointing to wallet.dat on your Pi.

> NOTE: if you had created the superuser bitg earlier, then this line should be correct already and you should not have to edit it.

7. Check the line:

> dest="/media/bitg/BITGBackup"

to see that the backup file is being writtten to the USB. 

> NOTE: If you are following this guide, and named your USB volume BITGBackup, then this line will also be correct. 

8. Press CTRL + X
9. Press Y and then press Enter
10. type

> sudo chmod u+x bitgbackup.sh

11. Press enter.
12. now type

> sudo ./bitgbackup.sh

13. Press enter.

You should see the script push out some output as it backs up to the USB drive. This should show you the directory listing of the USB so you can see if your backup file exists.

Open the USB drive in file explorer and confirm that the script has generated the backup file by opening the backup zip file to see that the wallet .dat file has been saved. 

You should see a file that is called something like BITG-WalletBackup-201805081029.tgz – the number at the end is the date and time of the backup so that you can keep track of them.

To automate this script, we must now setup a CRON job to manage it.

1. Open the terminal and type the following

> sudo crontab -e

2. press enter
3. if its the first time you are doing this, you must now choose a text editor - I suggest choose 2 (nano) and press enter
4. navigate to the bottom of the file using your arrow keys
5. type the following, each on a new line
```
59 11 * * * /home/bitg/bitgbackup.sh
59 23 * * * /home/bitg/bitgbackup.sh
```
6. press CTRL + X
7. type Y and press enter

The above will now run the backup shell script every day at 11:59 and again at 23:59 and will save a zipped copy of the wallet.dat file to the USB drive. Make sure to check on this regularly and make additional backups off this USB drive.

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

![go to Vultr](../assets/vultr1.png?raw=true)

3. Select a location (anywhere is fine)

![go to Vultr](../assets/vultr2.png?raw=true)

4. Choose a server type – this must be Ubuntu 16.04

![go to Vultr](../assets/vultr3.png?raw=true)

5. Choose a server size – we recommend the $5/month server

![go to Vultr](../assets/vultr4.png?raw=true)

6. Scroll down to step 7 and give the server a hostname (like BITG or Bitcoin Green) and label (bitg masternode server)

![go to Vultr](../assets/vultr5.png?raw=true)

7. Check that quantity is 1 and then click the Deploy Now button

![go to Vultr](../assets/vultr6.png?raw=true)

Vultr will now create your VPS server. Give it a few minutes to do so, and then click the Servers tab on the left menu again to see your new server in the list.

![go to Vultr](../assets/vultrserverlist.png?raw=true)

Click on the server to see the details screen where you will see the IP number assigned to this server. Make a note of this as we will need it for the next step.

![go to Vultr](../assets/vultripaddress.png?raw=true)

Now we need to setup the masternode configuration file on the Pi wallet. Before we do so we need the following set of info. 

> NOTE: you need a seperate dataset for EACH MN you setup, and you MUST have already transferred your MN collateral to the MN address so that we have a transaction hash value.

For each MN we need:

1. masternode private key 
2. transaction hash
3. output index 
4. your VPS servers IP number and the port for your masternode (its recommended that you start with port 9333 (the bitg default port) and work upwards from that for each seperate node, ie 9334, 9335 etc.

Go to the bitg wallet on the Pi.

1. To generate the masternode private key, go to Tools > debug console and type 

> masternode genkey 

2. Press enter
3. copy and paste the result to a text editor

> NOTE: You must repeat this for each masternode you want to setup.

To generate the txid and output index, type 

> masternode outputs

This will generate a set of txid & output index for each MN address that you sent coins to.

Copy and paste the result to a text editor

Now open the masternode configuration file, and for each masternode you want to setup, enter all this information on a new line as follows

MasternodeAlias(eg. MN1) ipnumber:port(eg.125.254.124.251:9333) MasternodePrivateKey(eg.93HaYBVUCYjEMeeH1Y4sBGLALQZE1Yc1K64xiqgX37tGBDQL8Xg) TransactionID(eg.2bcd3c84c84f87eaa86e4e56834c92927a07f9e18718810b92e0d0324456a67c) Index(eg.1)

eg.
```
MN1 234.178.115.231:9333 93HaYBVUCYjEMeeH1Y4sBGLALQZE1Yc1K64xiqgX37tGBDQL8Xg 2bcd3c84c84f87eaa86e4e56834c92927a07f9e18718810b92e0d0324456a67c 1
MN2 234.178.115.231:9334 76HGT56432aashjhYT65435jGHHJGkjhhi887JHKkh908KJkjhh 98dsd664463008097fdsfsdf90sssa198df77987896fs6676686sdd85675477j 1
```

> NOTE: keep all the info for each MN on its own line

Save and close the config file.

Close the wallet.

Restart the wallet.

OK so now we need to move across to setting up the VPS on Vultr.

## ON THE VPS ##

> NOTE: This section is based on the excellent cold wallet setup guide & script provided by XeZZoR which you can get here https://goo.gl/S7fKdP  

First we need to connect to the VPS server you created at Vultr.

On your Pi, open the console

Type 

> sudo su

Enter your password

You should now be root user (console should show a '#' instead of a '$')

Now type 

>ssh your_server_ip (eg. ssh 192.168.0.1) 

and press enter

Enter the password for the server (you can copy this from the Vultr dashboard – then use your mouse to right click and paste it into the terminal, and then press enter)

![go to Vultr](../assets/vultrpwd.png?raw=true)

Once you are logged in to the server via SSH, type/copy & paste the following commands into your terminal -  this will fetch and run XeZZoR's automatic server installation & setup script.

> wget https://raw.githubusercontent.com/XeZZoR/scripts/master/BITG/setup.sh

press enter

> chmod 755 setup.sh

press enter 

> ./setup.sh

press enter

This will start the BITG masternode setup script. 

If its the first time you are using it, you will need to install all needed dependencies – in which case type `y` and press enter.

If you have already installed these, type `n` and press enter.

> NOTE: Installing the dependencies can take a while. If you are prompted to proceed with operation type `y` and press enter.

The script will get to a point where you must start configuring the masternodes. 

This is where you need the masternode alias, VPS server IP and port, & masternode private key information that we used earlier.

Enter the number of masternodes you want to setup, and then enter the relevant info for each node when prompted.

For the RPC ports, you can enter any valid free ports – the script recommends you start at 17100 and work upwards from there for each masternode.

Once the script has finished running, you will see a message that says 

`Bitcoin Green server starting.`

If at any time you made an error just enter CTRL + C and then you can restart the script by typing

> ./setup.sh

During setup, each masternode is given an alias and has its own control script in the ~/bin folder. To check on the status of each masternode, type

> cd bin

then type 

> ls

You will see the files available in the /bin folder. 

You want to run one that looks like `bitcoingreen-cli_mn1.sh`. If you see this file, then the script has run successfully.

Exit the terminal on your Pi.

Now go to your Vultr dashboard, and restart the VPS. This is required after installing all the server dependencies during setup.

To do so, click the Servers tab on the left of your Vultr dashboard. Then click on the 3 dots ... to the right of the text that sayys 'Running' and then click Server Restart.

Onnce the VPS is back up and running, you need to SSH into the VPS again from the terminal on your Pi.

Once you are back in the VPS, type 

> cd bin

then type

> bitcoingreend_mn1.sh -reindex

This will restart the masternode.

You can now check on the masternode synchronisation status by typing

> bitcoingreen-cli_mn1.sh mnsync status

Run this until you see it say `“IsBlockChainSynced”: true` then type

> bitcoingreen-cli_mn1.sh masternode status

Run this until you see `“message”:Masternode successfully started”`.

Now we must go back to the Pi to complete the final steps of switching on your masternode.

## ON YOUR PI ##

1. On the Pi, open the wallet and let it synchronise
2. Unlock the wallet by going to Settings > Unlock wallet – enter your passphrase
2. Go to the Masternodes tab
3. You should see your Masternode(s) in the list
4. Right click on it and click Start Alias
5. You should see Protocol field show 70912 and Status field should show ENABLED
6. Wait ~48 hrs for the node to start earning rewards

Now the final step is to lock the wallet and then unlock the wallet for staking only

1. Go to Settings > Lock wallet
2. Go to Settings > Unlock wallet
3. Enter your passphrase and tick “Unlock for staking only”
4. Click OK

# Congratulations! #

You are now setup to earn masternode rewards and staking rewards to your Raspberry Pi :)

## Additional Steps ##
### HIGHLY Recommended but not required to earn rewards :) ###

**Backup the SD Card**

A good thing to do now is to make a regular disk image backup of the SD card, so that if it fails, you can easily restore it.

You can use the excellent raspbiBackup script. Get instructions on how to use it here: https://www.linux-tips-and-tricks.de/en/quickstart-rbk

> NOTE: You can also automate this to regularly backup your SD card image using a CRON job to your USB drive.

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