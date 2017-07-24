Cloud Server Migration
23 JULY 2012 on linux, openstack, rackspace, cloud servers, migrating, rsync, system administration
Here is what this Tool / Article Does
This write up and script will migrate any Linux instance from one place to another. While there are always caveats in all things Technology, I have been successful in migrating instances that were locally created as well as on several Cloud Providers. While there are situations which will make migrations impossible, in most cases moving instances around to other similar instances should be a breeze. If easy is your thing go ahead and skip to here: [Migrate using RSYNC The Easy Way.

Table of Contents

The use cases
What I have tested
Cloning or Migrating a Cloud Server
Estimated time of Completion
Migrate using RSYNC The Easy Way
Walkthrough of Easy Migration Process
Doing the RSYNC Operation By Hand
RSYNC Prerequisites
Installing Screen
The Start of the Migration
Creating the Exclusions List
Setting the IP Variable
Performing the RSYNC Action
Performing the Second RSYNC Action
Restarting the Target Server
What needs to be done to move your Amazon AMI Linux to another Provider
Amazon AMI Specific Exclusion List
Post RSYNC Clean up of an AMI Instance
The use cases
Migrating from one provider to another
Cloning an instance
Upgrading Infrastructures
Setting up Geographic Redundancies
What I have tested
Rackspace Instances from XenClassic To XenServer
Rackspace Instances from XenServer to Open Cloud powered by OpenStack
Rackspace Instances from XenClassic To Open Cloud powered by OpenStack
Amazon EC2 to the Rackspace Cloud
Rackspace Cloud to Amazon EC2
KVM to Rackspace Cloud
Caveats 
There are two issues I have found while migrating instances around. However, the most important caveat was related to the instance type. You Must Have a Similar Instance to Migrate Too. Additionally to migrate a linux server from one place to another you must also have setup the instance to use a Single partition for installation. If you are using a multi-partition virtual instance, you will have to manually migrate the partitions accordingly. Which can be successfully accomplished by migrating the instance by hand. Other than the one caveat, of having Similar Instance, I have not had this process fail.

Testament that It Works 
I know that this method works for a variety of situations, I have even had this process complete successfully when migrating an instance that uses Amazon AMI Linux.

if you move an instance using Amazon AMI Linux to another hosting provider, there will be some things that will have to be done post migration to have a fully functional instance.

Amazon AMI Related Migration issues are covered at the bottom of the article.

Estimated time of Completion
The Estimated time to completion based on Gigabytes of Consumed Space Computations have been made using an average transfer rate of 
20 Megabytes a second.

  ---------------- --------------
  The Estima       ted time
  --------------   ------------
  Space Used       EST Time
  ============     ============
  10G              9 Minutes
  20G              17 Minutes
  40G              34 Minutes
  80G              68 Minutes
  160G             136 Minutes
  320G             272 Minutes
  620G             544 Minutes
  1200G            1088 Minutes
  ---------------- --------------
Cloning or Migrating a Cloud Server
Migrating your Cloud Servers from any platform can be done in many ways. In this document we will be covering, how to do this from a Live Legacy System In the Rackspace Cloud to a newly provisioned Open Cloud System still residing in the Rackspace Cloud. While this method was specifically tailored to Rackspace Systems, this method could be used when migrating any system for any place to anywhere. To make an overly simple statement, we will be moving data from one instance to another. This can be done by copying the BLOCK device, tar'ing up the important contents and sending it to the new system, or using RSYNC. In this document we will cover how to migrate your instance using RSYNC.

The process we will be discussing has an an application that I have written. This application will simplify the entire transfer process. However, for those of you that enjoy pain, we will be covering how this is all done by hand too.

To use the pre-written application to migrate you will only need to know three things to make this work.

Create a New Open Cloud Server with a similar OS as your Source 
Server
Record the IP Address of the New Open Cloud server
Record the ROOT password of the new Target server
To create new servers, you have two options within the Racksapce Cloud Environment, The Control Panel or the API. I use the API as I find it faster and easier than to GUI interface but both are perfectly valid. If you are on a Linux / Unix based system you can access the cloud servers API simply by using the Python-Nova Client.

You can read more about the python-nova client here

For those of you that want to know how to Administer your servers from the CLI without a wrapper, I would advise you have a look at the API documentation found on the Rackspace API docs page.

Process Overview: 
When you create a new Open Cloud Instance and have the intention of migrating your Legacy Cloud server to the new Open Cloud infrastructure you will need to deploy a similar Distribution of the Source Server. For example, if you are Migrating from Ubuntu 10.04 in Legacy you will need to Deploy a new Ubuntu 10.04 Open Cloud Server. This will help ensure that the Migration is successful. You should know that it is possible that there may not be an exact match for your instance in the New Open Cloud. If you are in a situation where there is no exact match you will have to choose the instance type that is the closest match. For example, if I were running a CentOS 5.5 system and I wanted to migrate to the Open Cloud I would deploy a new to CentOS 5.6 system. In this example these instances are similar enough be migrated successfully even though they are not an exact match. While it is completely viable to migrate mismatched OS types, I recommend that you perform a Distribution Upgrade to "at least" the base version of the Target system, if you are caught in one of these mismatch scenarios. I have not encountered any of these types of issues that were deal breakers, but it is always a better to plan to plan for success.

Migrate using RSYNC The Easy Way
To view the script click here

Script Method

To download the script you will have to use git.

git clone git://github.com/cloudnull/InstanceSync.git  
Once you have the script you will have to make it executable or call a 
BASH shell before the script.

Here is how to make the script executable

chmod +x ./rsrsyncLive.sh  
Then you simply run it.

./rsrsyncLive.sh
Here is how to call a bash shell and run the script.

bash ./rsrsyncLive.sh  
Once started please read what the script is telling you. Failure to do 
so could result in a migration failure.

Walkthrough of Easy Migration Process
Here is a walk through for the scripts operation
Once you have deployed your new Open Cloud Instance, you will need to go to the Source cloud server and being the migration process. If you have decided to use one of the scripts you will have to download it on to the Source server and run it, everything else will be taken care of for you. Please remember that the script must be run as the ROOT user. While it is possible for this to work with SUDO, I would recommend that you escalate to ROOT.

Once the script has been started it will prompt you for the IP address of the Target server. This will be the IP address of your newly created Open Cloud Server. The script will then ask you what directories you are wanting to migrate from. If you are simply moving the entire server from one place to another, you can press "Enter" and the script will make the assumption that the entire Source server will be migrated; the script will assume this is done with the "/" partition. If you would like to only move a certain Source directory you can specify it at this time, but If you Specify anything in this field you will also need to specify the target directory too.

The script will then do an OS detection on the Source which will be used to ensure the Target is setup correctly. The OS detection will be used on the Target server for preserving the networking. This is why it is important to choose a similar distribution to the Source server when you deploye your new Open Cloud Server.

After the OS Detection is completed the script will setup a key based authentication from the Source server to the Target server. This is all done on standard ports using the ROOT user. Don't worry though, you do not need to make any configuration changes on your Source server if you have disabled the ROOT user for use with SSH. On the Source server you will be sending only outbound traffic, so no changes need to be made. The script will send the key based authentication from the Source server to the Target server. At this time you will need to enter the password for the Target Server.

Once your password has been excepted, the script will ensure that the RSYNC application has been installed on the Target Server. After installation the script will then commence the migration. This can take a while, at this point I would go get some coffee, you can refer to the estimated time for migration provided earlier.

Once the RSYNC's first pass is complete, the script will wait 5 seconds and then perform a second pass. This second pass is a little different from the first pass. The first pass used time stamps to evaluate what needed to be copied over. The second uses checksums to evaluate the differences and if there is anything that the first pass missed, the script we will get it on the second pass. Once this operation is is completed, the script will reboot the Target server.

After a few seconds your Target Server should be back online. I would recommend that you login to the Target server and make sure everything is functional. Please remember that the password that was provided to you when the Open Cloud server was first deployed will no longer be valid. The Target server will be a Clone of the Source server, so the user restrictions, passwords, and firewall rules will all be active on the new instance.

Doing the RSYNC Operation By Hand
For those of you that don't enjoy simplicity and or enjoy pain, maybe you are a command line masochist, here is how to migrate your Legacy Cloud Server to the Open Cloud by hand.

To do this migration effectively I would open two terminals, one to connect to the Source server and the other to connect to the Target server. I make this recommendation so that you have two active and open SSH sessions during this process. Once you have your active session Make sure that you have RSYNC installed on your instances.

RSYNC Prerequisites
RSYNC is a fantastic tool that can be used for a lot of different operations. The one thing that is required before you can use RSYNC is that it be installed. When you are setting up you Target Server and your Source Server, you have to install RSYNC on both to allow RSYNC to do it's Magic. RSYNC is found in all package managers and base software repositories that I am aware of. Here is a simple list of how to install RSYNC in most cases.

Debian or Ubuntu

apt-get update  
apt-get install rsync  
RedHat or CentOS

yum install rsync  
OpenSUSE

zypper in rsync  
Arch Linux

pacman -S rsync  
Gentoo

# Should already be installed, but if you have to ask you should probably not use Gentoo. 
Installing Screen
I would also recommend that you install screen on the Source Server so 
that you can maintain a persistent session during this potentially long 
process.

Debian or Ubuntu

apt-get update  
apt-get install screen  
RedHat or CentOS

yum install screen  
OpenSUSE

zypper in screen  
Arch Linux

pacman -S screen  
Gentoo

emerge --sync  
emerge screen  
The Start of the Migration
Now that screen and RSYNC are installed on the Source server, and RSYNC is installed on the Target server we need enter into a screen session on the Source Server and then setup our RSYNC command.

Enter this command to enter a screen session. * This enters a screen session and names the session OCSMigration, which will make it easy to reattach later if need be.*

screen -S OCSMigration  
Creating the Exclusions List
Before we setup our RSYNC command you will need to identify the networking scripts and or configuration files that are need on the Target server to ensure that the networking is not destroyed with the migration. To setup this rule I set a variable in the shell which points to a file or directory that would be needed to ensure that the Target servers networking is not broken. Basically we are going to tell RSYNC that we want to exclude some file or files when we do the migration

Here I am setting up a variable which i will use to build my exclusion file.

EXCLUDEME=/tmp/excludeme.file  
# General Exclude List; The exclude list is space Separated
EXCLUDELIST='/boot /etc/conf.d/net /etc/fstab /etc/hostname /etc/HOSTNAME /etc/hosts /etc/init.d/nova-agent* /etc/mdadm* /etc/mtab /etc/network/* /etc/networks* /etc/network.d/* /etc/rc.conf /etc/resolv.conf /etc/sysconfig/network/* /etc/sysconfig/network-scripts/* /etc/udev/rules.d/* /lock /net /tmp /usr/sbin/nova-agent* /usr/share/nova-agent* /var/cache/yum/*'  
Once I am satisfied with the exclusion file, I take the space separated variables and build them into a list.

# Building Exclude File - DONT TOUCH UNLESS YOU KNOW WHAT YOU ARE DOING
EXCLUDEVAR=$( echo $EXCLUDELIST | sed 's/\ /\\n/g' )  
echo -e $EXCLUDEVAR > $EXCLUDEME  
Next I continue setting my exclude variables. These included variables have been setup so that the system is instructed to skip a "cloudinit"application that is known to cause problems in other Cloud Host Providers.

find / -name '*cloudinit*' >> $EXCLUDEME  
find / -name '*cloud-init*' >> $EXCLUDEME  
Setting the IP Variable
Just as we set a variable up for the excluded networking I also setup a variable for the IP address of the Target server. This is so that I can call the RSYNC command again later, we will need this for the second RSYNC process.

TIP="THE IP ADDRESS OF THE TARGET SERVER"  
Performing the RSYNC Action
Now we can setup our RSYNC command with all of the needed flags and options to ensure that we have a successful migration.

rsync -e "ssh" -rlpEAXogDtSzh -P -x --exclude-from="$EXCLUDEME" / root@$TIP:/;  
In this command you can see that I am using SSH for the RSYNC and a whole mess of flags. I would recommend that you review the RSYNC man page for a full explanation of the flags that are being used, but here is a simplification of the process. We are persevering all of the things found on the Source operating System, showing progress during the operation, not going across file system boundaries, excluding the networking scripts or configuration files we specified earlier, and copying the "/" partition of the Source Cloud Server to the "/"partition on the Target Open Cloud Server. Once your command is ready to go, press the "Enter" and go entertain yourself for a while, this is a potentially long process.

Performing the Second RSYNC Action
Upon completion of the RSYNC command I would recommend that your do it one more time, but with one different flag. In the next command you will see that the only difference is the use of a "c" flag. This flag tells RSYNC to go through the Target system and find anything that is not an exact match of the files found on the Source server, this is done by using checksumming. If there are any miss-matched files found RSYNC will copy over the miss-matched files on the Target server, which effectively completes the cloning process.

rsync -e "ssh" -crlpEAXogDtSzh -P -x --exclude-from="$EXCLUDEME" / root@$TIP:/;  
Restarting the Target Server
Now that the RSYNC command has completed, you should go to the terminal session on the Target server, which you logged into earlier, and reboot the Target server. This can be done using this simple command.

shutdown -r now  
If you did not login to the Target server or if you are no longer logged into the Target server you can also perform the reboot from the control panel, or via the API. No matter how you do it, you MUST reboot the Target server in order to complete the Migration Process. Upon Completion of the reboot, you will have an exact clone of the Source Server but with new IP addresses On the Rackspace Open Cloud.

What needs to be done to move your Amazon AMI Linux to another Provider
In order to move your Amazon AMI Linux Server away from Amazon EC2 to another Provider you can use the previously described method. However, you will need to do a bit of clean up to complete the operation.

Thoughts Before the Move 
In order to successfully move your Amazon AMI Linux server you are going to have to move your Amazon Instance to a target instance that is similar. I have had great success moving into CentOS 6+, RHEL 6+, and Fedora 15. The success of this migration will hinge on choosing the correct Target Instance. The basis for all of my tests were done using the most current AMI Instance available which is 2012.03, but the process should work on all current AMI instances.

Before You do your First Migration Pass 
Before you perform your first Migration pass, you will need to make a few additions to the exclude file. These additions are to avoid issues that pertain only Amazon AMI Linux.

Amazon AMI Specific Exclusion List
Just as I had done earlier, I setup a variable before to build my excludes. This makes it easy for me to see and understand.

# Amazon Exclude List; The exclude list is space Seperated
AMAZONEXCLUDELIST='/etc/issue /etc/selinux/config /etc/sysctl.conf /etc/yum.repos.d/amzn-*'  
I will then add these additional excludes to the exclude file we setup earlier.

AMAZONEXCLUDEVAR=$( echo $AMAZONEXCLUDELIST | sed 's/\ /\\n/g' )  
echo -e $AMAZONEXCLUDEVAR >> $EXCLUDEME  
Once I have added the additional Excludes I perform the RSYNC Action.

Post RSYNC Clean up of an AMI Instance
After the completion of the migration we are going to need to remove several RPMs that were installed by Amazon. These RPM cause system problems and can be reinstalled on the target if you choose, but I have found removing them is faster and just as effective.

for i in $( rpm -qa | grep -i -e epel-release -e system-release -e sysvinit -e perl-io -e perl-file -e perl-http -e perl-lwp -e perl-net -e aws -e perl-libwww ); do rpm -e --nodeps $i; done  
Additionally you will have to repair the YUM-Version due to an RPM called "system-release". This RPM can also be removed and the proper release RPM installed, or you can glean the distribution info from the /etc/issue file and force yum to use a different release server type. I find throwing the variable in there is easy and quite effective.

echo "$( cat /etc/issue | head -1 | awk '{print \$3}' | awk -F '.' '{print \$1}' )" > /etc/yum/vars/releasever  
Kevin Carter
Developer, Cloud Builder, Loves Open Source, and Hates Nonsense (I know, those last two items are closely related).

San Antonio
