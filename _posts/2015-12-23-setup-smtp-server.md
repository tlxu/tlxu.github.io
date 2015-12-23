---
layout: post
title:  "Setup SMTP server to sendmail"
date:   2015-12-23
categories: Linux
---
$sudo apt-get install mailutils

Postfix was not set up.  Start with:
  cp /usr/share/postfix/main.cf.debian /etc/postfix/main.cf

If you need to make changes, edit
  /etc/postfix/main.cf (and others) as needed.
I added the following two parameters:
  myhostname=ubuntu-xu
  mail_name=tianli.xu@gmail.com

To view Postfix configuration values, see postconf(1).
After modifying main.cf, be sure to run '/etc/init.d/postfix reload'.

To start/restart postfix service with:
$sudo service postfix restart

An example to send an email(tianli.xu@yahoo.com) with an attachement:
$ echo "This is the body of the email" | mail -a 'From:"Tianli Xu"<tianli.xu@gmail.com>' -s "This is the subject line" -A ./demo.txt tianli.xu@yahoo.com


Once the SMTP is done, the following Python code can send all the .txt files in a directory as an email message:
{% highlight python %}
#!/usr/bin/env python3

import os
# Import smtplib for the actual sending function
import smtplib

# Here are the email package modules we'll need
from email.mime.image import MIMEImage
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

COMMASPACE = ', '

# Create the container (outer) email message.
msg = MIMEMultipart()
msg['Subject'] = 'Our family reunion'
# me == the sender's email address
# family = the list of all recipients' email addresses
me = 'tianli.xu@gmail.com'
tolist = 'tianli.xu@yahoo.com'

msg['From'] = me
#msg['To'] = COMMASPACE.join(tolist)
msg['To'] = tolist
msg.preamble = 'CTV news'

# Assume we know that the image files are all in PNG format
for file in os.listdir("./"):
    # Open the files in binary mode.  Let the MIMEImage class automatically
    # guess the specific image type.
    if file.endswith(".txt"):
        with open(file) as fp:
            txt = MIMEText(fp.read())
        txt.add_header('Content-Disposition', 'attachment', filename=file)
        msg.attach(txt)

# Send the email via our own SMTP server.
s = smtplib.SMTP('localhost')
s.send_message(msg)
s.quit()

{% endhighlight %}

Refer to [[https://docs.python.org/3/library/email-examples.html for more details.]]
