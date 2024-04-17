+++
title = 'Zabbix appliance configuration'
date = 2024-04-16T15:45:01+03:00
draft = false
+++
**Migrating to Zabbix appliance should be less maintenance but there are some things to set up.**  
 - Change time zone  
`cp /usr/share/zoneinfo/Europe/Vilnius /etc/localtime`
 - Install & enable VM guest agent  
`yum install qemu-guest-agent`  
`systemctl enable qemu-guest-agent`
 - Migrating an existing database one usually needs to change zabbix cache size to something bigger  
`vi /etc/zabbix/zabbix_server.conf`  
`CacheSize=1G`

**As I am migrating a database I need more space in the appliance so this was helpful [by Elier Herrera](https://medium.com/@elierherrera96/how-to-increase-zabbix-appliance-db-disk-space-79ac2eac8279)**  
 - Shutdown the VM, increase the disk size, boot up the VM, SSH into it.  
`sudo fdisk /dev/sda`
 - Type the letter **P** to print the partition table and take note of the starting sector for /dev/sda4 and /dev/sda5 partitions.
 - Type the letter **D**, then **5** to delete the existing /dev/sda5 partition.
 - Type **D**, then **4** to delete the existing /dev/sda4 (extended partition).
 - Type **N**, then **E** (for extended), then **4** for the partition number, and use the start sector of the old /dev/sda4 partition (just **press Enter**).
 - For the last sector, just **press Enter** to accept the default, which should be the end of the available space.
 - Now, type **N**, then **L**, (for logical).
 - The first sector should be the start sector of the old /dev/sda5 (just **press Enter**).
 - Again, for the last sector, just **press Enter** to accept the default, which should be the end of the available space.
 - If you see a prompt asking to remove an existing signature, enter **N** for No, otherwise existing data will be wiped.
 - Type **W** to write changes to the disk & reboot.
 - After the machine reboots, use xfs_growfs to resize the filesystem.  
`sudo xfs_growfs /var/lib/mysql`
 - Finally, verify that the filesystem has been resized correctly. You should see now that the partition /dev/sda5 has more available disk space (depending on how much you allocated in the VM settings)  
`df -h`

**Next is the DB migration [by Markku Leiniö](https://majornetwork.net/2022/01/moving-zabbix-database/)**  
 - SSH into the old VM with the DB and dump it.  
`mysqldump --single-transaction zabbix | gzip > /mnt/zbx-dump.sql.gz`
 - SCP the DB to the new VM.  
`scp /mnt/zbx-dump.sql.gz user@newvmip:/var/lib/mysql/`  
 - **Optional:** Username syntax for remote access (instead of or in addition to 'zabbix'@'localhost'): 'zabbix'@'10.10.10.0/255.255.255.0'
Use the password you are using in Zabbix server configurations. Do not import any SQL schema at this point, we want an empty database to start with.  

 - Use the usual mysql commands to create the new database and correct user and permissions on the new server: create database, create user, grant all privileges.
`mysql -uroot -p<password>`  
`create database zabbix character set utf8mb4 collate utf8mb4_bin;`  
`create user 'zabbix'@'localhost' identified by '<password>';`  
`grant all privileges on zabbix.* to 'zabbix'@'localhost';`  
`SET GLOBAL log_bin_trust_function_creators = 1;`  
`quit;`
 - Import the DB and wait a long time to finish  
`cd /var/lib/mysql`  
`zcat zbx-dump.sql.gz | mysql -u zabbix -p zabbix`
 - Finish the DB configuration  
`mysql -uroot -p<password>`  
`SET GLOBAL log_bin_trust_function_creators = 0;`  
`quit;`  
**Optional:**
 - Set DBHost to the new database server address.  
`vi /etc/zabbix/zabbix_server.conf`
 - Restart Zabbix server  
`systemctl restart zabbix-server`
 - Set $DB['SERVER'] to the new database server address.  
`vi /etc/zabbix/web/zabbix.conf.php`
 - Restart Apache  
`systemctl restart apache2`

**Note:** If you didn’t use the old (existing) database user password in step 3, you need to change also the passwords in the configuration files in this step.

**Looking for errors and as I got errors I had to apply this [from a forum post](https://www.zabbix.com/forum/zabbix-troubleshooting-and-problems/457128-database-problem)**  
`more /var/log/zabbix/zabbix_server.log`  
 - It is necessary rebuilt the trigger in schema data base. You can see the triggers with the command in DB  
`mysql`  
`show triggers in zabbix;​`  
 - Delete triggers in schema database  
`mysql`  
`use zabbix`  
`drop trigger zabbix.items_name_upper_insert;`  
`drop trigger zabbix.items_name_upper_update;`  
`drop trigger zabbix.hosts_name_upper_insert;`  
`drop trigger zabbix.hosts_name_upper_update;`  
 - Create new triggers in DB zabbix  
`mysql`  
`use zabbix`  
`create trigger hosts_name_upper_insert before insert on hosts for each row set new.name_upper=upper(new.name);`  
`create trigger items_name_upper_insert before update on hosts for each row set new.name_upper=upper(new.name);`  
 - Execute unique command in mysql  
`delimiter //`  
`create trigger hosts_name_upper_update before update on hosts for each row begin if new.name <> old.name then set new.name_upper = upper(new.name); end if; end //`  
`create trigger items_name_upper_update before update on items for each row begin if new.name<>old.name then set new.name_upper=upper(new.name); end if; end //`  
`delimiter ;`
