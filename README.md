# SG248458-Implementation-Guide-for-IBM-Blockchain-Platform-for-Multicloud Scripts and Packages

This repository contains submodules to:

1. [nfs-helm-dynamic-multiarch - NFS Multi-Architecture Dynamic Provisioning Packages (nfs-client and nfs-full)](nfs-helm-README.md)
2. [IBM_Blockchain_Platform_for_Multicloud_Automation - IBM Blockchain Platform for Multicloud Web Console Installation Automation](automation-README.md)

*Note: Clicking on either of the above will take you to the corresponding README*

## Releases

This repository has releases that contain packages for the nfs-client (`nfs-client-provisioner-bundle.tgz`) and nfs-full dynamic provisioners (`nfs-full-provisioner-bundle.tgz`) including helm charts and multi-architecture images for s390x and amd64. You can load those with cloudctl load-archive in IBM Cloud Private. Please go to the releases section of this repository to download the packages.

## Get this repository

1. clone using ssh key: 
```
git clone --recursive -j8 git@github.com:IBMRedbooks/SG248458-Implementation-Guide-for-IBM-Blockchain-Platform-for-Multicloud.git ibpbook-scripts
```

Note: The final word `MultiCloud-Automation` will be the name of the directory once cloned. You can change this as you see fit to have a different directory name. For example, if you wanted the directory to be named `ibpbook-scripts` you would use `git clone git@github.ibm.com:Garrett-Lee-Woodworth/MultiCloud-Lab-Automation.git ibpbook-scripts`

2. clone using https: 
```
git clone --recursive -j8 https://github.com/IBMRedbooks/SG248458-Implementation-Guide-for-IBM-Blockchain-Platform-for-Multicloud.git ibpbook-scripts
```

Note: The note above regarding directory naming applies to the https method of git clone as well.