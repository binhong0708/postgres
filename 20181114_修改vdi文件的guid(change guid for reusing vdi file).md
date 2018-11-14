Step1. Copy VDI file from “D:\VirtualBox
VMs\master_mysql_server\ master_mysql_server.vdi” to “D:\software\Linux” ,and
rename file to “slave1_mysql_server.vdi”

![1_path](.\images\20181114\1_path.png)



Step2. If we direct config a exists vdi file ,it will raiser error like below

![2_show_error](.\images\20181114\2_show_error.png)



Step3.   We need to change the guid of VDI file

Open cmd.exe with administrator:

D:\Program Files\Oracle\VirtualBox>VBoxManage.exe internalcommands sethduuid D:\software\Linux\slave1_mysql_server.vdi

![3_sethduuid](.\images\20181114\3_sethduuid.png)



Step4.  Add vdi file to “Oracle VM”

![4_add_file.png](.\images\20181114\4_add_file.png)



Step5.   Start the new virtual machine “slave1_mysql_server”

![5_start_virtual_machine.png](.\images\20181114\5_start_virtual_machine.png)



Step6.  Login “slave1_mysql_server”

You will find the hostname still not changed, we need change the hostname now.

![6_still_not_change_hostname.png](.\images\20181114\6_still_not_change_hostname.png)

Step7. vi hosts

sudo hostname your-new-name

\# sudo hostname Slave1MySQL 



Modify hostname 

\# vi /etc/hostname

![7_vi_host.png](.\images\20181114\7_vi_host.png)



Restart network service or reboot

\1.      Replace the contents of /etc/hostname with the desired hostname (you can edit with sudo nano /etc/hostname)

\2.      In /etc/hosts, replace the entry next to 127.0.**1**.1 with the desired hostname (you can edit with sudo nano /etc/hosts)

\3.      Execute sudo service hostname restart; sudo service networking restart

\4.      Reboot

\#reboot