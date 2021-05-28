---
layout: post
title: "Export emails from Gmail to another IMAP server"
tags: server
---

If you decided that you want to move away from Gmail, there's an easy way to move all emails from Gmail to a different
IMAP server. Using the tool `imapsync` that's an easy task. It transfers emails from one IMAP server to another.

https://github.com/imapsync/imapsync
https://imapsync.lamiral.info/

In this guide I'm using imapsync version 1.977 on Linux. There's a good [guide on how to use imapsync with Gmail](https://github.com/imapsync/imapsync/blob/master/FAQ.d/FAQ.Gmail.txt). [Link to installation guide] - just install perl dependencies

You should provide the passwords for the two IMAP server through a text file so that it's not stored in the Shell history.

First the Gmail connection: The IMAP server is `imap.gmail.com` and the username `user@gmail.com`. For the password you
have to generate an app password, it will not work with your Gmail account password. Google provide instructions for
[generating app passwords](https://support.google.com/accounts/answer/185833). Generate an App Password for the app
*Mail* and store the 16-character password in a text file called `password1.txt`. Remember to remove the app password again after the email migration is complete.

For the destination IMAP server, figure out the server host, the username, and your password. Store the password in a file
called `password2.txt`.

I'm overwriting the two parameters from `--gmail` because it's not `[Gmail]` anymore, see

```
Host1 folder   16/22 [[Google Mail]/All Mail]            Size: 6769848488 Messages: 71145 Biggest:  31498222
Host2 folder   16/22 [[Google Mail].All Mail]             does not exist yet
Host2-Host1                                                    -6769848488           -71145          -31498222
```

```bash
./imapsync --host1 imap.gmail.com --user1 user@gmail.com --passfile1 ../password1.txt \
        --host2 imap.example.com --user2 youremail@example.com --passfile2 ../password2.txt \
        --gmail1 \
        --regextrans2 "s,\[Google Mail\].,," \
        --folderfirst "INBOX" \
        --exclude "All Mail" \
        --exclude "Important" \
        --exclude "Spam" \
        --exclude "Starred" \
        --addheader --dry
```

If you want to keep the `All Mail` folder, you want to use `--folderlast  "All Mail"` instead of
`--exclude "All Mail"`.

Useful Gmail search: `has:nouserlabels -label:sent -label:chats -label:trash`

Explain `--folderlast "Starred"`.

This might take a long time. For my setup I averaged around 1 message per second (average message size was 160 KB).
