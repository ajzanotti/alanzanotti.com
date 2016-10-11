---
layout: post
title: Sending Mail Using Telnet
author: Alan Zanotti
date: 2016-10-03 23:09:07 -0400
excerpt: <p>Simple Mail Transfer Protocol (SMTP) is the official Internet standard for the transmission of email. It outlines a process for the transport of mail and is one of the major underpinnings of the Internet that we take for granted. This post is going to remove the convenience of modern mail clients that you're used to and show you how to manually send mail over SMTP using telnet.</p>
---

Simple Mail Transfer Protocol (SMTP) is the official Internet standard for the
transmission of email that was first defined by RFC 821 in 1982 __CITATION__. SMTP
is one of the major underpinnings of the Internet that we take for granted and that
has enabled email to become a part of daily life. This post is going to remove the
convenience of modern mail clients that you're used to and show you how to manually
send mail over SMTP using telnet.

## A Simple Message

Before you can send an email to your best friend John, we need to know where the
message is going to be sent. Assuming that John's email is jdoe@example.com, we can
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

The output shows that there are two MTAs for the domain: mail01 and mail02. You're
going to telnet to mail01, not because it's first but because its preference (10)
is lower than mail02's preference (20) __(rfc 5321, page 70)__. Port 25 is the
standard SMTP port that you'll need to connect to in order to send your message.

Once you've initiated a telnet session to mail01, you cannot begin to send a message
until you receive an acknowledgement from the server. In SMTP all server responses start
with a three-digit response code __CITATION__. The 220 code is the initial greeting that announces
the server is opening its part of the connection __rfc 5321, page 47__.

The first SMTP command that you send is the extended hello or EHLO, which is used
to identify the client to the server. The EHLO argument is the fully qualified domain
name of the client or its IP address if it doesn't have one __rfc 5321, page 32__.
The server then responds with a 250 success response code and a list of available
SMTP extensions that you may use.

{% highlight plaintext %}
[azanotti@SMTPDemo ~]$ telnet mail01.example.com 25
Trying 192.168.2.100...
Connected to mail01.example.com.
Escape character is '^]'.
220 mail01.example.com ESMTP Postfix
EHLO 192.168.2.50
250-mail01.example.com
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
{% endhighlight %}

Following the EHLO, you send the MAIL and RCPT commands to identify the address
sending the message and the address receiving the message __rfc 5321, pages 34-35__.
Multiple recipients can be specified by issuing the RCPT command multiple times.

{% highlight plaintext %}
MAIL FROM: <yourAddress@domain.com>
250 2.1.0 Ok
RCPT TO: <jdoe@example.com>
250 2.1.5 Ok
{% endhighlight %}

The DATA command signifies the start of the message __CITATION__. A message
starts with a header section that is separated from the body content by an empty
line __rfc 5322, page 7__. There are many headers available but you don't need anymore
than what you currently have in use __rfc 5322, page 19__.

To signal to the server that you're ready to send, the message must be terminated
by sending a period on a line by itself __rfc 5321, page 36__. If the server accepts
responsibility for delivering your message it will respond with a 250 code and your
email is on its way to John.

{% highlight plaintext %}
DATA
354 End data with <CR><LF>.<CR><LF>
Date: Tue, 04 Oct 2016 19:38:20 -0400
From: Your Name <yourAddress@domain.com>
To: John Doe <jdoe@example.com>
Subject: Thought you should know...

Cats are awesome!
.
250 2.0.0 Ok: queued as C8D7E8C757
QUIT
221 2.0.0 Bye
Connection closed by foreign host.
{% endhighlight %}

This simple message just says "Cats are awesome!", but the body of a message can
be anything you want provided that the text is US-ASCII __rfc5322, page 9__.
It's possible to send messages in character sets other than US-ASCII but that's beyond
the scope of this post.

Also, while the telnet command here was split into three different pieces in order
to facilitate the learning process, these three pieces are actually all part of
sending one message in one telnet session.


## A More Complex Example

The example from the previous section is as simple as it gets, and it's boring too.
In most cases, modern mail clients are capable of handling more types of content
and presentation styles than clients in the 80s when SMTP was first introduced. There
are still mail clients that support plain text only and how certain clients display
a message can be subject to configuration and policy.

Fortunately, some very smart people have addressed these problems for us with Multipurpose
Internet Mail Extensions (MIME) __CITATION__. MIME offers us a variety of different
content types to send such as text, images, and audio. It even allows us to send
alternative versions of a message and permits mail clients to choose which of those
versions to present to a user.

Suppose you wanted to email John a link to a funny cat video you found youtube.
You could do it in exactly the same way as in the previous section or you could be
a little fancy.

{% highlight plaintext %}
Date: Tue, 04 Oct 2016 20:53:48 -0400
From: Your Name <yourAddress@domain.com>
To: John Doe <jdoe@example.com>
Subject: Check out this cat video!
MIME-Version: 1.0
Content-Type: multipart/alternative; boundary=simpleboundary

--simpleboundary
Content-Type: text/plain; charset=US-ASCII

You're going to love this cat!

https://www.youtube.com/watch?v=dQw4w9WgXcQ

--simpleboundary
Content-Type: text/html; charset=US-ASCII

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

The first thing to note is that you've added two new mail headers. MIME-Version is
always set to 1.0; simple enough. Content-Type varies and in this case states that
there will be alternative versions of the same message and that those alternatives will
be separated by the text "simpleboundary".

Each alternative will begin with the boundary text prepended with two dashes (&#8208;&#8208;simpleboundary),
followed by a header section for that alternative, and then the message body. The final
alternative will be followed by &#8208;&#8208;simpleboundary&#8208;&#8208; to terminate the list of alternatives.
When using the multipart/alternative content type, always list your alternatives in
order of increasing "richness". So, your most basic option is always first and your
fanciest option is last. Clients will display the richest one that they support.

This particular message gives two options: the plain text message like the first example (text/plain)
and an richer HTML version with a hyperlink (text/html). Note that the message must
still be terminated by a period on a line by itself.

## Even More Complex

{% highlight plaintext %}
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


# Problems and Pitfalls
