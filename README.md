# Cifscloak
### Mount cifs shares using encrypted credentials

Cifscloak is a simple python based solution for encrypting and storing cifs credentials.  
It plays reasonably well with systemd in that it performs a number of retries meaning that you don't have to call in systemd targets thereby slowing down the boot process.  

### Tested so far  
Ubuntu 20.04, python3.8.

### Quick start:  

1/ Install

`python3 -m pip install cifscloak`  

Script installs to:  
/usr/local/bin/cifscloak.py  

2/ Create an encrypted cifstab and add cifs mounts.  
cifscloak.py addmount --name <give_name_to_mount> --sharename <share_name> --mountpoint <mount_point> --i <cifs_server_address> --options <cifs_mount_options> --user cifsusername

`sudo cifscloak.py addmount -n films -s myfilms -m /mnt/films -i myfileserver -o "ro" -u cifsuser`  
`Password:`  

`sudo cifscloak.py addmount -n games -s mygames -m /mnt/games -i myfileserver -u cifsuser`  
`Password:`

3/ Mount one or more cifs shares.  
cifscloak.py mount --names <name1> <name2>
Or mount all shares.
cifscloak.py mount -a

`sudo cifscloak.py mount -n films games`

4/ Unmount one or more cifs shares.  
cifscloak.py mount -u --names <name1> <name2>
Or unmount all cifs shares named in cifstab
cifscloak.py mount -u -a

`sudo cifscloak.py -u -n films games`

5/ List cifs share aliases stored in the cifstab.  

`sudo cifscloak.py listmounts`

6/ Remove one or more cifs shares from the cifstab.  
cifscloak.py removemounts --names <name1> <name2>

`sudo cifscloak.py removemounts -n films games`

7/ Create systemd file.  

`sudo cifscloak.py systemdfile -a`

#Generated by cifscloak  
[Unit]  
After=multi-user.target  
Description=cifscloak  
  
[Service]  
Type=oneshot  
RemainAfterExit=yes  
ExecStart=/usr/local/bin/cifscloak.py mount -a  
ExecStop=/usr/local/bin/cifscloak.py mount -u -a  
  
[Install]  
WantedBy=multi-user.target  

`sudo cifscloak.py systemdfile > /etc/systemd/system/cifscloak.service`  
`systemctl enable cifscloak`  
`systemctl start cifscloak`  

### Uninstall
`python3 -m pip uninstall cifscloak`  
`rm /root/.cifstab`  

### Synopsis
This utility should be run through sudo or directly as root.

The following directory and files are created the first time that the script is executed:  
> 0755 /root/.cifstab/  
> 0400 /root/.cifstab/.keyfile  
> 0644 /root/.cifstab/.cifstab.db  

cryptography.fernet is used to generate the .keyfile and take care of encryption.  
sqlite3 is used to store encrypted cifs information into /root/.cifstab/.cifstab.db

Of course if you have the .keyfile and cifstab.db it is going to be really easy to decrypt and display the passwords.  
Be sure that the cifscloak.py script is not writable by anyone except root otherwise it will be trivial for users to have it write out the passwords somewhere the next time that the script is executed.

For example:  
> 550 /usr/bin/cifscloak.py

### Mount cifs shares at boot time through systemd
Cifscloak can generate a simple systemd file that seems to work fine for me on Ubuntu. Initially I did not write in any retry mechanism because it just felt sloppy but after systemd gave me a bit of a ride ( through my lack of understanding ) and I read the documentation which suggested that causing the boot to wait is bad, I instead wrote in a retry.  

* Mountpoint directories are automatically created with default permissions.

## Help
### cifscloak.py -h
usage: cifscloak.py [-h] {addmount,mount,removemounts,listmounts} ...  
  
cifscloak - command line utility for mounting cifs shares using encrypted passwords  
  
positional arguments:  
  {addmount,mount,removemounts,listmounts}  
                        Subcommands  
    addmount            Add a cifs mount to encrypted cifstab. addmount -h for help  
    mount               Mount cifs shares, mount -h for help  
    removemounts        Remove cifs mounts from encrypted cifstab. removemount -h for help  
    listmounts          List cifs mounts in encrypted cifstab  
  
optional arguments:  
  -h, --help            show this help message and exit  
  
### cifscloak.py addmount -h

usage: cifscloak.py addmount [-h] -n NAME -s SHARENAME -i IPADDRESS -m MOUNTPOINT -u USER [-o OPTIONS]  
  
optional arguments:  
  -h, --help            show this help message and exit  
  -n NAME, --name NAME  Connection name e.g identifying server name  
  -s SHARENAME, --sharename SHARENAME  
                        Share name  
  -i IPADDRESS, --ipaddress IPADDRESS  
                        Server address or ipaddress  
  -m MOUNTPOINT, --mountpoint MOUNTPOINT  
                        Mount point  
  -u USER, --user USER  User name  
  -o OPTIONS, --options OPTIONS  
                        Quoted csv options e.g. "domain=mydomain,ro"   
 
## cifscloak.py removemounts -h
  
usage: cifscloak.py removemounts [-h] -n NAMES [NAMES ...]  
  
optional arguments:  
  -h, --help            show this help message and exit  
  -n NAMES [NAMES ...], --names NAMES [NAMES ...]  
                        Remove cifs mounts e.g. -a films music  
  
## cifscloak.py mount -h

usage: cifscloak.py mount [-h] [-u] [-r RETRIES] [-w WAITSECS] (-n NAMES [NAMES ...] | -a)  
  
optional arguments:  
  -h, --help            show this help message and exit  
  -u                    Unmount the named cifs shares, e.g -a films music  
  -r RETRIES, --retries RETRIES  
                        Optional ( default: 3 ) - Retry count, useful when systemd is in play  
  -w WAITSECS, --waitsecs WAITSECS  
                        Optional ( default: 5 seconds ) - Wait time in seconds between retries  
  -n NAMES [NAMES ...], --names NAMES [NAMES ...]  
                        Mount reference names, e.g -n films music. --names and --all are mutually exclusive  
  -a, --all             Mount everything in the cifstab.  
  
## cifscloak systemdfile -h

usage: cifscloak.py systemdfile [-h] (-n NAMES [NAMES ...] | -a)  
  
optional arguments:  
  -h, --help            show this help message and exit  
  -n NAMES [NAMES ...], --names NAMES [NAMES ...]  
                        Add named shares to the systemd unit file  
  -a, --all             Add all cifstab shares to the systemd unit file  
  
