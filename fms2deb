#!/bin/bash

# Usage
#
if [ -z "$1" ]; then 
   echo "Packaging script for Flash Media Server : FlashMediaServer4.5_x64.tar -> fms_4_5_x.deb"
   echo "Usage : sudo ./fms2deb <full path of FlashMediaServer4.5_x64.tar> <buildNumber>"
   echo ""
   echo "Only works on a Debian-based system"
   echo "Needs to be run as root"
   echo "A gpg key is required for signing. Run gpg --gen-key if you don't have one."
   exit 0
fi
 
 
# Start packaging
#
FMS_TARGZ="$1"
echo "Original FMS archive  : $FMS_TARGZ"
FMS_BUILD="$2"
if [-z $FMS_BUILD ]; then
	FMS_BUILD="1"
fi

echo "FMS build number : $FMS_BUILD"

# Create temporary directory
#
TEMP_DIR=`mktemp -d -t fms.XXXXXXXXXX`
echo "Working in temp directory : $TEMP_DIR"



#Extract FMS archive
#
echo "Creating extract dir $TEMP_DIR/original"
mkdir $TEMP_DIR/original
tar -xf $FMS_TARGZ -C $TEMP_DIR/original
ARCHIVE_ROOT=$TEMP_DIR/original/`ls $TEMP_DIR/original` 
echo "Archive root $ARCHIVE_ROOT"


# Find version
#
FMS_VERSION=`echo "$ARCHIVE_ROOT" | sed -r 's/.*_([0-9])+_([0-9])+_([0-9]+)_.*/\1.\2.\3/'`
FMS_VERSION_FILE=`echo "$ARCHIVE_ROOT" | sed -r 's/.*_([0-9])+_([0-9])+_([0-9]+)_.*/\1_\2_\3/'`

if [ -n "$FMS_VERSION" ]; then
	echo "Version FMS found : $FMS_VERSION"
else
	echo "FMS version not found"
	exit 1
fi



# Create deb structure
#
DEB_ROOT=$TEMP_DIR/deb
mkdir $DEB_ROOT



# DEBIAN
# 
mkdir -p $DEB_ROOT/DEBIAN
echo "Package: fms"                                         			>> $DEB_ROOT/DEBIAN/control
echo "Version: $FMS_VERSION-$FMS_BUILD"                                            >> $DEB_ROOT/DEBIAN/control
echo "Section: Adobe"                                                   >> $DEB_ROOT/DEBIAN/control
echo "Priority: optional"                                               >> $DEB_ROOT/DEBIAN/control
echo "Architecture: all"                                                >> $DEB_ROOT/DEBIAN/control
echo "Maintainer: Nicolas Richeton <nicolas.richeton@gmail.com>"    >> $DEB_ROOT/DEBIAN/control
echo "Depends: libnspr4-0d (>=4.8.6), libcap2, locate"                  >> $DEB_ROOT/DEBIAN/control
echo "Description: Adobe Flash Media Server" >> $DEB_ROOT/DEBIAN/control

echo "Import archive content"
mkdir -p $DEB_ROOT/opt/adobe/fms
mv $ARCHIVE_ROOT/* $DEB_ROOT/opt/adobe/fms/


# etc
#
echo "Create /etc/adobe/fms/services : fms & fmsadmin"
mkdir -p $DEB_ROOT/etc/adobe/fms/services
echo "/opt/adobe/fms" >> $DEB_ROOT/etc/adobe/fms/services/fms
echo "fms" >> $DEB_ROOT/etc/adobe/fms/services/fmsadmin

echo "Import /etc/init.d/fms"
mkdir -p $DEB_ROOT/etc/init.d
mv "$DEB_ROOT/opt/adobe/fms/fms"   "$DEB_ROOT/etc/init.d/"

# Delete Whitelist Mac & Windows
echo "Removing Whitelist Mac & Windows"
rm -Rf "$DEB_ROOT/opt/adobe/fms/tools/Whitelist"

echo "Moving far to tools/"
mv  "$DEB_ROOT/opt/adobe/fms/far"  "$DEB_ROOT/opt/adobe/fms/tools/far"

echo "Moving f4fpackager and dependencies to tools"
mkdir "$DEB_ROOT/opt/adobe/fms/tools/f4fpackager"
mv `find  "$DEB_ROOT/opt/adobe/fms" -maxdepth 1 | grep "f4fpackager"` "$DEB_ROOT/opt/adobe/fms/tools/f4fpackager/"
mv  "$DEB_ROOT/opt/adobe/fms/libexpat.so.1"  "$DEB_ROOT/opt/adobe/fms/tools/f4fpackager/"
cp `find  "$DEB_ROOT/opt/adobe/fms" -maxdepth 1 | grep "libadbe*"` "$DEB_ROOT/opt/adobe/fms/tools/f4fpackager/"

echo "Removing checksn, fmsconfig, fmsini, installFMS, scramble, sysconfig"
rm "$DEB_ROOT/opt/adobe/fms/sysconfig"
rm "$DEB_ROOT/opt/adobe/fms/scramble"
rm "$DEB_ROOT/opt/adobe/fms/installFMS"
rm "$DEB_ROOT/opt/adobe/fms/fmsconfig"
rm "$DEB_ROOT/opt/adobe/fms/checksn"

# Do not remove fms.ini : required for postinstall script. Move to tools ?
#rm "$DEB_ROOT/opt/adobe/fms/fmsini"

# Remove sample applications 
# They should already been in /samples directory
find "$DEB_ROOT/opt/adobe/fms/applications/" -mindepth 1 -delete

# This code set the service to autostart.
#echo "Create .autostart"
#touch $DEB_ROOT/opt/adobe/fms/.autostart

# Create .services
echo "Create .services"
echo "/etc/adobe/fms/services"  >  "$DEB_ROOT/opt/adobe/fms/.services"

echo "Create Apache cacheroot"
APACHE=`ls "$DEB_ROOT/opt/adobe/fms/" | grep Apache`
mkdir "$DEB_ROOT/opt/adobe/fms/$APACHE/cacheroot"





# Create logs directory
mkdir $DEB_ROOT/opt/adobe/fms/logs

# Create tmp directory
mkdir $DEB_ROOT/opt/adobe/fms/tmp


# Renaming template configuration files 
echo "Renaming empty confiuguration files as templates"
mv  $DEB_ROOT/opt/adobe/fms/conf/fms.ini   $DEB_ROOT/opt/adobe/fms/conf/fms.ini.template

echo "Removing default keys"
rm "$DEB_ROOT/opt/adobe/fms/phds/common-key.bin"
rm "$DEB_ROOT/opt/adobe/fms/phls/liveeventkey.bin"
rm "$DEB_ROOT/opt/adobe/fms/phls/vodkey.bin"

echo "Changing ownership to www-data"
chown www-data:www-data -R $DEB_ROOT/opt

echo "Setting flags"
chmod -R 777 $DEB_ROOT/opt/adobe/fms/applications
chmod -R 755 $DEB_ROOT/opt/adobe/fms
chmod -R 750 $DEB_ROOT/opt/adobe/fms/conf
find $DEB_ROOT/opt/adobe/fms/conf -type f | xargs chmod a-x

echo "Removing Apache"
rm -Rf "$DEB_ROOT/opt/adobe/fms/$APACHE"

echo "Remove uninstaller"
rm "$DEB_ROOT/opt/adobe/fms/uninstallFMS"

########
# Pre and post install scripts
########

echo "#!/bin/bash" >> $DEB_ROOT/DEBIAN/preinst
chmod 555 $DEB_ROOT/DEBIAN/preinst


echo "#!/bin/bash" >> $DEB_ROOT/DEBIAN/postinst
# libcap.so.1
#
# Locate 64 bit libcap
echo "echo \"Looking for libcap\""											 >> $DEB_ROOT/DEBIAN/postinst
echo "LIBCAP=\`locate  \"libcap.so.2\" | grep \"^[^32]*/libcap.so.2\$\"\`"	 >> $DEB_ROOT/DEBIAN/postinst
echo "if [ -n \"\$LIBCAP\" ]; then"											 >> $DEB_ROOT/DEBIAN/postinst
echo "    echo \"libcap found\""											 >> $DEB_ROOT/DEBIAN/postinst
echo "else" 																 >> $DEB_ROOT/DEBIAN/postinst
echo "    	exit 1" 														 >> $DEB_ROOT/DEBIAN/postinst
echo "fi"																	 >> $DEB_ROOT/DEBIAN/postinst
echo "ln -s  \"\$LIBCAP\" /opt/adobe/fms/libcap.so.1"						 >> $DEB_ROOT/DEBIAN/postinst

echo "if [ ! -f  /opt/adobe/fms/phds/common-key.bin ]; then"										>> $DEB_ROOT/DEBIAN/postinst
echo "	echo \"creating keys\" " 																	>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/tools/scramble/scramble -KeyGen 16 > /opt/adobe/fms/phds/common-key.bin" 	>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/tools/scramble/scramble -KeyGen 16 > /opt/adobe/fms/phls/liveeventkey.bin" 	>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/tools/scramble/scramble -KeyGen 16 > /opt/adobe/fms/phds/vodkey.bin"			>> $DEB_ROOT/DEBIAN/postinst
echo "fi" 																							>> $DEB_ROOT/DEBIAN/postinst

echo "if [ ! -f  /opt/adobe/fms/conf/fms.ini ]; then " 																>> $DEB_ROOT/DEBIAN/postinst
echo "	echo \"creating fms.ini\" " 																				>> $DEB_ROOT/DEBIAN/postinst
echo "	cp /opt/adobe/fms/conf/fms.ini.template /opt/adobe/fms/conf/fms.ini" 										>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini SERVER.ADMINSERVER_HOSTPORT :1111" 				>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini ADAPTOR.HOSTPORT :1935" 							>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini APP.JS_SCRIPTLIBPATH /opt/adobe/fms/scriptlib" 	>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini VHOST.APPSDIR /opt/adobe/fms/applications" 		>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini LIVE_DIR /opt/adobe/fms/applications/live" 		>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini VOD_COMMON_DIR /opt/adobe/fms/webroot/vod" 		>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini VOD_DIR /opt/adobe/fms/applications/vod/media" 	>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini SERVER.HTTPD_ENABLED false" 						>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini SERVER.ADMIN_USERNAME fmsadmin" 					>> $DEB_ROOT/DEBIAN/postinst
echo "  ADMIN_PWD=\`/opt/adobe/fms/tools/scramble/scramble -KeyGen 16\`"											>> $DEB_ROOT/DEBIAN/postinst
echo "  echo \"Generated admin password: \$ADMIN_PWD, please change it NOW\""										>> $DEB_ROOT/DEBIAN/postinst
echo "  pushd \"/opt/adobe/fms/\" > /dev/null "																		>> $DEB_ROOT/DEBIAN/postinst
echo "	echo \$ADMIN_PWD | ./fmsadmin -user fmsadmin -console -conf ./conf/Server.xml > /dev/null"					>> $DEB_ROOT/DEBIAN/postinst
echo "  popd >/dev/null "																							>> $DEB_ROOT/DEBIAN/postinst
echo "  echo \"\" "																									>> $DEB_ROOT/DEBIAN/postinst
echo "	OWNER_ID=\`id www-data | sed -e 's/).*//; s/(.*//; s/^.*=//;'\`"											>> $DEB_ROOT/DEBIAN/postinst
echo "	OWNER_GROUP_ID=\`cat /etc/group | grep \"^www-data:\" | cut -f3 -d:\`" 										>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini SERVER.PROCESS_UID \$OWNER_ID" 					>> $DEB_ROOT/DEBIAN/postinst
echo "	/opt/adobe/fms/fmsini -chgtag /opt/adobe/fms/conf/fms.ini SERVER.PROCESS_GID  \$OWNER_GROUP_ID"				>> $DEB_ROOT/DEBIAN/postinst
echo "fi" 																											>> $DEB_ROOT/DEBIAN/postinst
echo "exit 0" 																											>> $DEB_ROOT/DEBIAN/postinst

chmod 555  $DEB_ROOT/DEBIAN/postinst

echo "#!/bin/bash"						 				>> $DEB_ROOT/DEBIAN/postrm
echo "if [ -f  /opt/adobe/fms/libcap.so.1 ]; then" 	 	>> $DEB_ROOT/DEBIAN/postrm
echo "	rm /opt/adobe/fms/libcap.so.1"	 				>> $DEB_ROOT/DEBIAN/postrm
echo "fi" 												>> $DEB_ROOT/DEBIAN/postrm
chmod 555  $DEB_ROOT/DEBIAN/postrm


########
# Packaging
########


# debug
find $TEMP_DIR

# Create package
echo "Creating fms_$FMS_VERSION_FILE.deb"
dpkg -b $DEB_ROOT $TEMP_DIR/fms_$FMS_VERSION_FILE.deb

# Sign package
# ensure dpkg installed
apt-get install dpkg-sig
dpkg-sig -s builder $TEMP_DIR/fms_$FMS_VERSION_FILE.deb

# Move final deb in current directory
mv $TEMP_DIR/fms_$FMS_VERSION_FILE.deb ./

# Remove tmp directory
echo "Deleting $TEMP_DIR"
rm -R "$TEMP_DIR"
