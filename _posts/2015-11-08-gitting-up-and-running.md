---
layout: post
title:  "Gitting up and running"
subtitle: "Making command line Git easier on my new Linux Box"
date:   2015-11-08 00:35:00
categories: [programming]
---

Shortly after I bought myself a nice and cheap old ThinkPad to run Linux, (*Ubuntu Mate 15.10*) that I could use to get some personal coding done, I ran into some issues using command line Git with Github. I've got Two-Factor Authentication enabled on my Github account, a fact that those of you familiar with how that works on Github will recognize as causing some minor changes in how one logs in via the command line. After embarassingly forgetting that I had to log in using a *personal access token* when using the command line, I made another discovery; When using a personal access token on Linux, you've got to re-enter that same token each and every time you would need to log in. Needless to say, this became incredibly annoying in quite quickly, and I set out for the internet in search of a way to make this process more seemless. I eventually pieced together what I needed to do, and below I lay out the steps I would've taken, had I found them all in a sensical order. If this can save anyone some of the trouble I went through figuring all this out on my own (*even if that someone is future me*), then I'll consider this a well spent blog post. 

Like I imagine most of you did, I discovered that a popular solution to this problem was to use a ```.netrc``` file with your personal access token included within, combined with git's credential-helper to do all of the token-remembering and login-lifting for you. 

On the surface, this was incredibly easy. All you need to do is create a ```.netrc``` in your home directory ```/home/USER``` that contains the following plain text:

```bash
machine github.com
login YOUR_GITHUB_USERNAME
password YOUR_PERSONAL_ACCESS_TOKEN
protocol https
```

Git will pick up on that, and use it to log you in whenever you push/pull/clone or whatever. **However**, you're also probably here because you're thinking "*But Kirby, that's hatefully insecure. Leaving a password - even a revokable token - out in plain text like that gives me chills!*". And you're right, so that's why the next step in our adventure is to encrypt that file using GPG so that we can forget it exists and get backing to coding in peace. 

Our first step, if you haven't already done so, is to generate a GPG key that we can use to sign things. This is super easy to do, and the process itself is very self explanitory. Simply run:

```bash
$ gpg --gen-key
```

And follow the on-screen instructions. **Be sure to use a password.** If it tells you that there are insufficent random bytes, just do something else until it's finished. Watch some netflix, code, surf the web, or even go to sleep and leave it running. Eventually it will finish up, and you'll have a GPG key we can use for our next steps. 

This next part is simple, but it's also the part that took me the longest. If you're running windows, you can simply use an explanation like [this one](http://stackoverflow.com/questions/5343068/is-there-a-way-to-skip-password-typing-when-using-https-github/18362082#18362082) to do everything we're doing here, but for Linux users I found the answers to be more elusive. 

Now that we've gotten ourselves a GPG key, we need to encrypt our .netrc file. That's easily accomplished by running 

```bash
$ gpg -e -r YOUR_GPG_ID ~/.netrc
```

That will output an encrypted version of your file, ```.netrc.gpg```. Now all we need to do is tell Github that it needs to decrypt this new file everytime we want to push/pull/etc and we're home free!

To do this, we need to make use of Git 1.7+'s "credential helper". Annoyingly, it doesn't immediately know to look for or decrypt .netrc.gpg files. So we'll run this command:

```bash
$ sudo chmod +x /usr/share/doc/git/contrib/credential/netrc/git-credential-netrc
```

This allows git to use the ```credential-netrc``` command to find and make user of our newly encrypted file. Then all we need to do is run:

```bash
$ git config -- global credential.helper /usr/share/doc/git/contrib/credential/netrc/git-credential-netrc
```

Which tells git to look for encrypted ```.netrc``` files and to decrypt them. At this point, we want to be sure to remember to run:

```bash
$ rm .netrc
```

So that all that's left on our machine is the GPG encrypted version of our credentials. You do need to remember that the first time you try to interact with Github through the commandline in a way that requires a log in, you will be prompted to provide your GPG password that you made when you initially generated your key. Don't forget it! Otherwise you'll have to start over again by creating a new ```.netrc```, GPG key, and encrypted version. 

Now, you should be pushing, pulling, and cloning via the command line with impunity. I hope this helped!