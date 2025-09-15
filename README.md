# pendmail

Use at your own risk! Still under testing.

1. Press `,p` on a mail or tagged mails. The script prompts for a date, 
   e.g., `2025-09-15T09:00`, or a duration, e.g., `+5d`, the mails are then 
   shifted to `pending` with an `X-Pending-Until` header.

2. Run the `pendmail` script regularly (via cron, while fetching mail...) to 
   shift mails back into the `inbox`.

Similar to:
* Getting Things Done [defer mail](https://gettingthingsdone.com/wp-content/uploads/2014/10/GettingEmail.pdf)
* Gmail [snooze](https://support.google.com/mail/answer/7622010?hl=en&co=GENIE.Platform%3DDesktop) feature

Recommendations:
* Test your initial configuration carefully on spam.
* Use the NeoMutt [trash](https://neomutt.org/feature/trash) feature to 
  backup mails as they are postponed.

## Requirements

* [NeoMutt](https://neomutt.org) with local MailDirs
* [procmail](https://porkmail.org/era/procmail/quickref)
* [dateutils](https://www.fresse.org/dateutils/)

## Installation

* Copy `pendmail` into the `PATH`
* Copy `pendmail.rc` into the same location or to `~/.procmail`
* Edit `pendmail.rc` to set `MAILDIR`, `DEFAULT`, `PENDING`, `BACKUP`
* Create a `pending` mailbox
```
mkdir -p $MAILDIR/pending/{cur,new,tmp}
```

## NeoMutt Configuration

* Declare the `pending` mailbox and show the `X-Pending-Until` header field
```
mailboxes +pending
hdr_order X-Pending-Until
unignore X-Pending-Until
```

* Macros to shift mail to the `pending` mailbox (update 
  `$HOME/bin/pendmail`...)
```
macro index <esc>p \
   '<enter-command> setenv PENDMAIL_MODE source<enter>\
<enter-command> source $HOME/bin/pendmail|<enter>\
<enter-command> unsetenv PENDMAIL_MODE<enter>\
<enter-command> set my_pipe_decode=$pipe_decode my_pipe_split=$pipe_split 
my_pipe_sep=$pipe_sep pipe_decode=no pipe_split=yes pipe_sep=<enter>\
<enter-command> setenv PENDMAIL_PENDING_UNTIL $my_pending_until<enter>\
<tag-prefix><pipe-message>$HOME/bin/pendmail<enter>\
<enter-command> unsetenv PENDMAIL_PENDING_UNTIL<enter>\
<enter-command> set pipe_decode=$my_pipe_decode pipe_split=$my_pipe_split pipe_sep=$my_pipe_sep my_pending_until=<enter>\
<tag-prefix><delete-message>' \
    'move mail(s) to pending'

macro pager <esc>p \
    '<pipe-message>$HOME/bin/pendmail<enter><delete-message>' \
    'move mail to pending'
```

## Design Notes

Known flaw: the NeoMutt macro always calls `<delete-message>` regardless of 
whether the mails have been moved successfully or not. Use undelete if 
unsure!

The NeoMutt [trash](https://neomutt.org/feature/trash) feature can help to 
save mails. On the other hand, every time a mail is made pending, a copy is 
placed in the trash...

Maybe this could be fixed by having `pendmail` update a file which is then 
`source`d to either show a message or delete the mails? Or by having the 
procmail script reroute to the inbox if `$PENDINGUNTIL` is invalid?

## Bugs?

Send a pull request!

