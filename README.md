# Cifscloak
### Linux cifs share mounting using encrypted credentials

Cifscloak is a quick and simple python based solution for encrypting and storing cifs credentials.  
Tested on Ubuntu 20.04.

### Quick start:

1/ Create an encrypted cifstab and add a cifs mount.  
./cifscloak.py addmount --alias <alias_to_mount> --sharename <share_name> --mountpoint <mount_point> --i <cifs_server_address> --options <cifs_mount_options>

`sudo ./cifscloak.py addmount -a films -s myfilms -m /mnt/films -i frankthefileserver -o "ro"`  
`Password:`  

`sudo ./cifscloak.py addmount -a games -s mygames -m /mnt/games -i frankthefileserver`  
`Password:`

2/ Mount one or more cifs shares.  
./cifscloak.py mount --aliases <alias_to_mount1> <alias_to_mount2>

`sudo ./cifscloak.py mount -a films games`

3/ Unmount one or more cifs shares.  
./cifscloak.py mount -u --aliases <alias_to_mount1> <alias_to_mount2>

`sudo ./cifscloak.py -u -a films games`

4/ List cifs share aliases stored in the cifstab.  

`sudo ./cifscloak.py listmounts`

5/ Remove one or more cifs shares.  
./cifscloak.py removemounts --aliases <alias_to_mount1> <alias_to_mount2>

`sudo ./cifscloak.py removemounts -a films games`

### Synopsis
This utility should be run through sudo or directly as root.

The following files are created the first time that addmount is executed:  
> 0755 /root/.cifstab  
> 0400 /root/.cifstab/.keyfile  
> 0644 /root/.cifstab/.cifstab.db  

cryptography.fernet is used to generate the .keyfile and take care of encryption.  
sqlite3 is used to store encrypted cifs info in .cifstab.db

Of course if you have the .keyfile and cifstab.db it is going to be really easy to decrypt and display the passwords so be sure that you place the cifscloak.py script in a secure location and set decent file permissions.  
For example:  
> 500 /usr/bin/cifscloak.py

### Mount cifs shares at boot time through systemd
I tested this systemd file under Ubuntu 20.04 and it seems to work ok, tbh I'm old school and find systemd a little bit tricky at times so this might not be the best solution?

> /etc/systemd/system/cifscloak.service

> [Unit]  
> After=multi-user.target  
> Description=cifscloak cifs share mount operation  
> 
> [Service]  
> Type=oneshot  
> RemainAfterExit=yes  
>  
> #Command to execute when the service is started  
> ExecStart=/usr/bin/python3 /usr/bin/cifscloak.py mount -n films games  
>   
> #Command to execute when the service is stopped  
> ExecStop=/usr/bin/python3 /usr/bin/cifscloak.py mount -n films games  
>   
> [Install]  
> WantedBy=multi-user.target  

> systemctl enable cifscloak  
> systemctl start cifscloak  
> systemctl stop cifscloak  

* Mountpoint directories are automatically created with default permissions.

## Help
### ./cifscloak.py -h
usage: cifscloak.py [-h] {addmount,removemounts,listmounts,mount} ...

cifscloak - command line utility for mounting cifs shares using encrypted passwords

positional arguments:  
  {addmount,removemounts,listmounts,mount}  
                        Subcommands  
    addmount            Add a cifs mount to encrypted cifstab. addmount -h for help  
    removemounts        Remove cifs mounts from encrypted cifstab. removemount -h for help  
    listmounts          List cifs mounts in encrypted cifstab  
    mount               mount -h for help  

optional arguments:  
  -h, --help            show this help message and exit

### ./cifscloak.py addmount -h

usage: cifscloak.py addmount [-h] -u USER -s SHARENAME -m MOUNTPOINT [-o OPTIONS] -i ADDRESS -a ALIAS

optional arguments:  
  -h, --help            show this help message and exit  
  -u USER, --user USER  User name  
  -s SHARENAME, --sharename SHARENAME  
                        Share name  
  -m MOUNTPOINT, --mountpoint MOUNTPOINT  
                        Mount point  
  -o OPTIONS, --options OPTIONS  
                        Quoted csv options e.g. "domain=mydomain,ro"  
  -i ADDRESS, --ipaddress ADDRESS  
                        Address e.g server address or ipaddress  
  -a ALIAS, --alias ALIAS  Unencrypted identifier for this cifs mount  
 

## ./cifscloak.py removemounts -h

usage: cifscloak.py removemounts [-h] -a ALIASES [ALIASES ...]

optional arguments:  
  -h, --help            show this help message and exit  
  -a ALIASES [ALIASES ...], --aliases ALIASES [ALIASES ...]  
                        Remove cifs mounts e.g. -a mnt1 mnt2  

## ./cifscloak.py mount -h

usage: cifscloak.py mount [-h] -a ALIASES [ALIASES ...] [-u]

optional arguments:  
  -h, --help            show this help message and exit  
  -a ALIASES [ALIASES ...], --aliases ALIASES [ALIASES ...]  
                        Mount reference aliases, e.g -a mnt1 mnt2  
  -u                    Unmount the aliased cifs shares, e.g -a mnt1 mnt2  