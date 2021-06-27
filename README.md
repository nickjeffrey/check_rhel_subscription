# check_rhel_subscription
nagios check to verify Red Hat Enterprise Linux subscription status

# Requirements
perl, ssh, cron on nagios server
Internet access to query Red Hat subscription servers

# Configuration

The subscription-manager command requires root privileges to run.
Rather than give the nagios user and additional privileges, this script will run from the root crontab every 12 hours,
generating a /tmp/nagios.check_rhel_subscription.tmp file.  This file will be read by the low-privileged nagios user
when the check is run from nagios.
Create a crontab entry for the root user similar to:
```
    59 11,23 * * * /usr/local/nagios/libexec/check_rhel_subscription   #nagios helper script
```

This script is executed remotely on a monitored system by the NRPE or check_by_ssh methods available in nagios.  

If you are using the check_by_ssh method, you will need to add a section similar to the following to the services.cfg file on the nagios server.  
This assumes you already have ssh key pairs configured.
```
    define service {
       use                             generic-service
       hostgroup_name                  all_rhel
       service_description             RHEL subscription
       check_interval                  7200     ; only check every 12 hours
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
      normal_check_interval           7200     ; only check every 12 hours
      }
```

If you are using the NRPE method, you will also need a command definition similar to the following on each monitored host in the /usr/local/nagios/nrpe/nrpe.cfg file:
```
    command[check_rhel_subscription]=/usr/local/nagios/libexec/check_rhel_subscription
```
