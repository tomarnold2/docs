---
layout: page
weight: 0
title: Postfix
group: mail-servers
seo:
  title: Integrate SendGrid with Postfix - SendGrid Documentation | SendGrid
  description: Setup SendGrid as a Postfix relay host for better deliverability and advanced statistics on your email.
navigation:
  show: true
---

Updating the Postfix configuration to use SendGrid as a relay host is easy. For more advanced configuration scenarios, you'll need to refer to the Postfix documentation.

<call-out type="warning">

Some implementations of Postfix only allow passwords to contain letters and numbers, meaning you may need to ensure your SendGrid password is only alphanumeric.

</call-out>

Find your Postfix config file, typically **/etc/postfix/main.cf**, and add the following:

```
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
smtp_tls_security_level = encrypt
header_size_limit = 4096000
relayhost = [smtp.sendgrid.net]:587
```

<call-out>

We highly recommend using an API key as your credential for any service that supports it. API keys are both more secure and easier to work with.

</call-out>

Now you need to specify your credentials (use `apikey` as username and an API Key as password) in the separate file **/etc/postfix/sasl_passwd** (you'll likely need to create it):

```
[smtp.sendgrid.net]:587 apikey:yourSendGridApiKey
```

Next, make sure the file has restricted read and write access only for root, and use the `postmap` command to update Postfix's hashtables to use this new file:

```bash
$ sudo chmod 600 /etc/postfix/sasl_passwd
$ sudo postmap /etc/postfix/sasl_passwd
```

Finally, restart Postfix:
```bash
$ sudo systemctl restart postfix
```


<call-out>

If you are getting ***no mechanism available*** error messages, it generally indicates that you are missing some SASL authentication libraries.

</call-out>

Install the missing module dependency using apt-get (i.e., Debian, Ubuntu):

```bash
$ apt-get install libsasl2-modules
```

 Or using a yum (i.e., RedHat, Fedora, CentOS):

```bash
$ yum install cyrus-sasl-plain
```

## Troubleshooting
 	
If port 587 is not working for you, please try 2525 in your postfix config. You may also need to edit /etc/postfix/master.cf to remove # from `tlsmgr unix - - n 1000? 1 tlsmgr`. 

For other potential errors, please navigate to the default maillog file on your server for debugging purposes. For a CentOS server, the file is in path /var/log/maillog by default. 

This file not only reports that the server was unable to use SendGrid to send email but also serves as an excellent debugging tool for new users that catalogues server parameters.
