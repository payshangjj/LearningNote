mysqld --user=root --wsrep-recover

mysqld_safe

修改文件里safe_to_bootstrap to 1
[root@master01 ambari-server]# cat /data/data01/mariadb/data/grastate.dat
# GALERA saved state
version: 2.1
uuid:    9d7a70d1-53c3-11e9-8452-e28704e2875d
seqno:   -1
safe_to_bootstrap: 0 

systemctl start mariadb
ambari-server start

galera_new_cluster

systemctl start ambari




参考:

[通过 resultful api 启动ambari service](https://community.hortonworks.com/content/supportkb/49134/how-to-stop-start-a-ambari-service-component-using.html)

https://mariadb.com/kb/en/library/galera-cluster-system-variables/

https://stackoverflow.com/questions/37212127/mariadb-gcomm-backend-connection-failed-110

https://mariadb.com/kb/zh-cn/getting-started-with-mariadb-galera-cluster/

http://galeracluster.com/documentation-webpages/restartingcluster.html
