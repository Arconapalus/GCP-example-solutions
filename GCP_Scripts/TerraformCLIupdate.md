# Automate TerraForm CLI update
I have created a bash script to use for upgrading the Terraform version. Mostly cumbersome upgrade manual, I took time out to automate this process and verify the integrity of the file. 

**TL:DR** 

If you want to know more about the mechanisms at hand, head [DOWNLOAD](https://www.terraform.io/downloads.html) and [CHECKSUM](https://www.hashicorp.com/security)

First, create the PGP key and name it Hashicorp.asc and save it in the same directory as the Terraform CLI will be installed at. (Click [CHECKSUM](https://www.hashicorp.com/security) to copy the PGP key) 

Second, I stored the Terraform CLI in the dir that I'm working in, save it at /usr/bin if you're using macOS or Linux. 
To start the script, I have created three arguments that will simplify the version and the file version that Hashicorp keeps updating. (for good reasons, I assume  developer want more features in Terraform I'm all for that) 

```bash script 
#!/bin/bash
# Update Terraform version bash script automation

# Variables 
version=$1 #version# 0.2.28

tfver=$2 # terraform_0_2_28_darwin_amd64.zip

tfsha=$3 # terraform_0.2.28_SHA256SUMS

tfshasig=$3 #terraform_0.2.28_SHA256SUMS.sig 


cd $HOME/Path/to/TerraformCLI

wget https://releases.hashicorp.com/terraform/$version/$tfver 

gpg --import hashicorp.asc


curl -Os https://releases.hashicorp.com/terraform/$version/$tfsha

curl -Os https://releases.hashicorp.com/terraform/$version/$tfshasig


gpg --verify $tfshasig $tfsha

shasum -a 256 -c $tfshasig

unzip $tfver

rm $tfver



 


``` 
The above image is showing three variables version number, file with version, SHA256SUMS with version, and SHA256SUMS.sig with version. 
Copy and paste the bash script and chmod +x automateTFCLI.sh, then execute the script with this command 

```
./automateTFCLI.sh 0.12.28 terraform_0.12.28_darwin_amd64.zip terraform_0.12.28_SHA256SUMS terraform_0.12.28_SHA256SUMS.sig
```

The script takes care of downloading, checking the integrity of the file and then, unzipping and deleting. 
Leaving the Terraform CLI inflation (meaning in use) 

Next run Terraform --version to output version and confirm you have the working Terraform. 
(side note, I was tired of typing long cli commands so I put an alias in bash_profile to shorten it i.e alias tf="terraform" and also 
tfplan="terraform init && terraform plan") 


