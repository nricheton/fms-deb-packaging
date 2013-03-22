Adobe Flash Media Server to Debian package script
=============

This script creates a Debian package from the Adobe Flash Media Server installation package. 


Goals
-------
Allow Adobe Flash Media Server installation simply using

    dpkg -i fms_4_5_5.deb
    
Also ease deployment with puppet or chef.


Contributing
------------

Want to contribute? Great! Improve the script and share your changes.


How to run 
-----------

For usage instructions : 
	./packageFms.sh 	
	
Sample use :
    sudo ./packageFms.sh <full path of FlashMediaServer4.5_x64.tar> <buildNumber>


Disclamer
-------

Use this script at your risks : it creates, move, delete files to create the package thus 
it may be dangerous in case of a bug. 

Review the content before running the script.
Use a VM for your first try.

Tested on Ubuntu



