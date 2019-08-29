# IBM Blockchain Platform for Multicloud Automation

The `IBM_Blockchain_Platform_for_Multicloud_Automation/` submodule of this repository contains automation scripts to create new namespaces with the necessary setup for the IBM Blockchain Platform for Multicloud console and deploy said console into each of these namespace chosen. You can clean up all artifacts including helm releases, the clusterrole, and the namespaces with all of the other deployed resources.

## Scripts

In order to accomplish the above stated tasks there are 4 scripts:

1. `Blockchain_Setup.sh` contains the initial variables and is the start script to execute with options depending on your needs described below in `Deploy the IBM Blockchain Platform for Multicloud Helm Chart, creating a namespace for each deployment with the number of charts you desire`.

2. `NamespaceSetup.sh` contains the logic to setup namespaces with necessary credentials based on clusterroles in this repo. Apply with `kubectl apply -f` as mentioned in the **Pre-reqs** section. It also gets the values necessary values for the IBM Blockchain Platform for Multicloud helm chart. For example, it finds first available ports for each chart and hands them out to helm releases in sequential order.

3. `create_optools.sh` contains the helm deploy logic for the IBM Blockchain Platform for Multicloud helm chart

4. `cleanupNamespaces.sh` contains the logic to clean everything up after the lab

## Pre-reqs 

(Install via your ibm cloud private for correct version numbers to match cluster)
Check output of commands to make sure versions much that of your cluster [i.e. client matches server] (an example is below for my cluster):

- kubectl command line (Version 1.12.4)
```
kubectl version 
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.4", GitCommit:"f49fa022dbe63faafd0da106ef7e05a29721d3f1", GitTreeState:"clean", BuildDate:"2018-12-14T07:10:00Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.4+icp-ee", GitCommit:"d03f6421b5463042d87aa0211f116ba4848a0d0f", GitTreeState:"clean", BuildDate:"2019-01-17T13:14:09Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
```

- cloudctl command line (Version 3.1.2)
```
cloudctl version
Client Version: 3.1.2-1203+81b254e18da556ae1d9b683a9702e8420896dae9
Server Version: 3.1.2-1203+81b254e18da556ae1d9b683a9702e8420896dae9
```

- helm cli  (Version 2.9.1)
```
helm version --tls
Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.9.1+icp", GitCommit:"8ddf4db6a545dc609539ad8171400f6869c61d8d", GitTreeState:"clean"}
```

- Use a computer running macos or a linuxos with the bash shell.

- Add neccessary clusterroles (once per cluster)

   You only need to apply these once per cluster, so if they already exist in the correct configuration, you can skip this step.

   If not, apply all while in the `IBM_Blockchain_Platform_for_Multicloud_Automation` you cloned with:

   ```
   kubectl apply -f .
   ```

## Login and Configuration

### Login to your ICP cluster

```
cloudctl login -a https://mycluster.icp:8443 -n default
```

#### Troubleshooting Login

##### Troubleshooting cannot connect error

1. Check to make sure vpn is running, if applicable

2. Check hostname is correct

3. Check that hostname maps to the correct IP in /etc/hosts file with `sudo cat /etc/hosts`. If not add hostname/ip mapping to /etc/hosts with:

   ```
   echo "192.168.22.81   wsc-ibp-icp-cluster.icp" | sudo tee -a /etc/hosts
   ```

##### Troubleshooting x509 error on Login to your ICP Cluster (ONLY If above step [`cloudctl login`] failed)

If this fails with x509 error, it means you need to trust a ca cert for the cluster.

One way to do this on mac is to run:

```
cloudctl login -a https://mycluster.icp:8443 -n default --skip-ssl-validation
```

This will download a ca.pem certificate for your cluster

You can then trust this certificate on macos with:

```
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ~/.helm/ca.pem
```

If you have docker and want to configure this as well, please restart docker to pick up the new certs in docker.

To make sure this has been done successfully use

```
cloudctl login -a https://mycluster.icp:8443 -n default
```

You should now find success. If you don't please troubleshoot further before progressing with a deploy as you will be setting yourself up for failure / wasting time.

### If you don't already have a local helm repo mirror, please configure one

```
helm repo add blockchain-charts --ca-file "${HOME}"/.helm/ca.pem --cert-file "${HOME}"/.helm/cert.pem --key-file "${HOME}"/.helm/key.pem https://mycluster.icp:8443/helm-repo/charts
helm repo update
```

## Deploy the IBM Blockchain Platform for Multicloud Helm Chart, creating a namespace for each deployment with the number of consoles you desire

```
TEAM_NUMBER=<number_of_teams> PREFIX=<chosen_prefix> ./Blockchain_Setup.sh
```

For example, to create `1` instance with PREFIX of `garrett` use:

```
TEAM_NUMBER=1 PREFIX=garrett ./Blockchain_Setup.sh
```

The prefix makes it so different users can coexist. Please check to make sure you prefix isn't being used by a namespace yet with `kubectl get ns | grep prefix` where prefix is the name of your prefix.

**If you wish to use one admin email/ admin username for all consoles instead of team users as initial admin users**


For example, to set up the admin email for 5 consoles use the following setup:

```
TEAM_NUMBER=5 PREFIX=<chosen_prefix> ADMIN_EMAIL="<admin_email>" ./Blockchain_Setup.sh
```

For example:

```
TEAM_NUMBER=5 PREFIX=email-time ADMIN_EMAIL="siler23@ibm.com" ./Blockchain_Setup.sh
```

**If you wish to start off midway through setup due to add additional consoles (or do to a snag), use START_NUMBER=x where x is the team you wish to start with**

For example, to set up team06 to team10 use:

```
TEAM_NUMBER=11 PREFIX=garrett START_NUMBER=6 ./Blockchain_Setup.sh
```

**Default helm repo is *blockchain-charts*. If your icp mirror is named something else (i.e. IloveBeingDifferent)**

```
helm repo update 
TEAM_NUMBER=5 PREFIX=garrett HELM_REPO=IloveBeingDifferent ./Blockchain_Setup.sh
```

**Default is using IBM Z (s390x). If you wish to use x86**

```
   TEAM_NUMBER=5 PREFIX=garrett ARCH=amd64 ./Blockchain_Setup.sh
```

**FYI**: *You can adjust other values in the same way for the other values set in `Blockchain_Setup.sh` as necessary. I have just listed the one I believe will be the most common above.*

### Troubleshooting ProxyIP issue 

Make sure you can get the ProxyIP with

```
kubectl get nodes -l proxy -o jsonpath='{.items[0].metadata.labels.kubernetes\.io\/hostname}' && echo
```

You should get an IP output like: `192.52.32.94` which you can use for the PROXY_IP value when running the script.

If not, then use an IP that your cluster nodePorts are available from for the PROXY_IP and run script with PROXY_IP entered:

```
TEAM_NUMBER=1 PREFIX=garrett PROXY_IP=192.52.32.94 ./Blockchain_Setup.sh
```

### Deployment Description and Resource Notes

This helm chart deploys the **optools pod** (with containers optools, couchdb, configtxlator, deployer, and operator)

Note: Chrome is not supported at this time. Please use Firefox or Safari but not IBM Firefox (since it uses Firefox ESR [Firefox Extended Support Release] instead of the latest)Additionally, you may need to delete your web browser cache from an earlier IBM Blockchain Platform for Multicloud instance for things to load properly if you are having problems seeing the sign-on screen.


The resource settings are set to observed defaults based on cluster max and overall usage for previous deployments. (Thus, some values are lower than regular defaults for lab purposes)
These can be found in the `create_optools.sh` script but should generally not be changed. 

**You can find the details for accessing each console in the note of the IBM Blockchain Platform for Multicloud helm chart**

```
NOTES:

Open IBM Blockchain Console:
https://192.168.22.81:30009
```

## Recommended Access After Deployment
**You can set your team and then run the following command echo command below to get these details for a given team.**

where team00 is your team and `garrett` is your PREFIX in the command `TEAM_NUMBER=<number_of_teams> PREFIX=<chosen_prefix> ./Blockchain_Setup.sh`.

```
team=team00 PREFIX=garrett
echo; echo "****************  ${team}  ****************"; echo "Open IBM Blockchain Console: https://192.168.22.81:$(kubectl get svc ${PREFIX}-${team}-ibp-console-optools -n ${PREFIX}-${team} -o jsonpath='{.spec.ports[0].nodePort}')"; echo "Please go to the following URL and accept the certificate: https://192.168.22.81:$(kubectl get svc ${PREFIX}-${team}-ibp-console-optools -n ${PREFIX}-${team} -o jsonpath='{.spec.ports[1].nodePort}')"; echo "USERNAME: ${team}@ibm.com"; echo "PASSWORD: ${team}pw"
```

**The deployment details for all teams are available for in the `portList.txt` file created by the script so that instructors can see all of these in one place for their reference.**

`portList.txt` will be created in the directory you clone this repo into (Default is `MultiCloud-Lab-Automation`). To view this file you could either open up `portlist.txt` in a text editor or use a terminal command after running the script, such as:

```
cat portList.txt
```

## CLEANUP
There is a cleanup script which will cleanup the helm charts for all of the namespaces as well as created secrets and namespace itself. This makes sure everything is cleaned up after a lab.

The command for cleanup will be printed at the end of `portList.txt` based on the setup command you entered. The full cleanup command will be printed in format:

```
Full Cleanup Command: TEAM_NUMBER=<number_of_teams> PREFIX=<chosen_prefix> ./cleanupNamespaces.sh
```

If START_NUMBER was set greater than 0, the partial cleanup command will also be in `portList.txt` printed in the format:

```
Cleanup Command for This Run: TEAM_NUMBER=<number_of_teams> START_NUMBER=<chosen_starting_team> PREFIX=<chosen_prefix> ./cleanupNamespaces.sh
```


** If you wish to start off midway through deletion due to a snag, use START_NUMBER=x where x is the team you wish to start with **

For example, to delete team06 to team10 use:

```
TEAM_NUMBER=11 PREFIX=garrett PRESTART_NUMBER=6 ./cleanupNamespaces.sh
```

## Notes
This script uses the docker credentials for the namespace that IBP was loaded to as set by `DOCKER_NAMESPACE` in Blockchain_Setup.sh. While theoretically you can set the number of teams as high as you want, there is of course a limit in terms of cluster capacity at a certain point.

This script has been tested up to 15 namespaces / IBM Blockchain Platform for Multicloud deploys. One of these deploys was then successfully tested with e2e deployment (i.e. deploy 2CAs, with 1 peer, 1 orderer, create/join channel, install/instantiate/invoke smart contract).

A run like `TEAM_NUMBER=15 PREFIX=garrett ./Blockchain_Setup.sh` should end as follows with the number of instances setup depending on the number you set:
```
@@@@@@@@  @@@  @@@  @@@  @@@   @@@@@@   @@@  @@@  @@@@@@@@  @@@@@@@   
@@@@@@@@  @@@  @@@@ @@@  @@@  @@@@@@@   @@@  @@@  @@@@@@@@  @@@@@@@@  
@@!       @@!  @@!@!@@@  @@!  !@@       @@!  @@@  @@!       @@!  @@@  
!@!       !@!  !@!!@!@!  !@!  !@!       !@!  @!@  !@!       !@!  @!@  
@!!!:!    !!@  @!@ !!@!  !!@  !!@@!!    @!@!@!@!  @!!!:!    @!@  !@!  
!!!!!:    !!!  !@!  !!!  !!!   !!@!!!   !!!@!!!!  !!!!!:    !@!  !!!  
!!:       !!:  !!:  !!!  !!:       !:!  !!:  !!!  !!:       !!:  !!!  
:!:       :!:  :!:  !:!  :!:      !:!   :!:  !:!  :!:       :!:  !:!  
 ::        ::   ::   ::   ::  :::: ::   ::   :::   :: ::::   :::: ::  
 :        :    ::    :   :    :: : :     :   : :  : :: ::   :: :  :   

It took 4 minutes and 56 seconds to setup 15 IBM Blockchain Platform for Multicloud instances each in an unique namespace
```

You would then cleanup with `TEAM_NUMBER=15 PREFIX=garrett ./cleanupNamespaces.sh` which should end as follows:
```
namespace "team14ns" deleted
+ set +x


       `..   `..      `........      `.       `...     `..
 `..   `..`..      `..           `. ..     `. `..   `..
`..       `..      `..          `.  `..    `.. `..  `..
`..       `..      `......     `..   `..   `..  `.. `..
`..       `..      `..        `...... `..  `..   `. `..
 `..   `..`..      `..       `..       `.. `..    `. ..
   `....  `........`........`..         `..`..      `..

It took 3 minutes and 32 seconds to cleanup 15 IBM Blockchain Platform for Multicloud instances and their namespaces
```