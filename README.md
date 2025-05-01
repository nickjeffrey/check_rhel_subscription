# check_rhel_subscription
nagios check to verify Red Hat Enterprise Linux subscription status

# Requirements
perl, ssh on nagios server

Access to root crontab on monitored RHEL host

Internet access from RHEL host to query Red Hat subscription servers

# Configuration

The subscription-manager command requires root privileges to run.
Rather than give the nagios user and additional privileges, this script will run daily from the root crontab,
generating a /tmp/nagios.check_rhel_subscription.tmp file.  This file will be read by the low-privileged nagios user
when the check is run from nagios.
Create a crontab entry for the root user similar to:
```
    59 23 * * * /usr/local/nagios/libexec/check_rhel_subscription   >/dev/null 2>&1 #nagios helper script
```

This script is executed remotely on a monitored system by the NRPE or check_by_ssh methods available in nagios.  

If you are using the check_by_ssh method, you will need to add a section similar to the following to the services.cfg file on the nagios server.  
This assumes you already have ssh key pairs configured.
```
    define service {
       use                             generic-service
       hostgroup_name                  all_rhel
       service_description             RHEL subscription
       check_command                   check_by_ssh!/usr/local/nagios/libexec/check_rhel_subscription
       }
```

Alternatively, if you are using the check_nrpe method, you will need to add a section similar to the following to the services.cfg file on the nagios server.  
This assumes you already have ssh key pairs configured.
```
   define service{
      use                             generic-service
      hostgroup_name                  all_rhel
      service_description             RHEL subscription
      check_command                   check_nrpe!check_rhel_subscription -t 30
            }
```

If you are using the NRPE method, you will also need a command definition similar to the following on each monitored host in the /usr/local/nagios/nrpe/nrpe.cfg file:
```
    command[check_rhel_subscription]=/usr/local/nagios/libexec/check_rhel_subscription
```

# Sample output
```
RHEL subscription OK - RHEL subscription is using Simple Content Access mode | days_until_expiry=0;;;;
```
