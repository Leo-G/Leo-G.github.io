DMARC record is mostly used to track phishing and spam. It can also be very instrumental in abolishing it if implemented correctly.

 

How to add a DMARC record?

You need to add SPF and DKIM before you add a DMARC record. For SPf click here and for dkim go here

Adding a DMARC record is the same as adding a DKIM or SPF record below is the specification

Tag NamePurposeSample

v	Protocol version	v=DMARC1
pct	Percentage of messages subjected to filtering	pct=20
ruf	Reporting URI for forensic reports	ruf=mailto:authfail@example.com
rua	Reporting URI of aggregate reports	rua=mailto:aggrep@example.com
p	Policy for organizational domain	p=quarantine
sp	Policy for subdomains of the OD	sp=reject
adkim	Alignment mode for DKIM	adkim=s
aspf	Alignment mode for SPF	aspf=r


Only the v (version) and p (policy) tags are required. Three possible policy settings, or message dispositions, are available:

none - Take no action. Log affected messages on the daily report only.
quarantine - Mark affected messages as spam.
reject - Cancel the message at the SMTP layer.

Alignment mode refers to the precision with which sender records are compared to SPF and DKIM signatures, with the two possible values being relaxed or strict. represented by "r" and "s" respectively. In short, relaxed allows partial matches, such as subdomains of a given domain, while strict requires an exact match.

Make sure to include your email address with the optional rua tag to receive the daily reports.

Here are some example DMARC TXT records (_dmarc.your_domain.com IN TXT) you may modify for your own use. Of course, replace "your_domain.com" and "postmaster@your_domain.com" with your actual domain and email address.

In the following example TXT record, if a message claims to be from your domain.com and fails the DMARC checks, no action is taken. Instead all of these messages appear on the daily aggregate report sent to "postmaster@your_domain.com."
"v=DMARC1; p=none; rua=mailto:postmaster@your_domain.com; ruf=mailto:authfail@your_domain.com"

In the next example TXT record, if a message claims to be from your domain.com and fails the DMARC checks, quarantine it 5% of the time. Then email daily aggregate reports to "postmaster@your_domain.com."
"v=DMARC1; p=quarantine; pct=5; rua=mailto:postmaster@your_domain.com ruf=mailto:authfail@your_domain.com"

In the final example, if a message claims to be from "your_domain.com" and fails the DMARC checks, reject it 100% of the time. Then email daily aggregate reports to "postmaster@your_domain.com" and "dmarc@your_domain.com."
"v=DMARC1; p=reject; rua=mailto:postmaster@your_domain.com, mailto:dmarc@your_domain.com ruf=mailto:authfail@your_domain.com"

The ruf and rua email addresses need to be at your_domain.com.

To check if your record is correct go here http://www.dmarcian.com/dmarc-inspector/

Goolge generallly sends you a report every 24 hours, if you don’t get any reports then check whether your receivers are checking DMARC.

If you don’t wanna go thru the above hassel, then you can goto http://www.kitterman.com/dmarc/assistant.html and generate a record.

Once you add the record you will get an email report in xml, you can then create a free account at http://www.dmarcanalyzer.com/  to evaluate the report.
