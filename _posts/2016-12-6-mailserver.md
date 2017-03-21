---
layout: post
title:  "Mailserver"
date:   2016-12-6 02:59:59
categories: mailer
---

hello guys, 
Today we will see how to create a basic mailserver, please follow my steps carefully, otherwise it will be hard to maintain your mailserver.

<h1>Prerequisite:</h1>
* ssh server access
* basic postfix config.
* domain mapping(DNS)

<h1>Requirements:</h1>
* one digitalocean droplet
* one domain name for your mailserver
* postfix

Assuming that DNS mapping is in place,
for postfix basic configuration files follow my <a href="https://github.com/sumitgupta0001/mailserver_basic"> mailserver github repository</a>

Lets start with an example, 
Go to gmail, open any email from promotions or any emailer whom you trust like(paytm or amazon), then do show original mail, under that section you will find few entries like 

{% highlight ruby %}

Message ID  <ff4b18069f3fa23a5676384ff.9d49e6aa58.20161221060620.e87052c3c2.33652e63@mail3.suw11.mcdlv.net>
Created at: Wed, Dec 21, 2016 at 11:37 AM (Delivered after 198 seconds)
From:   Shuttl <info@shuttlemails.com>Using MailChimp Mailer - **CIDe87052c3c29d49e6aa58**
To: xyz@gmail.com
Subject:    Get Rs. 1000 PayTM cash for each person you refer 😀
SPF:    PASS with IP 198.2.xxx.x Learn more
DKIM:   PASS with domain mail3.suw11.mcdlv.net Learn more

{% endhighlight %}

As you can see this section contains the basic info of the sender, reciever and it also contains <code>SPF</code> and <code>DKIM</code>, so what are these?

So, these are basically the records that are used to identify your server ,domain and to confirm that you are not spammer. 

We will see working and enabling of each one by one.

<h2>SPF(SENDER POLICY FRAMEWORK)</h2>

We recommend that you create a Sender Policy Framework (SPF) record for your domain. An SPF record is a type of Domain Name Service (DNS) record that identifies which mail servers are permitted to send email on behalf of your domain.

The purpose of an SPF record is to prevent spammers from sending messages with forged From addresses at your domain. Recipients can refer to the SPF record to determine whether a message purporting to be from your domain comes from an authorized mail server.

For example, suppose that your domain example.com uses Gmail. You create an SPF record that identifies the G Suite mail servers as the authorized mail servers for your domain. When a recipient's mail server receives a message from user@example.com, it can check the SPF record for example.com to determine whether it is a valid message. If the message comes from a server other than the G Suite mail servers listed in the SPF record, the recipient's mail server can reject it as spam.

If your domain does not have an SPF record, some recipient domains may reject messages from your users because they cannot validate that the messages come from an authorized mail server.


Create an SPF record for your domain

Log in to the administrative console for your domain.
(in my case its bigrock.com )
Locate the page where you can update the DNS records.
(in my case Account Settings->  click on the desired domain -> at the end you will see DNS management)

You may need to enable advanced settings.
Create a TXT record containing this text:   
<code>"v=spf1 a mx ip4:(ip-address) include:_spf.google.com ?all"</code>

<code>Note:</code> In the hostname if you are using the main domain name then just type "@", for eg. if you are using example.com as your mailserver then just type '@' in the hostname, else if you are using the subdomain, like mailer.example.com , then pass mailer as the hostname.

Save your changes.

<code>Note:</code> Your new SPF record can take up to 48 hours to go into effect, but this usually happens more quickly.

Now when SPF in place , lets set up our DKIM :

<h1>About DKIM</h1>

DKIM is an Internet Standard that enables a person or organisation to associate a domain name with an email message. This, in effect, serves as a method of claiming responsibility for a message. At its core, DKIM is powered by asymmetric cryptography. The sender's Mail Transfer Agent (MTA) signs every outgoing message with a private key. The recipient retrieves the public key from the sender's DNS records and verifies if the message body and some of the header fields were not altered since the message signing took place.

To configure DKIM , please follow this <a href="https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy">Digitalocean post</a> .

Once you are done with the process , open mail.txt, copy the key under the p parameter,
and add it to TXT record

{% highlight ruby %}

Name: mail._domainkey.example.com.

Text: "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC5N3lnvvrYgPCRSoqn+awTpE+iGYcKBPpo8HHbcFfCIIV10Hwo4PhCoGZSaKVHOjDm4yefKXhQjM7iKzEPuBatE7O47hAx1CJpNuIdLxhILSbEmbMxJrJAG0HZVn8z6EAoOHZNaPHmK2h4UUrjOG8zA5BHfzJf7tGwI+K619fFUwIDAQAB"

{% endhighlight %}

<code>Note: </code> Make sure you copy your own key , avoid spacing while copying the key from nano or vim editor and the name format shouldnt be changed.

DKIM config is complete, please note that changes may take couple of hours to propogate.

To check you dkim status follow this <a href="http://dkimcore.org/tools/keycheck.html">link </a> 

<code>Note:</code> In my case pass mail as a selector, and example.com as domain name.

These are the most basic records required for preventing your mails from being marked as spam, make sure A record for domain mapping, AAAA record for ip6 domain mapping is in place.

<h2>MX Record</h2>
We also require a MX(mail exchanger) record.

MX (Mail Exchanger) records specify and prioritize the incoming mail servers that receive email messages sent to your domain name. There is often no need to modify your MX records. Sometimes you have to update them if you host a website with one network (such as ours) but you have email hosted in another.

<h1>To add an MX record</h1>

Log in to your domain account.
On your product list next to Domains, click the Text Icon (plus sign button) to expand the list:
Expand Domains List

Next to the domain you want to manage, under the Action section, click on the Manage DNS button:
Manage DNS Button

At the bottom of the Records section, click Add and select MX from the drop-down list.

Complete the following fields:

<code>Name -</code>  Enter the domain name or subdomain for the MX record. For example, type @ to map the record directly to your domain name, or enter the subdomain of your host name, such as www or ftp.

<code>Value -</code> Enter the mail server's address, such as smtp.secureserver.net.
Priority - Enter, or select the priority you want to assign to the mail server.

<code>TTL -</code> Select how long the server should cache the information.

Click Save.

<h2>PTR RECORD</h2>

You can think of the PTR record as an opposite of the A record. While the A record points a domain name to an IP address, the PTR record resolves the IP address to a domain/hostname.

PTR records are used for the reverse DNS (Domain Name System) lookup. Using the IP address you can get the associated domain/hostname. An A record should exist for every PTR record.

The usage of a reverse DNS setup for a mail server is a good solution. Some external mail exchange servers make reverse DNS lookups before accepting messages originating from your mail server.

You can check whether there is a PTR record set for a defined IP address. The syntax of the commands on a Linux OS are:

<code>dig -x (ip-address)</code>

You will see the domain name as a output, in case if you dont see the domain name, then You should contact your ISP and ask him to add a PTR record for your ips.

Since I am using digitalocean droplet, so:

DigitalOcean automatically configures the reverse dns entry (PTR) on their end. It will be the hostname you choose when you set up your dropplet. You can change/check this in the control panel by selecting your dropplet, then settings, then rename. As it says, changing the name there will update the PTR but not the hostname of the system, that is something you will need to do, instructions for that vary depending on the system you have installed.

You can check the PTR by using the host command as follows:

<code>dig -x (ip-address)</code>

You will have to substitute your IP address in the command above.
Example:
dig -x 8.8.8.8
8.8.8.8.in-addr.arpa domain name pointer google-public-dns-a.google.com.

You may also be a little confused: You can not update the PTR of an IP address you have not been assigned . The IP address you are using from DigitalOcean may be able for you to use, but it has not really been assigned in a way you can freely control as far as reverse dns goes, therefor you can't change a PTR record for an IP you don't have access to. (That's why mail servers check PTR; if you have a properly configured reverse dns record your more likely to be authorized to send mail from that IP.)

Now with all the records in place, postfix configured , try this python script to test your mailserver:

{% highlight ruby %}

#!/usr/bin/python

import smtplib

sender = 'from@fromdomain.com'
receivers = ['to@todomain.com']

message = """From: From Person <from@fromdomain.com>
To: To Person <to@todomain.com>
Subject: SMTP e-mail test

This is a test e-mail message.
"""

try:
   smtpObj = smtplib.SMTP('localhost')
   smtpObj.sendmail(sender, receivers, message)         
   print "Successfully sent email"
except SMTPException:
   print "Error: unable to send email"

{% endhighlight %}

Voila! Mail successfully sent, check the original mail, and see for your records as being marked as PASS. For more pyhton scripts follow this article from <a href="https://www.tutorialspoint.com/python/python_sending_email.htm">tutorials point</a> .


In case if you still get the message as spam , then checkout your records on these websites

<a href="http://mxtoolbox.com">Mxtoolbox</a>

<a href="https://www.port25.com/authentication-checker/">For auth verification</a>

<a href="http://spamcheck.postmarkapp.com/">Spam check content</a>

To maintain the reputation of your email-id , avoid sending mail as a spam, so if your sending to bulk email-ids, verify that email-id really exist.

<h1>How to test if the email address actually exists</h1>

To check if user entered email xyz@gmail.com really exists go through the following in command prompt on windows / terminal on mac. 

Step 1 – Find mail exchanger or mail server of gmail.com

COMMAND:
nslookup -q=mx gmail.com
RESPONSE:
Non-authoritative answer:
gmail.com   mail exchanger = 40 alt4.gmail-smtp-in.l.google.com.
gmail.com   mail exchanger = 20 alt2.gmail-smtp-in.l.google.com.
gmail.com   mail exchanger = 10 alt1.gmail-smtp-in.l.google.com.
gmail.com   mail exchanger = 5 gmail-smtp-in.l.google.com.
gmail.com   mail exchanger = 30 alt3.gmail-smtp-in.l.google.com.

Step 2 – Now we know the mail server address so let us connect to it. You can connect to one of the exchanger addresses in the response from Step 1.

COMMAND:
telnet alt2.gmail-smtp-in.l.google.com 25
RESPONSE:
Connected to alt2.gmail-smtp-in.l.google.com.
Escape character is ‘^]’.
220 mx.google.com ESMTP r63si40328457plb.39 - gsmtp

COMMAND:
helo hi
RESPONSE:
250 mx.google.com at your service

COMMAND:
mail from: <youremail@gmail.com>
RESPONSE:
250 2.1.0 OK r63si40328457plb.39 - gsmtp

COMMAND:
rcpt to: <xyz@gmail.com>
RESPONSE:
550 5.1.1 <xyz@gmail.com>: Recipient address rejected: User unknown in virtual alias table

COMMAND:
quit
RESPONSE:
221 2.0.0 Bye

To automate the process via script use the following <a href="https://www.scottbrady91.com/Email-Verification/Python-Email-Verification-Script">link .</a>



<h2>Alias Creation</h2>

Alias Creation are used to recieve mails on behalf of your mailserver
<h2>Steps</h2>


<p>Set the location of the virtual_alias_maps table. This table maps arbitrary email accounts to Linux system accounts. We will create this table at /etc/postfix/virtual. We can use the postconf command:</p>

<code>sudo postconf -e 'virtual_alias_maps= hash:/etc/postfix/virtual'</code>

Next, we can set up the virtual maps file. Open the file in your text editor:

<code>sudo nano /etc/postfix/virtual</code>

The virtual alias map table uses a very simple format. On the left, you can list any addresses that you wish to accept email for. Afterwards, separated by whitespace, enter the Linux user you'd like that mail delivered to.

I am using debian as a user, so i map the address to debian.

{% highlight ruby %}
info@domain_name debian
bounces@domain_name debian
no-reply@domain-name debian
alert@domain_name debian
{% endhighlight %}

this means , if someone sends mail to info@domain_name  then those mail will be delievered to debian user.

We can apply the mapping by typing:

<code>sudo postmap /etc/postfix/virtual</code>

Restart the Postfix process to be sure that all of our changes have been applied:

<code>sudo systemctl restart postfix</code>

Make sure your postfix aliases :"/etc/aliases" looks like 

{% highlight ruby %}
# See man 5 aliases for format
postmaster:    root
root:   newsletter
{% endhighlight %}


here root has given its access to send mail to the newsletter, you can configure any name here, but if you change make sure you change the name from postfix config also .

<code>Note</code> If you modify your alliases, then use comand 

<code>sudo newaliases</code>

Now if you send the mail to info@domain_name then it will be delievered to debian , you can check your mails inside /var/mail/debian.

to see mails install mailutils first:

<code>sudo apt-get install mailutils</code>

<code>sudo mail</code>

and if you are logged in as debian then you will be able to see the debian user mail.

<h1>To update /etc/aliases</h1>

First check your aliase location:

<code>postconf alias_maps </code>

then update the file

after that use:

<code>newaliases </code>

<h1>Accessing postfix from outside</h1> 
Suppose you are hitting your postfix server from another server, eg. in case of celery, then pass the ip of your postfix server inside the server variable(inside main sendmail file). Also add this new ip to the mynetworks(main.cf). Now since your new server doesnt have a domain name , so to avoid any error , we have to pass this ip in "/etc/postfix/header_filters". 
Eg./^Received:.*128\.199\.xxx\.xxx.*/ IGNORE 

Since this new ip doesnt have any certificates passed, so we have to add an entry of this ip to the (/etc/opendkim/TrustedHosts) of postfix server.
So, now when this new ip hits the postfix of main server , it will allow to hit as its own trusted server.

For postfix basic configuration files follow my <a href="https://github.com/sumitgupta0001/mailserver_basic"> mailserver github repository</a>

<h2>Spamming</h2>

Instead of all these efforts, if you still get your mails as spam , then try to avoid these practices:

<h1>1. Avoid Purchased Lists</h1>
Have you ever been tempted to grow your list by a million potential customers in no time? Have you been to forums where thousands of “targeted leads” are sold for a few bucks?

Purchased lists are ticking time bombs, waiting to devastate your reputation as a sender. Riddled with dead emails and spam traps, they quickly inform mailbox providers that you break the rules by sending unsolicited emails.

At best, your messages may end up in junk folders. At worst, you may be branded as a spammer.

If you still buy emails lists, STOP NOW.

<h1>2. Watch What You Say</h1>
Spam filters analyze your content. There are no magic keywords to enhance deliverability, but limiting the use of risky words—such as free, buy, promo, etc.—reduces the likelihood of your emails landing in the spam folder.

Moreover:

Link only to legitimate sites with reputable domains.
Don’t go crazy with email size (30 kb is just fine.)
Balance the image-to-text ratio.
Host your images at credible services only.

<h1>3. Team Up With A Reliable ESP</h1>
Email Service Providers (ESP) are evaluated as senders based on the reputation of the Internet Protocol (IP) addresses and domains of their clients.

Careless ESPs with low scores on the IP addresses of their senders are destined for spam folder delivery. Eventually, they will be blocked by the providers like Gmail, Yahoo! Mail, and Hotmail.

ESPs that send only solicited emails and ban spammers from their platforms have greater credibility with mailbox providers. Their Customers are more likely to experience undisturbed inbox delivery if they follow the steps outlined in this post.

<h1>4. Get Certified!</h1>
If you are on a dedicated IP space, you should definitely look at the certification provided by a company called Return Path. Once they audit your mailing practices, you can get a Sender Score Certified status which will guarantee that you inbox at most of the major ISPs out there. This service is not free, but it definitely deserves a closer look. The money spent on the fees should be easily returned by the increased conversions.

<h1>5. Avoid Dirty Tricks</h1>
What may have been effective in 1997 no longer works today. Remember, being caught red-handed in any of these practices may cause permanent damage to your deliverability ratios:

Hashbusting: Inserting random characters in the subject line or content to fool spam filters, e.g. “F.ree. p.r!z.e”
Deceptive Subject Lines: Starting the subject line with “Re:” or “Fwd:” to suggest an ongoing communication with the sender.
Misleading Claims: Subject line stating that the recipient has won a prize, while the copy lists conditions that have to be met in order to claim it.
Image Text: Concealing a text message in an image to fool spam filters.
6. Whitelist Me, Please!
Your Email Marketing Service (EMS) asks mailbox providers, such as Gmail and Yahoo Mail, to whitelist your domain or Internet Protocol (IP) address. That is why it’s important to send marketing emails through a reputable EMS, rather then sending emails from your own email server or email account.

When confirming your new subscribers (e.g. via a welcome email), ask them to add your “From” address to their address books. It is a foolproof way to release all future emails from the constraints of the spam filters. This is so easy, yet practiced so rarely.

<h1>6. No Risk, No Problem</h1>
Your email campaigns may contain risky elements that are detrimental to the deliverability of your messages. Here’s a brief checklist to go through before you hit the “Send” button:

Be careful with words associated with the language of sales. If overused, they may trigger spam filters and route your emails to junk folders. Risky words include: “prize”, “free”, “bonus”, “buy, “purchase”, “order” etc.
Common sense will tell you that one exclamation mark per sentence is enough. Never shout at your subscribers, (e.g. “Buy my e-book now!!!”). Exclamation marks are especially risky in email subject lines.
Never overdo the use of “ALL CAPS.” When emphasis is needed, use a maximum of one word per sentence in all capitals, never a whole sentence.



<code>This is all for mailserver, happy coding :)</code>

