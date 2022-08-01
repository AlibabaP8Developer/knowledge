# MySQL备份与恢复

备份某—个数据库: mysqldump -uUsername -pPassword 数据库名 > xxx.sql

恢复 mysql -uUsername -pPassword 数据库名 < /root/完全备份文件名.sql

**MySQL自动备份脚本：**

https://github.com/lzhjavagithub/JavaStudy/blob/master/doc/linux/mysqlBackup_159.sh