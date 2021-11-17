---
layout: post
author: lynn
tags: gcloud authentication authorization make
---

* T0C
{:toc}

---
<!--
<a href="" target="_blank"></a>

&nbsp;

<figure><center><img src="/assets/images/" style="width:100%">
<figcaption></figcaption></center></figure><br>

[another post]({% post_url 2021-11-03-journal %}))
-->

Working as a cloud or devops engineer for a consulting company means you do a lot of context switching. Specifically, switching between your various AWS profiles, GCP projects, or Azure tenants. You've got different accounts under your main company such as your personal sandbox, your coworker's sandbox, the main business infrastructure project, etc. You've got multiple projects for your variety of customers (past and present), and then of course there are different isolated environments for each one. The list goes on! 

This post aims to showcase how I handle the creation and context switching of my Google Cloud Platform projects via `gcloud`.

## Creating
For anyone who's worked in GCP, there are multiple commands you need to run in order to get started. Some are long and difficult to remember. There is the option to run `gcloud init`. It'll run through an interactive setup which'll authorize gcloud to access GCP with the credentials you pass in, as well as set up a new configuration or update an existing one. But that's not something you would want to run every time you need to create a project and set its corresponding properties. For the non-interactive setup, you'd run the following commands for each new configuration:

```bash
gcloud config configurations create $CONFIG_NAME
gcloud config configurations activate $CONFIG_NAME
gcloud auth login
gcloud config set project $PROJECT_NAME
gcloud config set compute/zone $ZONE
```

Some masochist or engineer who enjoys toil would be running `gcloud --help`, or scrolling up on their shell's command history, or googling for the commands every single time they needed to set up a new configuration. Or maybe not if they memorized these commands after running them the first time! But my memory is just a smidge better than a goldfish's and I revel in enough pain at work as is. So let me introduce my <a href="http://www.sis.pitt.edu/mbsclass/tutorial/advanced/makefile/whatis.htm" target="_blank">makefile</a>!

```make
EMAIL=lynn.dong@company.com  # set default variable

gcc:
	gcloud config configurations create $(GCP_PROJECT_ID) && gcloud config set project $(GCP_PROJECT_ID) && gcloud config set account $(EMAIL)
	# example command: make gcc GCP_PROJECT_ID=bananabread EMAIL=lynn.dong@bread.io
```

Now instead of those 5 long commands, a new project creation would require just this:

```bash
make gcc GCP_PROJECT_ID=$PROJECT --directory $PATH_TO_MAKEFILE
```

- to avoid changing into your `makefile` directory every time you run this command, pass in the directory and this command will be totally reusable for years to come!
- defaults
  - your configuration is set to the name of the GCP project ID
  - the email is set to what's defined in the `makefile` or you can pass in a different one by adding `EMAIL=$EMAIL`

One command to create them all! 😊

After this initial setup, the process only requires:
1. typing `make` in your terminal
1. doing a reverse search of your command history
1. passing in the GCP project name!


## Switching

Alright, so switching configurations normally looks a bit like this:

```bash
gcloud config configurations list
gcloud config configurations activate $PROJECT_ID
```

Not as bad as creating but I know plenty of engineers who would struggle to remember and accurately type this out. I see you guys out there running your commands with all your typos and wondering why it didn't execute properly!! 👀

There are 2 ways you can switch configurations more easily. You can use environment variables or aliases.


### 1. Environment Variables

Set your default configuration in your shell's configuration file.

```bash
export CLOUDSDK_ACTIVE_CONFIG_NAME=$PROJECT_ID
```

Use a different configuration and override the default by exporting a different value for the variable on the command line! I typically just begin typing out `export CLOUD` and scroll up in my command history until I find the project I'm looking for.

### 2. Aliases

I'm sure you know about aliases but if you didn't set this up already, here are what mine look like:

```bash
# GCloud
alias gccl="gcloud config configurations list"
alias gcca="gcloud config configurations activate"
alias gccd="gcloud config configurations delete"
alias gcal="gcloud auth list"
alias gal="gcloud auth login"
alias gaal="gcloud auth application-default login"
```

So now, switching configurations should look like this:

```bash
gccl
gcca $PROJECT_ID
```

I'm always trying to improve my workflows. If there's a better way to do these things or if you ended up incorporating this into your own workflow, <a href="https://www.linkedin.com/in/lynndong/" target="_blank">let me know</a>! I hope I've saved you some time and pain and I'll see you in the next post!
