---
layout: post
title: "OpenStack : Git Gerrit and Jenkins Workflow"
date: 2014-02-03 14:54:23 -0500
comments: true
categories: [Gerrit, Git, Jenkins, CI]
---
# Gerrit Workflow  #

![Alt text in case picture load fails](/downloads/code/GerritGitJenkinsWorkflow.png "Git Gerrit Jenkins Workflow")
## Git Account Setup ##

You'll need a [Launchpad account](https://login.launchpad.net), since this is how the Web interface for the Gerrit Code Review system will identify you. This is also useful for automatically crediting bug fixes to you when you address them with your code commits.

If you haven't already, [join The OpenStack Foundation](https://www.openstack.org/join/) (it's free and required for all code contributors). Among other privileges, this also allows you to vote in elections and run for elected positions within The OpenStack Project. When signing up for Foundation Membership, make sure to give the same E-mail address you'll use for code contributions, since this will need to match your preferred E-mail address in Gerrit.

Visit https://review.openstack.org/ and click the Sign In link at the top-right corner of the page. Log in with your Launchpad ID.

Because Gerrit uses Launchpad OpenID single sign-on, you won't need a separate password for Gerrit, and once you log in to one of Launchpad, Gerrit, or Jenkins, you won't have to enter your password for the others.

You'll also want to upload an SSH key while you're at it, so that you'll be able to commit changes for review later.

Ensure that you have run these steps to let git know about your email address:

{% codeblock Git Config %}
git config --global user.name "Firstname Lastname"
git config --global user.email "your_email@youremail.com"
{% endcodeblock %}
To check your git configuration:
{% codeblock Git Config %}
git config --list
{% endcodeblock %}
## Git Review Installation ##

We recommend using the "git-review" tool which is a git subcommand that handles all the details of working with Gerrit, the code review system used in OpenStack development. Before you start work, make sure you have git-review installed on your system.

On Ubuntu, MacOSx, or most other Unix-like systems, it is as simple as:
{% codeblock %}
pip install git-review
{% endcodeblock %}

On Ubuntu Precise (12.04) and later, git-review is included in the distribution, so install it as any other package:
{% codeblock %}
apt-get install git-review
{% endcodeblock %}
On Fedora 16 and later, git-review is included into the distribution, so install it as any other package:
{% codeblock %}
yum install git-review
{% endcodeblock %}
On Fedora 15 and earlier you have to install pip (its package name is `python-pip`), then install git-review using pip in a conventional way.

On Red Hat Enterprise Linux, you must first enable the EPEL repository, then install it as any other package:
{% codeblock %}
yum install git-review
{% endcodeblock %}
On openSUSE 12.2 and later, git-review is included in the distribution under the name python-git-review, so install it as any other package:
{% codeblock %}
zypper in python-git-review
{% endcodeblock %}

All of git-review's interactions with gerrit are sequences of normal git commands. If you want to know more about what it's doing, just add -v to the options and it will print out all of the commands it's running.



## Project Setup ##

Clone a project in the usual way, for example:
{% codeblock %}
git clone git://git.openstack.org/openstack/nova.git
{% endcodeblock %}
You may want to ask git-review to configure your project to know about Gerrit at this point. If you don't, it will do so the first time you submit a change for review, but you probably want to do this ahead of time so the Gerrit Change-Id commit hook gets installed. To do so (again, using Nova as an example):
{% codeblock %}
cd nova
git review -s
{% endcodeblock %}
Git-review checks that you can log in to gerrit with your ssh key. It assumes that your gerrit/launchpad user name is the same as the current running user. If that doesn't work, it asks for your gerrit/launchpad user name. If you don't remember the user name go to the settings page on gerrit to check it out (it's not your email address).

Note that you can verify the SSH host keys for review.openstack.org here: https://review.openstack.org/#/settings/ssh-keys

If you get the error "We don't know where your gerrit is.", you will need to add a new git remote. The url should be in the error message. Copy that and create the new remote.
{% codeblock %}
git remote add gerrit ssh://<username>@review.openstack.org:29418/openstack/nova.git
{% endcodeblock %}
In the project directory, you have a `.git` hidden directory and a `.gitreview` hidden file. You can see them with:
{% codeblock %}
ls -la
{% endcodeblock %}

## 1.4 Normal Workflow ##

Once your local repository is set up as above, you must use the following workflow.

Make sure you have the latest upstream changes:

{% codeblock %}
git remote update
git checkout master
git pull --ff-only origin master
{% endcodeblock %}

Create a topic branch to hold your work and switch to it. If you are working on a blueprint, name your topic branch bp/BLUEPRINT where BLUEPRINT is the name of a blueprint in launchpad (for example, "bp/authentication"). The general convention when working on bugs is to name the branch bug/BUG-NUMBER (for example, "bug/1234567"). Otherwise, give it a meaningful name because it will show up as the topic for your change in Gerrit.

{% codeblock %}
git checkout -b TOPIC-BRANCH
{% endcodeblock %}

To generate documentation artifacts, navigate to the directory where the pom.xml file is located for the project and run the following command:

{% codeblock %}
mvn clean generate-sources
{% endcodeblock %}

### 1.4.1 Committing Changes ###

Git commit messages should start with a short 50 character or less summary in a single paragraph. The following paragraph(s) should explain the change in more detail.

{% codeblock %}
If your changes addresses a blueprint or a bug, be sure to mention them in the commit message using the following syntax:

Implements: blueprint BLUEPRINT
Closes-Bug: ####### (Partial-Bug or Related-Bug are options)
{% endcodeblock %}

For example:

{% codeblock %}
Adds keystone support

...Long multiline description of the change...

Implements: blueprint authentication
Closes-Bug: #123456
Change-Id: I4946a16d27f712ae2adf8441ce78e6c0bb0bb657
{% endcodeblock %}

Note that in most cases the Change-Id line should be automatically added by a Gerrit commit hook that you will want to install. See Project Setup for details on configuring your project for Gerrit. If you already made the commit and the Change-Id was not added, do the Gerrit setup step and run:

{% codeblock %}
git commit --amend
{% endcodeblock %}

The commit hook will automatically add the Change-Id when you finish amending the commit message, even if you don't actually make any changes.

Make your changes, commit them, and submit them for review:

{% codeblock %}
git commit -a
git review
{% endcodeblock %}

*Caution: Do not check in changes on your master branch. Doing so will cause merge commits when you pull new upstream changes, and merge commits will not be accepted by Gerrit.*

Prior to checking in make sure that you run "[tox](http://testrun.org/tox/latest/)".



### 1.4.2 Review ###

### 1.4.3 Work in Progress ###

### 1.4.4 Long-lived Topic Branches ###

### 1.4.5 Updating a Change ###

### 1.4.6 Add dependency ###