---
layout: post
title: Packer Templates v2.0.0 Released
description: The v2.0.0 release of ajzanotti/packer-templates adds its first Ubuntu template and an amazon-import option for CentOS 7.
keywords: Packer,Vagrant,Ubuntu,CentOS,packer-templates,amazon-import
date: 2017-01-06 19:27:53 -0500
excerpt: The v2.0.0 release of ajzanotti/packer-templates has a lot to offer. In addition to a complete reorganization that better facilitates the use of the project, v2.0.0 adds the first Ubuntu template, as well as an amazon-import option for CentOS 7.
---

The v2.0.0 release of [ajzanotti/packer-templates](https://github.com/ajzanotti/packer-templates)
has a lot to offer. In addition to a complete reorganization that better facilitates
the use of the project, v2.0.0 adds the first Ubuntu template, as well as an amazon-import
option for CentOS 7.

## CentOS 7 Amazon Import

The CentOS 7 templates are comprised of three pieces. The first creates an OVA
that can then be imported into the vagrant and amazon-import templates for further
provisioning and post-processing.

To create the OVA

{% highlight plaintext %}
packer build -var-file=variables/common.json centos/7/ova-base.json
{% endhighlight %}

Prior to building the amazon-import template, the [vmimport IAM role](http://docs.aws.amazon.com/vm-import/latest/userguide/import-vm-image.html)
needs to be configured in AWS and any customizations to the [cloud-init](https://cloudinit.readthedocs.io/en/latest/)
config should be applied.

{% highlight plaintext %}
packer build -var-file=variables/aws.json -var "ova_path=path/to/OVA" centos/7/amazon-import.json
{% endhighlight %}

The upload to Amazon S3 and subsequent conversion from OVA to AMI can take a long
time, but the end result is a custom image in AWS that's tailored to your needs.


## Ubuntu Xenial

The command to build the Ubuntu Xenial Vagrant box is typical of other Vagrant box
builds

{% highlight plaintext %}
packer build -var-file=variables/common.json ubuntu-xenial/vagrant.json
{% endhighlight %}
