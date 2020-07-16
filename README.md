# SAP on AWS Immersion Day - SAP HANA Express (HXE) Lab

![GitHub Logo](/images/logo.png)

# System Architecture

The starting point of this lab is a system architecture based on 2 EC2 instances: a HANA database (r5.large instance with SUSE Linux Enterprise Server 15 with `hdbadm` password `Aws12345`) and a Jump Box RDP instance (t2.small instance with Windows Server 2019 and login through RDP username: `Administrator` and password `SAPonAWS$DSAG2019!`). 

The HANA instance is deployed in a private subnet of the default Virtual Private Cloud (VPC). Therefore, the only way to access it is to SSH into it from the Jump Box instance, that is, instead, placed in a public subnet of the default VPC. The Jump Box comes also with an Eclipse-based version of HANA studio pre-installed. This could be used to administrate the HANA instance keeping our security best practices. Moreover, both HANA instance and Jump Box instance come with fully configured Security Groups and IAM roles and permissions to grant minimum priviledge permissions model.
The architecture is fully deployed in eu-central-1 region (Frankfurt).
The deployment of the full architecture has been done through a CloudFormation template. 


![GitHub Logo](/images/logo.png)


# Lab 1 - Set up RDP Jump Box with HANA Studio

In this first lab, the goal is to correctly set up the RDP Jump Box host (located in the public subnet of the VPC) so that you can login into the HANA host (located in the private subnet of the VPC) by using the SAP native tool HANA Studio. There are two steps so: from your laptop you connect to the Jump Box through RDP; and from the Jump Box you connect to the HANA host trough HANA Studio.

### Prerequisites (P)

Before login to the instances, you need to set up some configuration on your HANA host as a prerequisite: 

* Enable the hdbadm (<sid>adm user of HANA) to run AWS CLI commands
* Set up correctly the mapping between HANA host private IP, hostname (hanaonaws01.local) and host alias (hanaonaws01)

### Step P.1

First, log into your HANA database server via Systems Manager Session Manager in the AWS Management Console. 

1. From the AWS Management console, navigate to the `EC2 Dashboard`
2. Go to `Running instances`
3. Click on the checkbox to the left of the SAP HANA EC2 instance (the one of r5.large type)
4. Click on the `Connect` button above
5. Select `Session Manager` as a Connection method
6. Click on the `Connect` button at the bottom of the dialog: a separate window or browser tab will open, which will give you a command prompt to do the following steps.

### **Step P.2**

After you log into the HANA EC2 instance, switch to the `root` user. For the root user, AWS Command Line Interface (AWS CLI) comes already installed in this deployment. For further information on how to install it, look at the [AWS CLI User Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv1.html).

1. Execute the following command: 

```
$ sudo su -
```

Now, you are prompted as `hanaonaws01` user.

### **Step P.3**

Give administrator access to `hdbadm`  user (this is to allow HANA administrator user, `hdbadm`, to have OS control and to run AWS CLI commands) by adding HANA admin to the `sudoers` file. The name of this user will vary based on what you put as the System ID for the HANA Database; for example, if you put “HD5”, then the user would be called instead “hd5adm”. 

1. Execute this command as root user (after `sudo su -`): 

```
$ visudo
```

1. Go to the end of the `sudoers` file and add the following line. 

If you are not familiar with `vi` and `visudo` commands: scroll down to the end of the file; hit the `o` key for opening a new line: at this point you should see the word `INSERT` at the bottom of the screen; add the text below and hit the return key; hit the `ESC` key: at this point `INSERT` should no longer appear at the bottom of the screen; type `ZZ` to save the file and close `visudo`.

```
`hdbadm ALL=(ALL) NOPASSWD: ALL`
```

### **Step P.4**

Keeping the root user (`sudo su -`), create a new entry in the `/etc/hosts` configuration file in order to associate the private IP address of the HANA EC2 instance to the `host-name` and the relative `host alias`. 

1. Lookup in the command line for the private IP address of the HANA EC2 instance by typing:

```
`$ ifconfig`
```

1. Copy the private IP address of the instance (you can find it next to `inet` in the `eth0` section). An alternative way of knowing this IP is to look at the `Description` tab of the HANA EC2 instance: `AWS Management Console` > `EC2 Dashboard` > `Running Instances` > `Select the HANA host` through the radio button next to it > `Description` tab > `Private IP`
2. In the command line, open the `/etc/hosts` file through the `vi` text editor:
3. `$ vi /etc/hosts`
4. Scroll down the content of the file until reaching the line that contains the string: `10.x.x.239 hanaonaws01.local hanaonaws01`
5. Press the `i` key in order to use the `INSERT` mode
6. Comment that line by inserting a `#` character at the beginning of the line
7. Add a new line after the previously commented line, by inserting the following string:

```
`<private-IP> hanaonaws01.local hanaonaws01`
```

The <private-IP> parameter is the value that you copied at point 1 and you should paste at the beginning of the new line; hanaonaws01.local is the host-name; hanaonaws01 is the alias for the host.

1. Save and close the text editor 
2. Try to ping the host-name to check everything works fine: 

```
$ ping hanaonaws01
```

If everything works, you should receive some bytes back from the ICMPv4 protocol.

1. Stop the process by pressing `CTRL`+`C`

Your HANA host has now a correct mapping between its private IP, its host name `hanaonaws01.local` and its host alias `hanaonaws01`. This operation can be also done via [YaST2 on SLES](https://www.suse.com/support/kb/doc/?id=000018501).

### Step 1.1

Now that prerequisites are done, you can start setting up the RDP host. First, connect to your RDP Windows Server instance by navigating to the `EC2 Dashboard` in AWS Management Console and by using a pre-configured Remote Desktop File containing the Public IP address of the RDP instance.

1. In the AWS Management Console, go to the `EC2 Dashboard` from the `Services` panel. 
2. Select the RDP EC2 instance (a.k.a. Jump Box); 
3. Click the `Connect` button at the top of the EC2 Dashboard screen
4. Select the option `A standalone RDP client`
5. Select `Download Remote Desktop File` and save it on your laptop
6. Once downloaded, open the file and select `Continue`
7. Insert the User Name and Password parameters:
    1. User Name: `Administrator`
    2. Password: `SAPonAWS$DSAG2019!`
8. Select `Done` and select `Continue` again
9. At that point, the Desktop of the RDP Jump Box instance will appear.

### Step 1.2

Now that you are logged into the RDP host, set up the HANA Studio to connect to the HANA instance. HANA Studio is provided as a plugin for the Eclipse IDE. The IDE comes pre-installed in the RDP host.

1. Double click on the icon of `Eclipse Java Neon`
2. When opening the SAP HANA Studio for the first time, you could get a prompt to select a directory as a workspace. If it happens, please, accept the default location. Moreover, if prompted to create a password hint for the master password, select `No`. If it doesn’t happen, you should directly land to SAP HANA Administration Console.
3. You should land to the SAP HANA Administration Console by default. If the Java perspective is set as as the default perspective for the IDE, click on the top-right button (with a Swiss-knife icon) `SAP HANA Administration Console` to change the current perspective (view) the HANA Administration console one


[Image: Screenshot 2020-06-19 at 17.38.59.png]

1. Under the `Systems` tab, select the down-arrow icon and click on `Add System...`


[Image: Screenshot 2020-06-19 at 17.37.48.png]

1. Fill in the parameters as follows: 

    1. Host Name: `<Private IP of HANA host>`. You can find it going to: 
        `AWS Management Console` > `EC2 Dashboard` > `Running Instances` > `Select the HANA host` > `Description` tab > `Private IP`
    2. Instance Number: `00`
    3. Mode: `Multiple containers`
        1. Tenant database: `HDB`
    4. Description: `HANA Studio for HANA host`
    5. Locale: `English (United States)`
    6. Folder: `/`


1. Select `Next`
2. Insert the following parameters: 
    1. User Name: `SYSTEM`
    2. Password: `Aws12345`
3. Select `Finish`


[Image: Screenshot 2020-06-19 at 16.51.38.png]
You should now be able to navigate in the catalog and see all the tables and schemas of your HANA database host.



# Lab 2 - SAP HANA instance AMI and resizing 

An EC2 Amazon Machine Image (AMI) can be used to create a blueprint of the currently running HANA instance and, to create additional EC2 instances starting from that. The goal of this lab is to deploy a secondary (standby) EC2 HANA server in another Availability Zone of the same Region (Frankfurt), starting from the currently running HANA virtual server. In order to keep consistency, by the way, it’s recommended for production environments (not in this lab), to stop the primary server before taking an Image from it. 

### Step 2.1

Before proceeding with taking the Image from the HANA EC2 server, you need to disable `HANA Autostart` first: 

1. After repeating the steps P.1 and P.2 of the Prerequisites of Lab 1 to be logged to HANA host through AWS Systems Manager Session Manager, login to HANA at OS level using `hdbadm` user: 

```
$ su - hdbadm
```

1. Run the command `cdpro` to go to the profile directory

```
$ cdpro
```

1. From the profile directory (`/usr/sap/SID/SYS/profile`), open the configuration file `HDB_HDB00_hanaonaws01.local`:

```
$ vi /usr/sap/HDB/SYS/profile/HDB_HDB00_hanaonaws01.local
```

1. Press the `i` key to go to `INSERT` mode and change the value of `Autostart` parameter from `1` to `0`: 

```
Autostart = 0
```

1. Press `ESC` key to exit `INSERT` mode
2. Save and close by pressing `SHIFT`+`Z`+`Z`

### Step 2.2

Now that you have disabled HANA `Autostart` parameter, you can create the AMI from the primary HANA server in order to deploy a secondary HANA server in another Availability Zone, to start a Highly Available setup.  This lab doesn’t aim to handle the failover in case of a disaster: for that, the best option is to use a clustering software (e.g. [SLES HA Add-on](https://www.suse.com/media/white-paper/sap_solutions_high_availability_on_SLES_for_sap_apps.pdf)). In this case, we only want to show how easy is to spin up a secondary HANA instance starting from the Amazon Machine Image of the primary HANA server.

1. In the AWS Management Console, go to the `EC2 Dashboard` from the `Services` panel. 
2. Select the primary HANA instance
3. Check the Availability Zone (AZ) tab in which it has been deployed and note it somewhere (you will choose another AZ to deploy the secondary HANA instance)
4. Select `Actions` 
5. Select `Image` 
6. Select `Create Image`
7. In the AMI dialog, enter the following parameters: 
    1. Image Name: `HANA Primary Image`
    2. Image Description: `HANA Primary Server Image`
    3. Leave `No reboot` checkbox unchecked. This ensure consistency for the image.
    4. Check `Delete on Termination` for all EBS Volumes (should be checked by default)
    5. Select `Create Image`
8. You will get a dialog box with a clickable string that says `View pending image ami-xxxxxxx`. Click on it and you will be redirected to the AMIs section of the AWS Management Console. Alternatively, go to `Services` > `EC2` > `AMIs `(under `Images`). Observe that status of AMI is `pending`, since the image is not yet created. 
9. Wait until `Status` changes from `pending` to `available`
10. Click on the radio button next to the AMI
11. Select `Actions` and then `Launch`
12. Select the following parameters for the Secondary HANA instance and click `Next` for each step. Leave the rest of not mentioned parameters as default: 
    1. Instance Type: `r5.large`
    2. Subnet: select a subnet corresponding to an Availability Zone that is different from the one of the Primary HANA instance
    3. Auto-assign Public IP: `Enable`
    4. IAM role: if available in the list, choose `mod-xxx-InstanceProfile-xxx` or `AmazonSSMRoleForInstancesQuickSetup`. If not, leave it blank
    5. In `Add Storage` section, leave everything as it is
    6. In `Add Tags` section, select `Add Tag` and write `Name` as a `Key` and `HANA Secondary` as a `Value`
    7. In Configure Security Group section, leave everything as it is (default inbound rule for port 22 for SSH connections) and optionally change the security group name to `hana-secondary-sg`
    8. Select `Review and Launch`
    9. Select `Launch`
    10. Select `Create a new key pair` choosing a simple key pair name (e.g. `sechana`) and click `Download Key Pair`: a .pem file (the key) will be downloaded on your laptop
    11. Select `Launch Instances`
13. Wait for the secondary HANA instance to be successfully deployed (you can check it in the `EC2 Dashboard`) and until `Status Checks` are 100% completed and `Alarm Status` shows `OK` 
14. Once ready, connect to the secondary HANA instance by using Systems Manager Session Manager if the IAM role set for the instance is `mod-xxx-InstanceProfile-xxx` or `AmazonSSMRoleForInstancesQuickSetup`. If this IAM role has not been set, use the downloaded Key Pair to SSH into the instance: 
    1. Select the radio button next to the instance
    2. Select Connect and the option A standalone SSH client
    3. Open a new Linux Shell on your laptop and execute the 2 commands suggested in the Connect dialog from the directory where the .pem file is stored (as an alternative, PuTTY can be used as well): 

```
$ chmod 400 <keyname>.pem
$ ssh -i "<keyname>.pem" ec2-user@ec2-xx-xxx-xxx-xxx.eu-central-1.compute.amazonaws.com
```

Fill in the  `<keyname>` and the `X`s with the name of the .pem file and the public IP address of your secondary HANA instance respectively.

1. Make sure that HANA system is started by verifying all the processes listed as an output of the command `sapcontrol` are listed as `green`

```
$ sudo su -
$ su - hdbadm
Password: Aws12345
$ sapcontrol -nr 00 -function GetProcessList
```

1. If not, launch the following command: 

```
$ sapcontrol -nr 00 -function Start
```

All the HANA processes will be launched after that command terminates his execution. After a minute approximately, all the processes should move to `green` status. You can check it by using the `GetProcessList` command again. In that case, the secondary HANA server would be up and running.


# Lab 3 - Backup for SAP HANA database

In this lab you will establish a backup schedule for your SAP HANA instance using two different approaches: a script launched via the command line and by using the AWS Backint Agent for backups.

* Task 3.1: Set up backups for SAP HANA host
* [Optional] Task 3.2: Backup to S3 via AWS SAP BACKINT



## Task 3.1: Set up backups for SAP HANA host

By using this method, a series of backups are executed for your HANA host such that at first, data from the EBS volumes attached to the HANA instance is copied to an EBS staging volume. As a second step, data in the staging EBS volume is copied to Amazon S3. The operations are scheduled by a Linux Shell script.

### **Step 3.1.1**

First, log into your **primary** HANA database server via Systems Manager Session Manager in the AWS Management Console. 

1. From the AWS Management console, navigate to the `EC2 Dashboard`
2. Go to `Running instances`
3. Click on the checkbox to the left of the SAP HANA EC2 instance (the primary one, not the secondary created in the previous lab)
4. Click on the `Connect` button above
5. Select `Session Manager` as a Connection method
6. Click on the `Connect` button at the bottom of the dialog: a separate window or browser tab will open, which will give you a command prompt to do the following steps.

### **Step 3.1.2**

After you log into the HANA EC2 instance, switch to the root user. For the root user, AWS Command Line Interface (AWS CLI) comes already installed in this deployment. For further information on how to install it, look at the [AWS CLI User Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv1.html).

1. Execute the following command: 

```
$ sudo su -
```

### **Step 3.1.3**

Since in lab 2 you created an AMI from the primary HANA instance, at the time of creation, you needed to reboot the primary instance. Therefore, it is necessary to start again the processes of the HANA system at OS level. In order to do so, launch the following command as an `hdbadm` user: 

```
$ su - hdbadm
Password: Aws123
$ sapcontrol -nr 00 -function Start
```

Wait for a minute since the processes are in GREEN status. 
You can check it through the command: 

```
$ sapcontrol -nr 00 -function GetProcessList
```

### **Step 3.1.4**

Create two Amazon S3 buckets for storing the backups of the HANA database. The first S3 bucket will be used to store SAP HANA backups done via Shell script; the second S3 bucket will be used to store SAP HANA backups done via AWS Backint Agent for SAP HANA. 

It is possible to create an Amazon S3 bucket both via AWS Command Line Interface (AWS CLI) and manually, via AWS Management Console. 

**AWS CLI**
If you want to create the bucket via AWS CLI, once logged into the HANA instance (`sudo su -`), write the following AWS CLI statement to create a new S3 bucket. Note: please, use `eu-central-1` as Region parameter, as all the infrastructure is hosted in Frankfurt region. Please, also use only lowercase (and URL compliant) characters for the S3 bucket names. 

```
$ aws s3 mb s3://hana-backup-<name initial + surname initial>-<last 2 digits of birth year> \
--region <aws region you are using>
```

Repeat the steps above for creating a second bucket, adding the prefix `-backint` to the bucket name. This second bucket is going to be used for setting up backups of the HANA host via AWS Backint Agent. 

In the case of Jeff Bezos, for example, the bucket names would be respectively: `hana-backup-jb-64` and `hana-backup-jb-64-backint`. 

**AWS Management Console**
As an alternative to the AWS CLI, you can create the S3 bucket manually by going to `Services` > `S3` from the AWS Management Console and by following these steps: 

* Select** **`Create bucket`
* Enter the following Bucket name: `hana-backup-<name initial + surname initial>-<last 2 digits of birth year>`
* Select the **Region**: `EU (Frankfurt)`
* Select `Create bucket`

Repeat the steps above for creating a second bucket, adding the prefix `-backint` to the bucket name. This second bucket is going to be used for setting up backups of the HANA host via AWS Backint Agent.

In the case of Jeff Bezos, for example, the bucket names would be respectively: `hana-backup-jb-64` and `hana-backup-jb-64-backint`. 

### **Step 3.1.5**

As HANA administrator user (`su - hdbadm`), create a key called BACKUP for the `SYSTEM` user of the HANA database, using the password that has been provided in the CloudFormation template used to deploy the HANA instance (`Aws12345`). 

```
`$ su - hdbadm
Password: Aws12345
$ hdbuserstore SET BACKUP "localhost:30015" SYSTEM Aws12345
$ hdbuserstore list`
```

In this case the HANA port number is `30015`, since the HANA instance number chosen by the CloudFormation template is `00`. In general, the port is `3<HANA_ID>15` for single tenant databases. The port is 3<HANA_ID>13 for multi-tenant databases.

### **Step 3.1.6**

Create and execute a Linux Shell script for executing the HANA database backup. The script aims to perform the following operations: 

* Use `hdbsql` (an SQL-based statement) to trigger a database backup to the backup staging EBS volume
* Copy the database backup `data` directories from the backup staging EBS volume to an S3 bucket
* Copy the database backup `log` directories from the backup staging EBS volume to an S3 bucket

Follow the tasks described above:

1. Create and open a new file in the default working directory of the HANA administrator user (`/usr/sap/HDB/HDB00/`) by using `vi` text editor, and call it `hana_backup.sh`

```
`$ vi /usr/sap/HDB/HDB00/hana_backup.sh`
```

1. Copy and paste in the script file the following lines: 

```
`#!/bin/sh
#set -x
S3Bucket_Name=<YOUR-S3-BUCKET>
TIMESTAMP=$(date +\%F\_%H\%M)
#exec 1>/backup/data/${SAPSYSTEMNAME}/${TIMESTAMP}_backup_log.out 2>&1
echo "Starting to take backup of Hana Database and Upload the backup files to S3"
echo "Backup Timestamp for $SAPSYSTEMNAME is $TIMESTAMP"
BACKUP_PREFIX=${SAPSYSTEMNAME}_${TIMESTAMP}
echo $BACKUP_PREFIX
#source HANA environmentsource 
$DIR_INSTANCE/hdbenv.sh
hdbsql -U BACKUP "backup data using file ('$BACKUP_PREFIX')"
echo "HANA Backup is completed"
echo "Continue with copying the backup files in to S3"
echo $BACKUP_PREFIX
sudo -u root /usr/local/bin/aws s3 sync /backup/data/${SAPSYSTEMNAME}/ s3://${S3Bucket_Name}/bkps/${SAPSYSTEMNAME}/data/ --include "${BACKUP_PREFIX}*" --exclude "*20191013_COMPLETE_DATA_BACKUP*"
echo "Copying HANA Database log files in to S3"
sudo -u root /usr/local/bin/aws s3 sync /backup/log/${SAPSYSTEMNAME}/ s3://${S3Bucket_Name}/bkps/${SAPSYSTEMNAME}/log/ --include "log_backup*" --exclude "*20191013_COMPLETE_DATA_BACKUP*"`
```

1. Edit `<YOUR-S3-BUCKET>` at line 2 by inserting the name of the first S3 bucket you’ve created at **step 3.1.4** (`hana-backup-<name-initial+surname-initial>-<last-2-digits-of-birth-year`). To edit files using vi or visudo, refer to step 

1. Save and close the file by pressing `ESC` and then `SHIFT`+`Z`+`Z`

1. Update the script permissions by writing the following line to provide read and execute access to the script for everyone: 

```
`$ chmod 755 /usr/sap/HDB/HDB00/hana_backup.sh`
```

1. Execute the script by typing: 

```
`$ /usr/sap/HDB/HDB00/hana_backup.sh`
```

1. Check the backups in the S3 bucket (`hana-backup-<name initial + surname initial>-<last 2 digits of birth year>`): you should find a folder called `bkps`, that contains another folder called `HDB` (instance name). In this last folder you can find both `data` backups folder and `logs` backups folder. In the log folder you should find both `HDB_DB` tenant logs and `SYSTEMDB` logs.



## [Optional] Task 3.2: Set up backups for SAP HANA host via AWS BACKINT Agent

AWS Backint Agent for SAP HANA (AWS Backint Agent) is an SAP-certified backup and restore application for SAP HANA workloads running on Amazon EC2 instances in the cloud. Backint Agent can back up (in full, incremental, and differential mode) your SAP HANA database to Amazon S3 and to restore it using SAP HANA Cockpit, SAP HANA Studio, and SQL statements. The difference between using Backint and using a Linux Shell script, is that in this last case you don’t need an EBS staging volume to store backup data: data is directly copied from your HANA EBS volumes to Amazon S3.

### **Step 3.2.1**

Install the AWS Backint Agent for SAP HANA on the HANA host by using an AWS Systems Manager document. Note: It is possible to install the AWS Backint Agent also by using AWS Backint installer. For further details on how to install the agent with AWS Backint installer and on how to configure the agent, view logs, and get the current agent version, please visit the SAP on AWS Technical Documentation > [SAP HANA Guides](https://docs.aws.amazon.com/sap/latest/sap-hana/aws-backint-agent-installing-configuring.html).

1. From the AWS Management Console, choose `Systems Manager` under Management & Governance, or enter `Systems Manager` in the `Find Services` search bar

1. From the `Systems Manager` console, choose `Documents` under `Shared Resources` in the left navigation pane

1. On the `Documents` page, select the `Owned by Amazon` tab. You should look for a document named `AWSSAP-InstallBackint` through the search bar.

1. Select the `AWSSAP-InstallBackint` document and choose `Run` command
2. Under the Command parameters, enter the following parameters:
    1. Choose the `Default` document version.
    2. `Bucket Name`: enter the name of the Amazon S3 bucket where you want to store your SAP HANA backup files (`hana-backup-<name initial + surname initial>-<last 2 digits of birth year>-backint`).
    3. `Bucket Folder`: optionally, enter the name of the folder within your Amazon S3 bucket where you want to store your SAP HANA backup files. Leave this blank in this case. 
    4. `System ID`: enter your SAP HANA System ID (`HDB` in this case).
    5. `AWS Region`: enter the AWS Region of the Amazon S3 bucket where you want to store your SAP HANA backup files (in this case, the same as the rest of the architecture: `eu-central-1`, that is Frankfurt). AWS Backint Agent supports cross-Region and cross-account backups. You must provide the AWS Region and Amazon S3 bucket owner account ID along with the Amazon S3 bucket name for the agent to perform successfully.
    6. `Bucket Owner Account ID`: enter the account ID of the Amazon S3 bucket where you want to store your SAP HANA backup files. You can find it by clicking on your name in the top right corner of the AWS Management Console and reading under Account. Don’t forget to delete hyphens in the account number after pasting it to the field of the form. 
    7. Leave blank the `KMS Key` parameter: you are not going to encrypt backups.
    8. Use `/hana/shared` as `Installation Directory`.
    9. `Modify Global ini file`: choose `modify` to modify the global.ini file.
    10. `Ensure No Backup In Process`: choose `Yes` to confirm that you have disabled existing backups and are ready to proceed with the installation. The SSM document will fail if you choose “No”.
3. Under `Targets`, select `Choose intances manually`, and then choose the HANA instance on which to install it from the list. If you are not able to find your instance in the list, verify that you have followed all of the steps in the prerequisites.
4. Under `Other parameters`, leave the field empty.
5. Leave the rest of the options as default.
6. Choose `Run`.

1. When the agent is successfully installed, you will see the `Success` status under the `Command ID`.

### **Step 3.2.2**

Now that the AWS Backint Agent for SAP HANA is installed on the host, you can proceed with the backup by using SQL statements via `hdbsql` command. Before executing the backup, you are going to create an entry to the HANA database, in order to back up the new content. 

1. First, log into your HANA database server via Systems Manager Session Manager in the AWS Management Console:
    1. From the AWS Management console, navigate to the `EC2 Dashboard`
    2. Go to `Running instances`
    3. Click on the checkbox to the left of the SAP HANA EC2 instance
    4. Click on the `Connect` button above
    5. Select `Session Manager` as a Connection method
    6. Click on the `Connect` button at the bottom of the dialog: a separate window or browser tab will open, which will give you a command prompt to do the following steps.
        
2. Login as HANA administrator for the `HDB` database: 

```
`$ su - hdbadm
Password: Aws12345`
```

1. Create an entry in the HANA `hdbuserstore` to connect to `SYSTEMDB` with the user `SYSTEM`. Use the command `hdbuserstore -i set SYSTEM <hostname>:3NN13@SYSTEMDB SYSTEM <Password>` to perform this task. Note that you can get the hostname of your HANA instance through the `hostname` command (should be `hanaonaws01`). Use the port `30013`, since the HANA instance number in this case is 00. Use `Aws12345` as a password.

```
`$ hostname`
$ hdbuserstore -i set SYSTEM hanaonaws01:30013@SYSTEMDB SYSTEM Aws12345
```

1. Launch `hdbsql` command to execute SQL statements against the database and list the latest 10 backups of the backup catalog (some old backups could appear): 

```
$ hdbsql -U SYSTEM
>> select top 50 * from m_backup_catalog order by SYS_START_TIME DESC
```

1. Execute a full backup of the system database by using the following SQL statement:

```
>> BACKUP DATA USING BACKINT ('/usr/sap/HDB/SYS/global/hdb/backint/SYSTEMDB/')
```

The backup process can take quite a bit of time (minimum 30m) with the default settings of the AWS Backint Agent. It is possible to fine tune the agent in order to get better performances to speed-up the backup process by increasing the size of the `data_backup_buffer_size` and the number of `parallel_data_backup_backint_channels`. More information can be found in the [AWS SAP HANA Guide](https://docs.aws.amazon.com/sap/latest/sap-hana/aws-backint-agent-installing-configuring.html#aws-backint-agent-sap-hana-parameters). 

1. Check the backups creation in the destination Amazon S3 bucket via AWS Management Console
2. While performing the backup, you can move to the following task by opening a new connection with the HANA instance by AWS Systems Manager Session Manager (as done at the beginning of this step)
3. [Optional] At finished backup, you can list again the backups through the previous SQL statement in hdbsql:

```
>> select top 50 * from m_backup_catalog order by SYS_START_TIME DESC
```

In this case, the new backup files will be listed as an output.


