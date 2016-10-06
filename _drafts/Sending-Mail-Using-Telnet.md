---
layout: post
title: Sending Mail Using Telnet
author: Alan Zanotti
date: 2016-10-03 23:09:07 -0400
excerpt: <p>Learn how to send email over SMTP without the benefit of a mail user agent.</p>
---

Simple Mail Transfer Protocol (SMTP) is the official Internet standard for the
transmission of email that was first defined by RFC 821 in 1982 __CITATION__. It outlines a
process for the _transport_ of mail, which is not to be confused with mail _submission_
typically performed by users in a graphical mail client __CITATION__.

## A Simple Message

First, determine the mail server:

{% highlight plaintext %}
[ajzanotti@SMTPDemo ~]$ dig example.com MX +noall +answer

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.4 <<>> example.com MX +noall +answer
;; global options: +cmd
example.com.              1800    IN      MX      10 mail01.example.com.
example.com.              1800    IN      MX      20 mail02.example.com.
{% endhighlight %}

Above are the mail servers for the example.com domain. We'll connect to mail01.example.com because its preference (10) is the lowest (rfc 5321, page 70)

{% highlight plaintext linenos %}
[ajzanotti@SMTPDemo ~]$ telnet mail01.example.com 25
Trying 192.168.1.109...
Connected to mail01.example.com.
Escape character is '^]'.
220 mail01.example.com ESMTP Postfix
EHLO mail.alanzanotti.localhost
250-mail01.example.com
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
MAIL FROM: <noreply@alanzanotti.localhost>
250 2.1.0 Ok
RCPT TO: <vagrant@example.com>
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
Date: Tue, 04 Oct 2016 19:38:20 -0400
From: NoReply <noreply@alanzanotti.localhost>
To: Vagrant <vagrant@example.com>
Subject: Thought you should know...

Cats are awesome!
.
250 2.0.0 Ok: queued as C8D7E8C757
QUIT
221 2.0.0 Bye
Connection closed by foreign host.
{% endhighlight %}

## A More Complex Example

{% highlight plaintext linenos %}
Date: Tue, 04 Oct 2016 20:53:48 -0400
From: NoReply <noreply@alanzanotti.localhost>
To: Vagrant <vagrant@localhost>
Subject: Check out this cat video!
MIME-Version: 1.0
Content-Type: multipart/alternative; boundary=simpleboundary

--simpleboundary
Content-Type: text/plain; charset=UTF-8

You're going to love this cat!

https://www.youtube.com/watch?v=dQw4w9WgXcQ

--simpleboundary
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html>
  <head>
  </head>
  <body>
    <div style="color:#000; background-color:#fff; font-family:HelveticaNeue, Helvetica Neue, Helvetica, Arial, Lucida Grande, sans-serif;font-size:16px">
      <div>
        You're going to love <a href="https://www.youtube.com/watch?v=dQw4w9WgXcQ">this cat!</a>
      </div>
    </div>
  </body>
</html>

--simpleboundary--
.
{% endhighlight %}

## Sending Images

{% highlight plaintext linenos %}
Date: Wed, 05 Oct 2016 20:25:09 -0400
From: NoReply <noreply@alanzanotti.com>
To: Alan Zanotti <az9281@yahoo.com>
Subject: I want this cat!
MIME-Version: 1.0
Content-Type: multipart/alternative; boundary=outer

--outer
Content-Type: text/plain; charset=UTF-8

I really want this cat. She's so cute!

[Image of a cat]

--outer
Content-Type: multipart/mixed; boundary=inner

--inner
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html>
  <head>
  </head>
  <body>
    <div style="color:#000; background-color:#fff; font-family:HelveticaNeue, Helvetica Neue, Helvetica, Arial, Lucida Grande, sans-serif;font-size:16px">
      <p>I really want this cat. She's so cute!</p>
      <img src="cid:1234567890" />
    </div>
  </body>
</html>

--inner
Content-Type: image/jpeg
Content-Transfer-Encoding: base64
Content-Disposition: inline; filename=cortana-the-cat.jpg
Content-ID: <1234567890>

[BASE64-ENCODED IMAGE DATA]

--inner--
--outer--
.
{% endhighlight %}
