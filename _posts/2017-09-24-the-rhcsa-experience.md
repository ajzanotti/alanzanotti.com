---
layout: post
title: The RHCSA Experience
description: Read about my recent RHCSA certification and the resources most beneficial to my success.
keywords: RHCSA,certification,RHEL7
date: 2017-09-24 18:02:35 -0400
excerpt: On 15 September 2017, I earned my Red Hat Certified System Administrator (RHCSA) certification from Red Hat. Overall, I found the process to be rewarding and I wanted to briefly discuss how I went about preparing for the exam, to hopefully benefit anyone else interested in getting certified.
---

On 15 September 2017, I earned my Red Hat Certified System Administrator (RHCSA)
certification from Red Hat. My Red Hat Certified Professional badge can be viewed
[here](https://www.redhat.com/rhtapps/certification/badge/verify/7TBPIJ6UN5YHPHTE6VOKW5QECQAEQU3CUPSQX2KSDXT6RW46LQ37ULE25V3KCXMMFRIX6PMBNQZGA4U5NQYTCNA62RUWOCM34WWBUYQ=).
Overall, I found the process to be rewarding and I wanted to briefly discuss how
I went about preparing for the exam, to hopefully benefit anyone else interested
in becoming certified.

Having never before taken a certification exam and without personally knowing any
Red Hat certified professionals, I had no frame of reference for what to expect
in the exam. In hindsight, I definitely over-studied, but the exam costs $400 and
I didn't want to spend that kind of money without doing everything I could to be
prepared. The first step for anybody interested in taking the RHCSA exam, or any
other exam for that matter, should be to review the exam [objectives](https://www.redhat.com/en/services/training/ex200-red-hat-certified-system-administrator-rhcsa-exam).
You should also get in the habit of periodically reviewing the exam objectives because
Red Hat reserves the right to make changes as they see fit, although I didn't notice
any changes between when I started preparing and when I took the exam.

Most of the objectives should be familiar to you from the start. If they're not,
it's worth reconsidering if now is the right time for you to be taking the exam.
Remember, studying is all well and good but there's no substitute for experience.
Practical use of knowledge is what will really cement it in your memory. I had been
doing Linux system administration for approximately 5 years when I began studying
and I was comfortable with most of the exam objectives. The only objectives that
were unfamiliar to me were those related to virtual machines. Of course, I have
experience with virtual machines, just not with Red Hat's KVM virtualization, so
I paid extra attention to preparing for those objectives.

Obviously, Red Hat Enterprise Linux 7 would be the ideal operating system to study
against, but I used CentOS 7 and never ran into any problems. Other distributions that
use Yum for package management and support KVM virtualization may be fine. However,
Fedora should be avoided because it's "bleeding edge" and may not be represent real
systems. To confirm that your system supports virtualization, look for either the
vmx (Intel) or svm (AMD) flag in the /proc/cpuinfo file. If neither flag is present,
ensure that virtualization is enabled in your BIOS. You should have enough memory,
CPU, and disk space to run at least two virtual machines.

{% highlight plaintext %}
grep -E '(vmx|svm)' /proc/cpuinfo
{% endhighlight %}

After you've reviewed the exam objectives and prepared a base study environment,
you can begin the real work. I did most of my studying from the following sources:

* Linux Bible, Ninth Edition (ISBN: 978-1-118-99987-5)
* RHCSA/RHCE Red Hat Linux Certification Study Guide, Seventh Edition (Exams EX200 & EX300) (ISBN: 978-0-07-184196-2)

The Linux Bible really takes you back to basics. Starting fresh can be beneficial,
and I did find a few interesting tidbits, but in the grand scheme of my studying
it didn't contribute much. The RHCSA/RHCE study guide was an excellent resource.
The authors, Michael Jang and Alessandro Orsaria, are both Red Hat certified and
in their book they look at each exam objective in an organized manner. Each chapter
has review questions and exercises to help you practice and remember what you've
read, but my favorite part of this book is that it has two RHCSA practice exams
in the back. If you use this book, I highly recommend taking both practice exams
and timing yourself. Plus, if you're considering earning a RHCE certification after
your RHCSA certification, this book has chapters and two practice exams for that
as well.

In addition to the books, I also read several manpages. That certainly exceeded
what was listed in the exam objectives, but doing so has already yielded benefits
for me in my work life, so I'm glad I took the time.

* bash
* systemd
* systemctl
* journalctl
* firewall-cmd
* yum

I can't provide any information about the exam itself, but some general advice would
be to make sure you're registering for the time and place that you think you are.
Also, it's always a good practice to arrive early. Finally, don't cause yourself
any more stress than you need to. If you've studied the exam objectives and practiced
the areas in which you're most deficient, you should do well.

