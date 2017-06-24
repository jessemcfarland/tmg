# Configure FTP, DHCP, and PXE Servers for Kickstart

## Download Installation Media
Download the latest CentOS ISO from http://isoredirect.centos.org/centos/7/isos/x86_64/.

Mount the ISO using the following commands:
```
sudo mkdir /mnt/centos
sudo mount -o loop /path/to/CentOS.iso /mnt/centos
```

## FTP Server
Install required packages and configure `vsftpd`.
```
sudo yum install vsftpd
sudo cp vsftpd.conf /etc/vsftpd.conf
```

Copy the installation media to the anonymous root directory.
```
sudo mkdir -p /var/ftp/pub/centos
sudo rsync -av /mnt/centos/ /var/ftp/pub/centos/
```

Set permissions on the anonymous root directory to allow anonymous access.
```
sudo chown -R nobody.nogroup /var/ftp
sudo find /var/ftp -type f -exec chmod 0644 {} \;
sudo find /var/ftp -type d -exec chmod 0755 {} \;
```

Restart `vsftpd` to activate configuration.
```
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
```

## DHCP Server
Install required packages and configure `dhcpd`.
```
sudo yum install dhcp
cp dhcpd.conf /etc/dhcp/dhcpd.conf # Replace variables with the appropriate values
```

Restart `dhcpd` to activate configuration.
```
sudo systemctl restart dhcpd
sudo systemctl status dhcpd
```

## PXE Server
Install required packages and configure `tftpd`.
```
sudo yum install xinetd tftp-server syslinux
sudo cp xinetd-tftp /etc/xinetd.tftp
```

Prepare `tftpboot` directory.
```
sudo mkdir /var/lib/tftpboot/pxelinux.cfg
sudo cp pxelinux.cfg-default /var/lib/tftpboot/pxelinux.cfg/default # Replace variables with the appropriate values
sudo mkdir /var/lib/tftpboot/centos
sudo cp -r /mnt/centos/images /var/lib/tftpboot/centos
sudo find /var/lib/tftpboot/centos -type f -exec chmod 0644 {} \;
sudo find /var/lib/tftpboot/centos -type d -exec chmod 0755 {} \;
cd /usr/share/syslinux
sudo cp pxelinux.0 menu.c32 memdisk mboot.c32 chain.c32 /var/lib/tftpboot
```

Restart `xinetd` to activate configuration.
```
sudo systemctl restart xinetd
sudo systemctl status xinetd
```
