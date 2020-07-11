# Start and stop Google Compute Engine for developing (keeps cost and resources down) 

This start/stop Google Compute Engine vm instance script starts a vm instance for anyone to use and test what then need to in dev environment. 
Once task are done simply exit out of ssh and the script will automatically turn off the vm instances. 

You will need the [GCLOUDSDK](https://cloud.google.com/sdk/install) and setup ssh keys or os login for the vm instance. 

Download to dir and chmod +x startopvm.sh (or whatever name you want to call it) and then enter the input of which vm instances you are using 

```
./startopvm.sh project_name vm_instance zone 
```



```
#! /bin/bash

gcloud info 

gcloud projects list

# arguments

Project=$1   #project id that you'll be working on 

vminsta=$2  #vm instances name that you are starting within that project id 

zone=$3    # zone of vm instance that is running in, this format will be in i.e  --zone=us-central1-a 




gcloud config set project $Project 

gcloud compute instances list 


gcloud compute instances start $vminsta $zone

gcloud compute ssh $vminsta $zone



# An error exit function

error_exit()
{
	echo "$1" 1>&2
	exit 1
}

# Using error_exit

if gcloud compute instances list --filter="status=running"; then

  echo "Instance name: $instances"
	
else
	error_exit "Cannot start!  Aborting."
fi 
  if echo "logout"; then 
  gcloud compute instances stop $vminsta $zone 

  echo "gcloud compute instances stopping"  

else

  error_exit "cannot stop" 
fi
```

When you're done developing or testing simply enter exit in the ssh vm instance and the script will shutdown the vm instance. 

