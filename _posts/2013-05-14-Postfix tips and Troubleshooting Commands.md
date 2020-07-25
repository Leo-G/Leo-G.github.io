Here's a list of stuff I user everyday and other email admins will also be using, Let me know if I missed anything

List/Print current mail queue



# postqueue –p
# mailq

If it's a huge que, you can pipe it with tail



# mailq | tail

Flush the queue

# postqueue -f

Schedule immediate delivery of all mail that is queued for the named as domain.come.

# postqueue -s domain.com

TO delete all queue



# postsuper -d ALL

To delete a particular message

# postsuper -d messageid

Reque the mail or resend particular mail

#postfix -r msgid

To find mail version

#postconf -d mail_version
mail_version = 2.6.6

You can also follow the steps in the link below which is the most simple and well explained documentation available with regards to Configuring postfix.

Postfix Configuration -
