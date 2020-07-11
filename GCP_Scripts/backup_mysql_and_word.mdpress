```bash script
#!/bin/bash

##  database backup and service backup script 

# Replace "/path/to/gsutil/" with the path of your gsutil installation.
export PATH="$PATH:/snap/bin/"

# Replace "/home/username/" with the path of your home directory in Linux/Mac.
# The ".boto" file contains the settings that helps you connect to Google Cloud Storage.#export BOTO_CONFIG="/home/.boto"

# A simple gsutil command that returns a list of files/folders in your bucket.
# Replace "yourbucketname" with a bucket name from your Google Cloud Storage.
# You can replace this line with your own gsutil command to upload a file, etc.


## date settings 

TODAY=$(date +%Y_%m_%d)
MONTH=$(date +%m)
YEAR=$(date +%Y)


## arguments
database=$1
Bucket=$2
dirloc=$3


#bucket name 
#gs://dbackuptest/wpdbup/$YEAR/$MONTH/


# VARIABLES # 

siteDB=/tmp/siteDB_$TODAY.sql.gz

siteFiles=/tmp/siteFiles_$TODAY.tar.gz

CHECKSUMF=/tmp/siteFile_$TODAY.CHECKSUM 

CHECKSUMDB=/tmp/siteDB_$TODAY.CHECKSUM 

CHECKDLDB=/tmp/DBchsum_$TODAY.txt

CHECKFDL=/tmp/FdlChum_$TODAY.txt


## MYSQL backup config ## 


mysqldump --defaults-extra-file=/etc/.dbup/mysqldump.cnf $database | gzip -c | tee "$siteDB" 

#Testing data integrity 
echo "backing up $database to $siteDB " 

if gzip -t $siteDB ; then 
	echo "File is ok "
else 
	echo "file is corrupt" > /etc/.files2/output.log 
fi 


#Provide secure hash for checksum of integrity of data 
sha512sum  $siteDB > $CHECKSUMDB

#display content 
sha512sum $CHECKSUMDB | grep -Eo '^[^ ]+'  

## Gcloud cloud storage gsutil command to uupload to storage ## 

gsutil -m cp $siteDB $CHECKSUMDB $Bucket$YEAR/$MONTH/

echo "backing up finished "

## Starting Downloading and testing checksum 

echo "Starting Downloading and testing checksum" 

gsutil cp $Bucket$YEAR/$MONTH/siteDB_$TODAY.CHECKSUM $CHECKDLDB
#gs://dbackuptest/wpdbup
#/$YEAR/$MONTH/siteDB_$TODAY.CHECKSUM $CHECKDLDB

sha512sum $CHECKDLDB | grep -Eo '^[^ ]+'  

gsutil cp $Bucket$YEAR/$MONTH/siteFiles_$TODAY.tar.gz /tmp/siteDB_$TODAY.sql.gz

#Checking both checksums 

if [ "$(sha512sum < $CHECKSUMDB)" = "$(sha512sum < $CHECKDLDB)" ]; then
	echo "CHECKSUM OK"
else
	echo "CHECKSUM NOPE"
fi





## WP backup config   ##  




## Tar gz wordpress-website wp config files ##
tar -czvf $siteFiles  $dirloc      #/var/www/html/wordpress-website/ 

#Checking Data integrity 
echo " backing up site files to tmp" 
if gzip -v -t $siteFiles ; then 
	echo "File is ok " 
else 
	echo "File is corrupt" > /etc/.files2/output.log 
fi

#provide secure hash for checksum of integrity of data 

sha512sum  $siteFiles > $CHECKSUMF

#display content 

sha512sum $CHECKSUMF | grep -Eo '^[^ ]+'

## GCP cloud storage gsutil cmd to upload to storage ##


gsutil -m cp  $siteFiles $CHECKSUMF $Bucket$YEAR/$MONTH/  


echo "backup finished" 

#Starting Downloading and testing checksums" 

gsutil cp $Bucket$YEAR/$MONTH/siteFile_$TODAY.CHECKSUM $CHECKFDL

#Display content 
sha512sum $CHECKFDL | grep -Eo '^[^ ]+'  

gsutil cp $Bucket$YEAR/$MONTH/siteFiles_$TODAY.tar.gz /tmp/siteFiles_$TODAY.tar.gz

#Checking both checksums 

if [ "$(sha512sum < $CHECKSUMF)" = "$(sha512sum < $CHECKFDL)" ]; then
	echo "CHECKSUM OK"
else
	echo "CHECKSUM NOPE"
fi


## Remove file in tmp dir upon exit 

rm $siteFiles 
rm $siteDB
rm $CHECKSUMF
rm $CHECKFDL
rm $CHECKDLDB
rm $CHECKSUMDB

exit 0
```
