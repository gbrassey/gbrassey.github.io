---
layout: post
title: A Brief Description of the SPF record
---

A client recently came to us with problems receiving emailed form submissions from their website. We did a little testing and realized that indeed the emails were being sent by the server but something was stopping them during transit. In our research, we had to dig deeper into SPF records.

An SPF (Sender Policy Framework) record, validates an IP address as permitted to send emails from that Domain. Once propagated, the DNS record serves as a list of verified senders, which the recipient of an email can use to check against.

The SPF record was introduced because the ubiquitous SMTP (Simple Mail Transfer Protocol) allows a computer to send email claiming to be from any domain. Spam and phishing emailers use this to send email purporting to be from a trusted source.

With an SPF record, a receivers spam filter is more likely to accept the message as it is coming from a validated source. It also discourages potential spammers from using your domain as a sending address, knowing that domains with SPF records are more likely to be rejected by recipient spam filters, if the senders IP does not match one associated with the record.

Once an SPF record is created, it is important to ensure that all computers that will be sending emails are included, as the presence of the record indicates that any email not declared is dubious. This is the problem we ran into, where an SPF record was added to the DNS zone, but did not include the server hosting the website in it’s list of IP addresses.

Here is an example SPF TXT record:

	example.com TXT “v=spf1 ip4:192.0.2.0/24 ip4:198.51.100.123 a ~all”

This post originally appeared on [The Mechanism's Blog](http://www.themechanism.com/voice/2014/10/17/a-brief-description-of-the-spf-record/ "A Brief Description of the SPF record").