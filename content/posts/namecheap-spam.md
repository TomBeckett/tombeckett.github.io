---
title: "Namecheap: A hive of spam and villainy"
date: "2022-01-15T01:00:00"
disableShare: true
hideFooter: true
tags: ['email', 'spam', 'hosting']
categories: ['meta']
---

Anyone who has ever sent lots of emails knows how it's hard to get them delivered without going into spam. There's an [entire]() [industry]() of [providers]() to pay for and lots of technologies to prove you're domain is legitimate such as [SPF](https://en.wikipedia.org/wiki/Sender_Policy_Framework) and [DMARC](https://en.wikipedia.org/wiki/DMARC).

Somehow, every few days I get an email like this:

![Spam Email Example](/images/posts/namecheap-spam/spam-example.png)

What amazes me is that this are *not* marked as spam yet somehow violate every rule:

- Titles like `WelcomeTo YourLifeInsurance 422`.
- Contain only a `<img />` or `<iframe />` pointing to an S3 bucket.
- Have an email address like `NXSlbwv-tsnObQ33wMv-noreply@ma-ek.wareeye.com`.
- Are tagged `to:` a random name that is not mine.

## The investigation begins

After a few months I snap and begin my quest to find out **who** is sending these to me. 

All domains are used once and purchased using [namecheap.com](https://namecheap.com) and are entirely anonymous:

```bash
tombeckett@Toms-MBP tombeckett % whois flurrieneat.biz

% IANA WHOIS server
% for more information on IANA, visit http://www.iana.org
% This query returned 1 object

Domain name: flurrieneat.biz
Registry Domain ID: D6A4FDFA75BD14759AD6BF3307DAD8B41-NSR
Registrar WHOIS Server: whois.namecheap.com
Registrar URL: http://www.namecheap.com
Updated Date: 2021-06-11T17:12:18.35Z
Creation Date: 2020-06-29T16:59:01.87Z
Registrar Registration Expiration Date: 2022-06-29T16:59:01.87Z
Registrar: NAMECHEAP INC
Registrar IANA ID: 1068
Registrar Abuse Contact Email: abuse@namecheap.com
Registrar Abuse Contact Phone: +1.9854014545
Reseller: NAMECHEAP INC
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Registry Registrant ID: 
Registrant Name: Redacted for Privacy
Registrant Organization: Privacy service provided by Withheld for Privacy ehf
Registrant Street: Kalkofnsvegur 2 
Registrant City: Reykjavik
Registrant State/Province: Capital Region
Registrant Postal Code: 101
Registrant Country: IS
Registrant Phone: +354.4212434
Registrant Phone Ext: 
Registrant Fax: 
Registrant Fax Ext: 
Registrant Email: f3084f1b2d9f46ee85993f43453c6e65.protect@withheldforprivacy.com
......

Name Server: ns1.flurrieneat.biz
Name Server: ns2.flurrieneat.biz
DNSSEC: unsigned
URL of the ICANN WHOIS Data Problem Reporting System: http://wdprs.internic.net/
>>> Last update of WHOIS database: 2022-01-15T21:19:17.15Z <<<

tombeckett@Toms-MBP tombeckett % 
```

I think whats happening is someone is buying brand new domains which are not flagged (yet) or recently expired domains which are trusted by providers. That *kind of* explains why they get through, but why namecheap?

## A spammers paradise

Namecheap have a unique set of features which make it perfect for sending large amount of spam:

- All domain information will point to [WithheldForPrivacy.com](https://withheldforprivacy.com/) (as seen in output above). 
- You can purchase a `.com` domain for only £5 using [bitcoin](https://www.namecheap.com/support/payment/bitcoin/).
- They provide you with [free email forwarding for 100 addresses](https://www.namecheap.com/support/knowledgebase/article.aspx/308/2214/how-to-set-up-free-email-forwarding/).
- The abuse team seems to be [extremely slow at investigating claims](https://www.ncsc.gov.uk/files/Active-Cyber-Defence-ACD-The-Fourth-Year.pdf).

UK's National Cyber Security Centre (NCSC) have released [their annual report](https://www.ncsc.gov.uk/files/Active-Cyber-Defence-ACD-The-Fourth-Year.pdf) with a section on phishing and its pretty damning:

![Spam Email Example](/images/posts/namecheap-spam/ncsc-report.png)

[In this year's report](https://www.ncsc.gov.uk/files/Active-Cyber-Defence-ACD-The-Fourth-Year.pdf) they state that Namecheap hosted 25%+ of fake UK govt phishing sites. 

They also concluded: 

> Looking specifically at the number of campaigns hosted by NameCheap against its monthly median attack
availability, we see that by mid-year the median takedown times were consistently in excess of 60 hours. This
undoubtedly made NameCheap an attractive proposition to host phishing and may explain the rise in monthly
hosted campaigns that followed for UK government-themed phishing.

## The company line

All this points to namecheap having a real problem with spam and in particular enforcing it in a timely manner.

When it comes to communication with namecheap you have two real options: 

- [Namecheap Subreddit](https://old.reddit.com/r/NameCheap/)
- [Namecheap CEO's Twitter](https://twitter.com/namecheapceo)

Firstly, the Subreddit is [flooded](https://old.reddit.com/r/NameCheap/comments/s4eaz0/receiving_spamscam_with_spoofed_from_headers_but/) [with](https://old.reddit.com/r/NameCheap/comments/s0xn62/blatant_copycat_scam_site/) [examples](https://old.reddit.com/r/NameCheap/comments/rpu323/smish_and_spam_campaign_using_namecheap_namecheap/) [of](https://old.reddit.com/r/NameCheap/comments/rlm58r/spam_sms_for_manifestdeedcom_on_namecheap_domain/) [spam](https://old.reddit.com/r/NameCheap/comments/r8vdb6/namecheap_still_contributing_to_the_ecosystem_of/) [reports](https://old.reddit.com/r/NameCheap/comments/ozhk7z/namecheap_is_contributing_the_ecosystem_of_crypto/) [and](https://old.reddit.com/r/NameCheap/comments/oxymku/namecheap_is_once_again_hosting_a_crypto_phishing/) [how](https://old.reddit.com/r/NameCheap/comments/ndwmep/crypto_scam_websites_run_rampart_on_your_platform/) [abuse](https://old.reddit.com/r/NameCheap/comments/n6rluf/daily_reminder_that_leakgirlscom_prolific_reddit/) [doesnt](https://old.reddit.com/r/NameCheap/comments/mrft0m/when_is_namecheap_going_to_pull_the_plug_on/) [care](https://old.reddit.com/r/NameCheap/comments/mo932s/received_a_phishing_email_to_the_correct_email/). If you have a domain to report, this seems to be the way to actually get a response.

Secondly, the CEO seems to get into real internet fights dismissing the problem:

{{< tweet user="NamecheapCEO" id="1369273660519964678" >}}

{{< tweet user="NamecheapCEO" id="1385573738779926528" >}}

{{< tweet user="NamecheapCEO" id="1430490073032630278" >}}

And its a big fan of Crypto, so I think this strategy is going to stay:

{{< tweet user="NamecheapCEO" id="1438535149310791686" >}}

## Conclusion

There can't be much money in £5 domains once a year, but if you can sell a domain for only for 47 hours to a spammer, then thats good recurring income. 

Ironically, by tackling the spam quickly they look like the good guys, a real win-win. Time to move email provider as I suspect these emails won't change any time soon.