---
layout: post
title: Sending Mail Using Telnet
author: Alan Zanotti
date: 2016-10-03 23:09:07 -0400
excerpt: <p>Simple Mail Transfer Protocol (SMTP) is the official Internet standard for the transmission of email. SMTP is one of the major underpinnings of the Internet that we take for granted and that has enabled email to become a part of daily life. This post is going to remove the convenience of modern mail clients that you're used to and show you how to manually send mail over SMTP using telnet.</p>
---

Simple Mail Transfer Protocol (SMTP) is the official Internet standard for the transmission
of email that was first defined by RFC 821 in 1982. SMTP is one of the major underpinnings
of the Internet that we take for granted and that has enabled email to become a part
of daily life. This post is going to remove the convenience of modern mail clients
that you're used to and show you how to manually send mail over SMTP using telnet.

## A Simple Message

Before you can send an email to your best friend John, we need to know where the
message is going to be sent. Assuming that John's email address is jdoe@example.com,
we can do that by looking up the mail exchanger (MX) records for the domain part
of his email address. The MX record is a type of DNS resource record that maps a
domain name to a list of message transfer agents (MTAs).

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
is lower than mail02's preference (20). Port 25 is the standard SMTP port that you'll
need to connect to in order to send your message.

Once you've initiated a telnet session to mail01, you cannot begin to send a message
until you receive an acknowledgement from the server. In SMTP all server responses
start with a three-digit response code. The 220 code is the initial greeting that
announces the server is opening its part of the connection.

The first SMTP command that you send is the extended hello or EHLO, which is used
to identify the client to the server. The EHLO argument is the fully qualified domain
name of the client or its IP address if it doesn't have one. The server then responds
with a 250 success response code and a list of available SMTP extensions that you
may use.

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
sending the message and the address receiving the message. Multiple recipients can
be specified by issuing the RCPT command multiple times.

{% highlight plaintext %}
MAIL FROM: <yourAddress@domain.com>
250 2.1.0 Ok
RCPT TO: <jdoe@example.com>
250 2.1.5 Ok
{% endhighlight %}

The DATA command signifies the start of the message. A message starts with a header
section that is separated from the body content by an empty line. There are many
headers available but you don't need any more than what is currently in use.

To signal to the server that you're ready to send, the message must be terminated
by sending a period on a line by itself. If the server accepts responsibility for
delivering your message it will respond with a 250 code and your email is on its
way to John.

{% highlight plaintext %}
DATA
354 End data with <CR><LF>.<CR><LF>
Date: Wed, 12 Oct 2016 21:22:10 -0400
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
be anything you want provided that the text is US-ASCII. It's possible to send messages
in character sets other than US-ASCII but that's beyond the scope of this post.

Also, while the telnet command here was split into three different pieces in order
to facilitate the learning process, these three pieces are actually all part of
sending one message in one telnet session.


## A More Complex Message

The example from the previous section is as simple as it gets. In most cases, modern
mail clients are capable of handling more types of content and presentation styles
than clients could in the 80s when SMTP was first introduced. There are still mail
clients that support plain text only, and how certain clients display a message
can be subject to configuration and policy.

Fortunately, some very smart people created Multipurpose Internet Mail Extensions (MIME)
to allow for a variety of content and presentation styles in email. Included among
the supported content types are HTML text, images, and audio. It even allows us
to send alternative versions of a message and permits mail clients to choose which
of those versions to present to a user.

Suppose you wanted to email John a link to a funny cat video you found YouTube.
You could do it in exactly the same way as in the previous section or you could
be a little fancy.

{% highlight plaintext %}
Date: Wed, 12 Oct 2016 21:27:10 -0400
From: Your Name <yourAddress@domain.com>
To: John Doe <jdoe@example.com>
Subject: Check out this cat video!
MIME-Version: 1.0
Content-Type: multipart/alternative; boundary=simpleboundary

--simpleboundary
Content-Type: text/plain

You're going to love this cat!

https://www.youtube.com/watch?v=dQw4w9WgXcQ

--simpleboundary
Content-Type: text/html

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

The first thing to note is that you've added two new mail headers. MIME-Version
indicates that a message is MIME-formatted and is always set to 1.0. Content-Type
varies and in this case states that there will be alternative versions of the same
message and that those alternatives will be separated by the text "simpleboundary".

Each alternative will begin with the boundary text prepended with two dashes
(&#8208;&#8208;simpleboundary), followed by a header section for that alternative,
and then the message body. The final alternative will be followed by
&#8208;&#8208;simpleboundary&#8208;&#8208; to terminate the list of alternatives.
When using the multipart/alternative content type, you must always list your alternatives
in order of increasing "richness". So, your most basic option is always first and
your fanciest option is last. Clients will display the richest one that they can
support.

This particular message gives two options: the plain text message like the first
example (text/plain) and a richer HTML version with a hyperlink (text/html). Note
that the message must still be terminated by a period on a line by itself before
it can be delivered via SMTP.


## Even More Complex Message

Building on top of what we've just learned with MIME and the multipart/alternative
content type, you have the basic knowledge you need to spice up your emails to John
with cute cat pictures by nesting alternatives together. Each instance of multipart/alternative
will need its own unique boundary text and both will need to be terminated separately.

{% highlight plaintext %}
Date: Wed, 12 Oct 2016 21:31:06 -0400
From: Your Name <yourAddress@domain.com>
To: John Doe <jdoe@example.com>
Subject: I want this cat!
MIME-Version: 1.0
Content-Type: multipart/alternative; boundary=outer

--outer
Content-Type: text/plain

I really want this cat. She's so cute!

[Image of a cat]

--outer
Content-Type: multipart/mixed; boundary=inner

--inner
Content-Type: text/html

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

/9j/4AAQSkZJRgABAgEASABIAAD/4RWJRXhpZgAASUkqAAgAAAAHABIBAwABAAAAAQAAABoBBQAB
AAAAYgAAABsBBQABAAAAagAAACgBAwABAAAAAgAAADEBAgAlAAAAcgAAADIBAgAUAAAAlwAAAGmH
...

--inner--
--outer--
.
{% endhighlight %}

The example above introduces the image/jpeg content type and with it three new headers:
Content-Transfer-Encoding, Content-Disposition, and Content-ID.

The Content-Transfer-Encoding header specifies the type of encoding transformation
applied to the body and thus the decoding operation necessary to restore it to its
original form. Without extensions, SMTP restricts messages to 7bit US-ASCII characters.
In this instance, the encoding is base64, meaning that the image was converted to
7bit US-ASCII text for easy transmission. To base64 encode a file on the command
line:

{% highlight plaintext %}
[azanotti@SMTPDemo ~]$ base64 someImage.jpg
{% endhighlight %}

The Content-Disposition header indicates the desired presentation semantics of a
message component. A Content-Disposition of inline means that a component should
be displayed automatically upon display of the message. Think of every time you
read an email with an image directly in the body instead of as an attachment, which
happens to be another Content-Disposition type.

The Content-ID is a label for uniquely identifying a message body part. Though optional,
by including it for the image you can then easily reference it by cid in the image
tag of your text/html message body part.


# Problems and Pitfalls

* Many ISPs block port 25, so trying the above examples from your home computer might
  not work.
* Don't expect to be able to send mail directly to a Gmail address unless you're
  sending from a server that has a good sending reputation.
* The correct order for SMTP commands is always EHLO, MAIL, RCPT, and then DATA.
  Any deviation from this will result in a "503 bad sequence of commands" response
  from the server.
* The date mail header expects a certain format and can be quickly generated from
  the command line:

{% highlight plaintext %}
[azanotti@SMTPDemo ~]$ date -R
Wed, 12 Oct 2016 20:36:53 -0400
{% endhighlight %}


# Related Documents

* Simple Mail Transfer Protocol ([RFC 5321](https://tools.ietf.org/html/rfc5321))
* Internet Message Format ([RFC 5322](https://tools.ietf.org/html/rfc5322))
* MIME Part One: Format of Internet Message Bodies ([RFC 2045](https://tools.ietf.org/html/rfc2045))
* MIME Part Two: Media Types ([RFC 2046](https://tools.ietf.org/html/rfc2046))
* MIME Part Five: Conformance Criteria and Examples ([RFC 2049](https://tools.ietf.org/html/rfc2049))
* The Content-Disposition Header Field ([RFC 2183](https://tools.ietf.org/html/rfc2183))
* Content-ID and Message-ID Uniform Resource Locators ([RFC 2392](https://tools.ietf.org/html/rfc2392))
