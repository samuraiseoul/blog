---
layout: post
title:  "Autodeploying a git repo with cron monitoring"
date:   2017-05-17
categories: autodeploy git cron bash script github deploy
tags: jekyll github deploy
---

As I was building my [slackbot](https://github.com/samuraiseoul/starbot), I was
always running it locally. The problem was that I had two computers where I could
be deving on it, and occasionally I would have two instances running.
This was a pain as it would make two post in the slack room where it was being
used, or if both computers were off, no one could use it. My friend was nice enough
to give me ssh access to his raspberry pi so I could run my bot there and not have
to buy server space or leave a dedicated computer running at all times. And while
it was a great solution for where to run the bot to prevent the issue of too many instances
or no instances, this presented a new problem, deploying. Having to ssh in and
go through the deploy process every time I knew would eventually get too annoying.
I knew I needed some kind of continuous integration and auto deploy process, but I didn't want
to pay for any services, nor for any servers to get something like Jenkins set up.
So I decided to build my own solution using cron and a simple script.

## Shell Scripting
Like any good developer, as I was preparing to write the shell script I had to
lay out the steps needed to make it work. I knew I needed to figure out if there
were any new commits, if so, deploy. Naturally, the next step was deciding what
a deploy consisted of. Deploying was as simple as killing the current running instance,
gitting the new code, building, and running again in the background. The script
below fetches the repo and greps the status for the word "behind", then simply checks
if it was found, and if so runs the commands needed to deploy.

```
#!/bin/bash
cd ~/starbot
git fetch
git=$(git status -uno | grep "behind")
if [[ ! $git == "" ]]; then
        ps -ef | grep "starbot" | awk '{print $2}' | xargs kill -9
        git pull
        mvn clean package
        java -jar target/*-dependencies.jar > /tmp/starbot.log &
fi
```

If you wanted to use this command yourself, just be sure to change the 'cd' to the
correct directory if it's different from where the script lives, otherwise delete
the line. After, change the commands in the if block to be specific to your
application's deploy process and then set cron to run the script.

## Cron
Cron jobs run a process at a specified interval. In this case, I want it to check
for changes and redploy if they are there once a minute. To add a new cron job first
run ``` crontab -e ```. It will most likely ask you to choose an editor to continue.
After you have the file opened simply add the following line ``` */1 * * * * bash /home/scott/monitor.sh ```.

The ``` */1 * * * * ``` specifies an interval to run at, in our case one minute.
you can also specify to run every hour on the 15 minute mark, or over the course
of weeks, days, or months. I found [this tutorial](https://www.computerhope.com/unix/ucrontab.htm)
to be quite useful when I was trying to figure the intervals out as well as cron
in general.

## Conclusion
Once you have your cron job set to run, you're set, just push to the repo and
it will poll at the specified interval and redeploy for you! I use this method
to auto deploy my blog now as well in combination with the deploy script I wrote
about in another [blog post]({{ site.baseurl }}{% post_url 2017-04-14-jekyll-deploy %}).
Thanks for reading.
