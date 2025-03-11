# Wazuh Email Alerts Configuration Guide

This guide covers the process of configuring Wazuh to send email alerts using Postfix as the mail transfer agent.

## Prerequisites

- A working Wazuh installation
- SMTP credentials (e.g., Gmail account)
- Root access to the Wazuh manager server

## Step 1: Install Required Dependencies

First, install Postfix and other required packages:

```bash
# Update and install Postfix
yum update && yum install postfix 

# Install SASL authentication libraries
yum install cyrus-sasl-plain

# Install mail utilities
dnf provides *bin/mailx
dnf install s-nail
```

## Step 2: Configure Postfix

Edit the Postfix configuration file:

```bash
vi /etc/postfix/main.cf
```

Add the following lines to the configuration file:

```
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
#smtp_tls_CAfile = /etc/ssl/certs/thawte_Primary_Root_CA.pem
smtp_use_tls = yes
compatibility_level = 2
```

> Note: For Gmail, you'll need to use an App Password if you have 2-factor authentication enabled.

## Step 3: Configure SMTP Authentication

Create and secure the password file:

```bash
# Create the password file (replace with your actual email and password)
echo "[smtp.gmail.com]:587 wazuhtest@testserver.com:mypassword" > /etc/postfix/sasl_passwd

# Generate hash database file
postmap /etc/postfix/sasl_passwd

# Set secure permissions
chmod 400 /etc/postfix/sasl_passwd
chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db

# Restart Postfix service
systemctl restart postfix
```

## Step 4: Test Email Delivery

Send a test email to verify your configuration:

```bash
echo "Hi! We are testing Postfix!" | mail -s "Test Postfix" destinationmail@testserver1.com
```

Check your destination email inbox to confirm receipt of the test email.

## Step 5: Configure Wazuh for Email Alerts

Edit the Wazuh manager configuration file:

```bash
vi /var/ossec/etc/ossec.conf
```

Add or modify the following sections in the configuration file:

```xml
<ossec_config>
  <global>
    <jsonout_output>yes</jsonout_output>
    <alerts_log>yes</alerts_log>
    <logall>no</logall>
    <logall_json>no</logall_json>
    <email_notification>yes</email_notification>
    <smtp_server>localhost</smtp_server>
    <email_from>inndataanalytics@gmail.com</email_from>
    <email_to>inndataanalytics@gmail.com</email_to>
    <email_maxperhour>12</email_maxperhour>
    <email_log_source>alerts.log</email_log_source>
    <agents_disconnection_time>10m</agents_disconnection_time>
    <agents_disconnection_alert_time>0</agents_disconnection_alert_time>
    <update_check>yes</update_check>
  </global>
  <alerts>
    <log_alert_level>3</log_alert_level>
    <email_alert_level>3</email_alert_level>
  </alerts>
  <!-- Other configuration sections remain unchanged -->
</ossec_config>
```

## Step 6: Restart Wazuh Manager

After making changes to the configuration, restart the Wazuh manager service:

```bash
systemctl restart wazuh-manager
```

## Configuration Parameters Explained

In the Wazuh configuration:

- `<email_notification>yes</email_notification>`: Enables email notifications
- `<smtp_server>localhost</smtp_server>`: Uses the local Postfix server
- `<email_from>`: Sender email address
- `<email_to>`: Recipient email address
- `<email_maxperhour>12</email_maxperhour>`: Limits emails to 12 per hour
- `<email_log_source>alerts.log</email_log_source>`: Source of alert information
- `<log_alert_level>3</log_alert_level>`: Minimum level for logging alerts
- `<email_alert_level>3</email_alert_level>`: Minimum level for email alerts

## Alert Levels

- 1: Low importance
- 2: Medium importance
- 3: High importance (default threshold)
- 4-15: User-defined levels of increasing importance

## Troubleshooting

### Common Issues

1. **Emails not being sent**:
   - Check Postfix logs: `tail -f /var/log/maillog`
   - Verify Wazuh manager logs: `tail -f /var/ossec/logs/ossec.log`

2. **Authentication failures**:
   - For Gmail, ensure you're using an App Password
   - Check for correct username/password in sasl_passwd file

3. **Configuration errors**:
   - Run `wazuh-logtest` to verify configuration
   - Check for XML syntax errors in ossec.conf

### Testing Alert Emails

To test if the alert system is working correctly, you can trigger a test alert:

```bash
/var/ossec/bin/ossec-control enable debug
/var/ossec/bin/ossec-testrule -D -c /var/ossec/etc/ossec.conf
```

## Additional Resources

- [Wazuh Documentation - Email Alerts](https://documentation.wazuh.com/current/user-manual/manager/manual-email-report/index.html)
- [Postfix Documentation](http://www.postfix.org/documentation.html)
