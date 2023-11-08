# To see whether unencrypted SQL connection is allowed through egress GW to the Database

Mesh outboundTrafficPolicy is set to REGISTRY_ONLY

Findings:
- app --> egress_gw --> External_svc works for Google HTTPS connection but not for unencrypted mariadb TCP/3306 connection
- app --> External_svc works for mariadb TCP/3306 connection

-----------------------------------------------------------

Test Commands:

mysql -h database-1.clawixyahyke.eu-central-1.rds.amazonaws.com -u admin -p -e 'SHOW databases;'
mysql -h database-1.clawixyahyke.eu-central-1.rds.amazonaws.com -P 3306 -u admin -p
admin123
