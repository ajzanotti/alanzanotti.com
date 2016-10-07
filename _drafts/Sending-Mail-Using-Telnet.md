---
layout: post
title: Sending Mail Using Telnet
author: Alan Zanotti
date: 2016-10-03 23:09:07 -0400
excerpt: <p>Simple Mail Transfer Protocol (SMTP) is the official Internet standard for the transmission of email. It outlines a process for the transport of mail and is one of the major underpinnings of the Internet that we take for granted. This post is going to remove the convenience of modern mail clients that you're used to and show you how to manually send mail over SMTP using telnet.</p>
---

Simple Mail Transfer Protocol (SMTP) is the official Internet standard for the
transmission of email that was first defined by RFC 821 in 1982 __CITATION__. It outlines a
process for the _transport_ of mail, which is not to be confused with the _submission_
of mail that is performed by a mail client __CITATION__. SMTP is one of the major
underpinnings of the Internet that we take for granted and that has enabled email to
become a part of daily life. This post is going to remove the convenience of
modern mail clients that you're used to and show you how to manually send mail over
SMTP using telnet.

In the examples below you're going to be emailing your best friend, John Doe. The
two of you bonded over your common, one true passion in life: cats. You used to
spend hours on the phone talking about how awesome cats are but then you learned
about the electronic mail and how you can send textual messages to each other,
and even pictures!

## A Simple Message

Before you can send an email to your best friend John, we need to know where the
message is going to be sent. Assuming that John's email is jdoe@example.com, we
do that by looking up the mail exchanger (MX) records for the domain part of his email
address. The MX record is a type of DNS resource record that maps a domain name
to a list of message transfer agents (MTAs) __CITATION__.

Dig is an excellent tool for querying DNS. The following command will query DNS
for any MX records associated with the example.com domain:

{% highlight plaintext %}
[azanotti@SMTPDemo ~]$ dig example.com MX +noall +answer

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.4 <<>> example.com MX +noall +answer
;; global options: +cmd
example.com.              1800    IN      MX      10 mail01.example.com.
example.com.              1800    IN      MX      20 mail02.example.com.
{% endhighlight %}

The output shows that there are two MTAs for the domain: mail01 and mail02. We're
going to telnet to mail01, not because it's first but because its preference (10)
is lower than mail02's preference (20) __(rfc 5321, page 70)__. Port 25 is the
standard SMTP port that we'll need to connect to in order to send our first message.
Since you're still new to electronic mail we'll start with a basic text only message
that says "Cats are awesome!"

{% highlight plaintext linenos %}
[azanotti@SMTPDemo ~]$ telnet mail01.example.com 25
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
MAIL FROM: <yourName@domain.com>
250 2.1.0 Ok
RCPT TO: <jdoe@example.com>
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
Date: Tue, 04 Oct 2016 19:38:20 -0400
From: Your Name <yourName@domain.com>
To: John Doe <jdoe@example.com>
Subject: Thought you should know...

Cats are awesome!
.
250 2.0.0 Ok: queued as C8D7E8C757
QUIT
221 2.0.0 Bye
Connection closed by foreign host.
{% endhighlight %}

The real action starts on Line 5 with the 220 response from the server. In SMTP
all server responses start with a response code. The 220 code is the initial greeting
that announces the server is opening its part of the connection __rfc 5321, page 47__.

The first SMTP command that you send is the extended hello or EHLO, which is used
to identify the client to the server. The EHLO argument is the fully qualified domain
name of the client or its IP address if it doesn't have one __rfc 5321, page 32__.
The server then responds with a list of available SMTP extensions that you may use.

Following the EHLO, you send the MAIL and RCPT commands to identify the address
sending the message and the address receiving the message __rfc 5321, pages 34-35__.
Multiple recipients can be specified by issuing the RCPT command multiple times.

The DATA command signifies the start of the message __CITATION__. A message
starts with a header section on Lines 21 - 24 and is separated from the body content
by an empty line __rfc 5322, page 7__. There are many headers available to use but
we don't need anymore than what we currently have __rfc 5322, page 19__. The date requires
a certain format and can be quickly generated with the following command:

{% highlight plaintext %}
[azanotti@SMTPDemo ~]$ date -R
Tue, 04 Oct 2016 19:38:20 -0400
{% endhighlight %}

The body of a message can be anything you want provided that the text is US-ASCII __rfc5322, page 9__.
It's possible to send messages in character sets other than US-ASCII but that's beyond
the scope of this post and you're still new to electronic mail.

To signal to the server that you're ready to send, Line 27 terminates the message
by sending a period on a line by itself __rfc 5321, page 36__. If the server accepts
responsibility for delivering your message it will respond with a 250 code and your
email is on its way to John.


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
