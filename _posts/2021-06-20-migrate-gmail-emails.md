---
layout: post
title: "Migrate emails from Gmail to another IMAP server"
tags: server
---

If you decided that you want to move away from Gmail, you can migrate all emails from Gmail to a different
IMAP server. Using the tool [`imapsync`](https://github.com/imapsync/imapsync) that's an easy task. It transfers emails
from one IMAP server to another and preserves all email data including headers, attachments, timestamps, and folder structure.

In this guide, I'm using `imapsync` version 1.977 on Linux. Find a list of installation guides for different
Linux systems [in this directory on GitHub](https://github.com/imapsync/imapsync/tree/master/INSTALL.d).

There's already a good official
[guide on how to use imapsync with Gmail](https://github.com/imapsync/imapsync/blob/master/FAQ.d/FAQ.Gmail.txt).
I encourage you to read that guide first, I'll only describe some of the additional information I consider to be useful
in this blog post. Server 1 is Gmail and server 2 is your destination IMAP server. The destination IMAP server can
already contain emails, `imapsync` does not delete any email on the destination server by default.

Instead of using the `--password1`/`--password2` arguments, you should provide the passwords for the two IMAP servers
through a text file so that it's not saved in the Shell history.

First the Gmail server: The IMAP server is `imap.gmail.com` and the username `user@gmail.com`. For the password you
have to generate an app password, it will not work with your Gmail account password. Google provides instructions for
[generating app passwords](https://support.google.com/accounts/answer/185833). Generate an App Password for the app
*Mail* and store the 16-character password in a text file called `password1.txt`. Remember to remove the app password
from your Google account after the email migration is complete. For the destination IMAP server, figure out the server
host, the username, and your password. Store the password in a file called `password2.txt`.

Gmail is not exposing system folders (e.g. sent emails, trash, drafts, etc.) with the `[Gmail]/` prefix anymore (the
version of `imapsync` I'm using assumes that) but with `[Google Mail]/`. You can see that in this output from a dry run:

```text
Host1 folder   16/22 [[Google Mail]/All Mail]            Size: 6769848488 Messages: 71145 Biggest:  31498222
Host2 folder   16/22 [[Google Mail].All Mail]             does not exist yet
Host2-Host1                                                    -6769848488           -71145          -31498222
```

The command `--regextrans2 "s,\[Google Mail\].,,"` will overwrite the default behavior and replace the `[Google Mail]/`
prefix instead of the `[Gmail]/`
prefix.

If you have emails that have more than one label in Gmail, you have to pay special attention to this situation. The
IMAP protocol only supports folders, but not labels. Gmail will expose the labels as folders, so emails with multiple
labels will be shown in multiple folders and essentially be duplicated. Note that emails that are starred or marked as
important are also exposed through respective folders. The `imapsync --gmail1` argument includes the
`--skipcrossduplicates` argument which will copy every email only once, even when it is contained in multiple folders.
It will only copy it in the first listed folder, so pay attention to the order of folders when you perform the dry run.
Use the arguments `--folderfirst`/`--folderlast` to change the order. I use the argument `--folderlast "Starred"` to
copy all starred emails in the end, so that only starred emails without labels will be copied (all others will be copied
to the respective label folder).

I use the `--exclude "All Mail"` argument because I only want to copy emails with at least one label. To check which
emails I leave behind in Gmail, I use the Gmail search `has:nouserlabels -label:sent -label:chats -label:trash`.
It will list all emails that don't have any labels, are not in the Sent folder, and are not in the trash. If you want
to transfer unlabeled emails, use `--folderlast "[Google Mail]/All Mail"` instead, which will copy all unlabeled emails
to the `All Mail` folder.

Before you copy the emails, perform a dry run with `--dry` which will describe what `imapsync` would do without actually
copying any emails. Carefully check the output to make sure it's exactly what you want to do.

```bash
./imapsync --host1 imap.gmail.com --user1 user@gmail.com --passfile1 password1.txt \
    --host2 imap.example.com --user2 youremail@example.com --passfile2 password2.txt \
    --gmail1 \
    --regextrans2 "s,\[Google Mail\].,," \
    --folderfirst "INBOX" \
    --exclude "All Mail" \
    --exclude "Important" \
    --exclude "Spam" \
    --folderlast "Starred" \
    --addheader --dry
```

Once you started the actual copy process without the `--dry` argument, this might take a long time. For my setup, I
averaged around 1 message per second (average message size was 160 KB). That resulted in 20 hours of copy time
for my 70,000 emails. I recommend running this command from a server, Raspberry Pi, or computer that will not be turned
off.
