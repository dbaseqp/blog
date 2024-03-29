---
layout: article
title: How to Use Multiple Git Accounts on One Computer
excerpt: "Learn how to commit to different repositories from different accounts after GitHub has deprecated password authentication for authenticated Git operations. Very useful if you are working on different projects for different organizations such as personal GitHub and a school GitHub."
categories: [article]
tags: [workflow, development]
aside:
  toc: true
---

# Basic Introduction 
If you're a developer who holds many titles in different organizations, you've probably used Git before. If you're anything like me, depending on which organization I may be doing work for (school, personal, club), I may use different accounts for different repositories and projects. Since GitHub has taken a big security move and deprecated password authentication for authenticated CLI Git operations (e.g. pushing), you can no longer simply swap between logon credentials.

# Tutorial
## Overview
Essentially, there are 2 correct ways of setting up multiple GitHub accounts: SSH keys (which works for all repos) and per-repo accounts via Personal Access Tokens. In my experience, using SSH keys is easier because I get to reuse my identity for all future GitHub repos, so long as I use the right credentials.

## SSH Keys with GitHub
*First*, generate an SSH key for if you don't have one already.
```bash
$ ssh-keygen -t ed25519 -C "your_email@example.com"
```
You can then press enter through the rest of the prompts, or if you would like more security, check out "[Working with SSH key passphrases](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases)."

The SSH key will be generated in the `~/.ssh` directory inside your user's home directory. There will be a private key and a public key. The public key ends with ".pub" extension. 

*Second*, go to GitHub and go to your account settings and go to "SSH and GPG keys". Click "New SSH key" and then copy-paste the content of the ".pub" file into the bottom field. Name your key in the top field, then click "Add SSH key".

*Third*, create/edit the file `~/.ssh/config`, and put the following contents inside:

```
# Default github account: personal
Host github.com
   HostName github.com
   IdentityFile ~/.ssh/personal_private_key
   IdentitiesOnly yes
   
# Other github account: school
Host github-school
   HostName github.com
   IdentityFile ~/.ssh/school_private_key
   IdentitiesOnly yes
```

> Set the `Host` field to something that will help you remember what account it is for. I used "github-school", but you could set it to "work" or "".

*Fourth*, add your SSH private keys to your agent.

```bash
$ eval "$(ssh-agent -s)"                # starts ssh-agent service
$ ssh-add ~/.ssh/personal_private_key   # adds personal private key
$ ssh-add ~/.ssh/school_private_key     # adds school private key
```

##### Note: The above commands are run in Bash

*Fifth*, try testing the connection for your SSH keys.

```bash
$ ssh -T git@github.com
$ ssh -T git@github-school
```

If it works, you should get something like the result below for each account.

```
Hi dbaseqp! You've successfully authenticated, but GitHub does not provide shell access.
```

*Voila!* You're done and should be able to push/pull with the correct accounts. To specify which account you want to be pushing/pulling with, configure your remote like so:

```bash
# For new projects
$ git clone git@github.com:dbaseqp/blog.git
# OR
$ git clone git@github-school:dbaseqp/blog.git

# For projects you already have
$ git remote set-url origin git@github.com:dbaseqp/blog.git
# OR
$ git remote set-url origin git@github-school:dbaseqp/blog.git
```

## Personal Access Tokens
When you create a remote (e.g. origin), you can associate the remote with your Personal Authentication Token. This technique basically is going to associate a certain remote repository with one of your accounts. 

GitHub's Personal Authentication Token can be found in your user account's settings, near the bottom under "Developer settings." Then go to "Personal access tokens" and generate a new token if you need to. From there you can select the permissions you would like to grant that token. By default, I would say selecting "repo" is usually good enough. Click the generate button to create the token.

Once you have copied your token, I recommend storing somewhere secure like in a password manager because you won't be able to access it again. 

Then, from your CLI, use the below command to add the remote.
```bash
$ git remote add origin https://USERNAME:personal_access_token@github.com/USERNAME/REPOSITORY.git
```
If you are reconfiguring an existing remote, you may need to use `set-url` instead of `add`, but it's the same process.

Now you should be able to push without any problems.

# Common Mistakes
## Config user
While scouring forums and tutorials, I found a lot of posts recommending to use commands such as:
```bash
$ git config --global user.name "First Last"
$ git config --global user.email "you@example.com"
```
For switching to a particular account in a single-repository, they recommended to edit the local name and email information by removing the global flag:
```bash
$ git config user.name "Firt Last"
$ git config user.email "you@example.com"
## or ##
$ git config --local "First Last"
$ git config --local user.email "you@example.com"
```
## Set remote
Another technique was to set the remote to an address that followed this pattern:
```bash
$ git remote add origin https://USERNAME@github.com/USERNAME/REPOSITORY.git
```

However, both of these  ***do not*** actually swap the account being used in that repository. The first strategy simply changes the display information for your commit when you push to the repository. The second just doesn't work at all.

# Conclusion
Hopefully, this has been a useful tutorial for you. GitHub is a great tool, but with the deprecation of password authentication, using Git on the command line to use GitHub can be a little confusing.
