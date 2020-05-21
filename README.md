# Transfer file between server using ASPERA

## General Information
### Aspera CLI Description
This program is capable of transferring file from source to destination server using command line interface to interact with the program.
This guide based on Aspera cli version 3.9.6 Linux Version, Installed on Ubuntu Server 16.04 LTS.
In the example, source data are stored on NSTDA server and file will be transferred to Google Cloud Storage. 
### Prerequisite
- Computer or virtual machine installed with Ubuntu Server 16.04 LTS or later.
- Google Cloud Storage within the close location to the computer or virtual machine above.
### Caution
- It is recommended to use Google Virtual Machine that has location within the same zone as Google Cloud Storage. It is to minimize or prevent any unexpected charge in case there are outgoing traffic from the Google Cloud Storage to the Virtual Machine when software is in checksum verification process.
- For more information about the service charge of Google Cloud Storage, [please follow this link](https://cloud.google.com/storage/pricing#network-pricing).
- User also be able to set alert for Google Cloud charges by using Budgets & alerts menu. [Please proceed through this link](https://cloud.google.com/billing/docs/how-to/budgets) for more information.
### Script Information
- **aspera** : This script can interact with Aspera transfer servers by several method. This script will not work if SSH port has been blocked on the Aspera transfer servers.
- **ascp** : This script let user quickly transfer file to and from Aspera transfer servers. 
- **ascp4** : This script has similar function to the ascp script but optimized for sending extremely large sets of individual files.

## Usage
This entire process will copy data from NSTDA server to the user's own server before writing data to Google Cloud Storage. 

### Installation
For detailed guide: [Aspera CLI 3.9.6 Linux Version Guide](https://download.asperasoft.com/download/docs/cli/3.9.6/user_linux/webhelp/index.html#dita/cli_install_container.html)

1. Download Aspera CLI installation shell script [from the website](https://downloads.asperasoft.com/en/downloads/62)
2. Upload the downloaded script to the server.
3. Run the installation shell script. In case of permission denied, user may need to grant additional permission to the file using following command
```
chmod +x aspera-cli-x.x.x.xxx.xxxxxxx-linux-xx-release.sh
```
4. After the installation has been completed, declare Aspera CLI in the PATH environment variable for easy access to the script using following command
```
export PATH=~/.aspera/cli/bin:$PATH
```

### `screen` Command
When transferring large file with limited connection speed using Virtual Machine. It is inconvenient to monitoring the process at all time.
screen command will enable user to create a separate session that still working in backgroud even after user has been disconnected.

This command is not installed by default, user can manually install this command using
```
sudo apt install screen
```

Then user can create separate session simply using 
```
screen
```
After the session has been created, user may execute any command normally.
When user need to detach the session, Press `ctrl+a` then press `d` to detach session.
The detached session will continue working in the background.

User can restore detached session by using
```
screen -r
```
This will return user to the detached session command window.

In case there is more than one detached session running in the background, use this command to get a list of the detached session.
```
screen -ls
```
The list of name and id of each detached session will appear. Then using
```
screen -r (session number)
```
to restore specific session.

For example, when using `screen -ls`, the returned output are:
```
    10835.pts-0.linuxize-desktop   (Detached)
    10366.pts-0.linuxize-desktop   (Detached)
```

User may use the following command to restore 10835.pts-0.linuxize-desktop session.
```
screen -r 10835
```

For more information about screen, please see [this article](https://linuxize.com/post/how-to-use-linux-screen/) or access [full user manual](https://www.gnu.org/software/screen/manual/screen.html)

### Transferring Files From Source Server
In this scenario, NSTDA server only allowed connection on port 80 and 33001. There are large number of files that required to be downloaded.
Thus, ascp4 script are selected for file transfer between NSTDA server and user's server.

Using the following command
```
ASPERA_SCP_PASS=(password) ascp4 -P 33001 --policy=fair -l 10g -k 3 (username)@(host url):(source path) (destination path)
```
Where

- `(password)` : Password of the `(username)` required to access the NSTDA server.
- `ascp4` : Calling ascp4 script
- `-P 33001` : Specified TCP port to initiate the FASP session. (In this case, port 33001 has been selected.)
- `--policy=fair` : This option specify bandwidth management rule of this transfer. `fair` means that script will use all available bandwidth, but if the congestion has occured, the transmission rate will be devided equally with the other request.
- `-l 10g` : Maximum transfer rate. "G/g" stands for gigabit per second.
- `-k 3` : Enable the resumption of partially transferred files. `3` means that the script will perform a comparison of full checksum and will resume download if they match, otherwise it will redownload the file.
- `(username)` : Username that are required to access the NSTDA server.
- `(host url)` : URL of the NSTDA server.
- `(source path)` : Specific path on the NSTDA server that user wish to download.
- `(destination path)` : Specific path on the user's server that user wish to store the downloaded files.

For more information about ascp4, please see [ascp4: Transferring from the Command Line with Ascp 4](https://download.asperasoft.com/download/docs/cli/3.9.6/user_linux/webhelp/index.html#entsrv_external/dita-a4/a4_container.html)

It is encouraged that user should download file from the source server to the user's server before writing those file to the Cloud Storage.
Directly transfer files to the Cloud Storage may incur unexpected charges, as script might need to download file from Cloud Storage to calculate checksum.


### Transferring Files To Google Cloud Storage
After the files has been downloaded and checksum verification has been performed on the downloaded file. These files will be transferred to the Google Cloud Storage.
by using gsutil, user will be able to access Google Cloud Storage by using command prompt.

User will need to login first, to enable the access to the specific Google Cloud Storage.
By using command

```
gcloud auth login
```
and following the instruction on the command prompt.

After user has been successfully logged in, use `gsutil rsync` to co 
```
gsutil -m rsync -r (source path) (destination path)
```
Where

- `gsutil` : Calling gsutil command
- `-m` : Cause rsync to run in parallel. This will improve performance in case user performing operations on a large number of files over a reasonably fast network connection.
- `rsync` : rsync is a tool to make content in `(destination path)` as same as `(source path)` by copying file from the source when it is not found or outdated on the destination and delete file from the destination when it is not found on the source.
- `-r` : Sync data recursively. All data and subdirectory in the `(source path)` will be synced. 
- `(source path)` : Specific path on the user's server that user store the downloaded files.
- `(destination path)` : Specific path on the Google Cloud Storage that user wish to transfer files.

For more information about gsutil, please see
[gsutil](https://cloud.google.com/storage/docs/gsutil)
[gsutil - rsync](https://cloud.google.com/storage/docs/gsutil/commands/rsync)

## Contact
This guide has been written by Pitinat A. (pitinat@gmail.com) for internal use of Mahidol University.