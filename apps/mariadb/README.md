# To see whether unencrypted SQL connection is allowed through egress GW to the Database

Findings:
when Mesh outboundTrafficPolicy is set to REGISTRY_ONLY,
- app --> egress_gw --> External_svc works for Google HTTPS connection but not for unencrypted mariadb TCP/3306 connection (serviceentry, virtualservice and egress-GW are configured) - This behaviour is similar to consul 
- app --> External_svc works for mariadb TCP/3306 connection (serviceentry is configured, no egress GW)
    - Authorization policies not working 

when Mesh outboundTrafficPolicy is set to ALLOW_ANY,
- app --> External_svc works for mariadb TCP/3306 connection even without a serviceentry is not configured (without egress GW)

-----------------------------------------------------------

Test Commands:

mysql -h database-1.c7i8akksyk5b.eu-central-1.rds.amazonaws.com -P 3306 -u admin -p
admin1234

while true; do mysql -h database-1.c7i8akksyk5b.eu-central-1.rds.amazonaws.com -P 3306; sleep 1; done