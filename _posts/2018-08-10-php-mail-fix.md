---
layout: post
title: "Send email using php mail(...) function"
description: "How to send an email to gmail... using built-in mail() function (working on Ubuntu 16.04)"
categories: [coding, mail, php]
tags: [tutorial]
redirect_from:
  - /2018/08/10/
---
I spend on day to figure out why I can't send an email to gmail or something else even function `mail($to, $subject, $message)` return `true`.

And here is the way how to fix it:

# The installation process

> sudo apt-get install ssmtp

#  Then edit the /etc/ssmtp/ssmtp.conf file with this information:

~~~
mailhub=smtp.gmail.com:587
UseSTARTTLS=YES
AuthUser=<YOUR-EMAIL>@gmail.com
AuthPass=<YOUR-PASSWORD>
~~~

#  Edit php.ini (usually in /etc/php5/cli/php.ini)

~~~
sendmail_path = /usr/sbin/sendmail -t
~~~

#  Restart server

> service apache2 restart

# Coding

~~~ php
<?php 
    $to      = 'someone@gmail.com';
    $subject = 'the subject';
    $message = 'hello';

    $res = mail($to, $subject, $message);
    if($res) {
        echo 'Send email success';
    }
    else {
        echo 'Send email fail';
    }
?>
~~~