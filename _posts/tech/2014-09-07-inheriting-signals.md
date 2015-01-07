---
layout: article
title:  "Inheriting Signals in *nix"
date:   2014-09-07 19:30:00
categories: tech
tags: [linux, signals]
comments: true
---

Recently I was working on managing the lifecycle of some ruby daemons and found some interesting things I didn't know
about the [linux signals](http://unixhelp.ed.ac.uk/CGI/man-cgi?signal+7). Signals in Linux are a way of communicating
between processes by sending asynchronous notifications to the process. You can send signals like `SIGTERM` asking the
process to exit or `SIGPHUP` to indicate that the terminal/controlling process was closed/terminated.

When a process receives a signal a default action happens, unless the process decides to handle it differently.
For example, a daemon usually catches the `SIGHUP` and ignores it since it is expected to continue to run even after the
terminal was closed. However, there some signals that you cannot catch. Signals `SIGKILL` and `SIGSTOP` cannot be
trapped, blocked or ignored.

Another thing about signals is that the when you create a child process using
[fork](http://www.ruby-doc.org/core-2.1.2/Process.html#method-c-fork), it inherits the signals from its parent.
Let's take an example:

{% highlight ruby %}
Signal.trap('TERM') do
  puts "Will exit in 1 sec"
  sleep 1
  puts "Exiting.."
  exit 0
end

# some important code

p = fork do
  loop do
    # something else that is important
  end
end

Process.kill('TERM', p)
=> "Will exit in 1 sec"
=> "Exiting.."
{% endhighlight %}

So as you can see here, I defined a trap for signal `SIGTERM` in the parent process and forked a child. When I sent a
signal `SIGTERM` to the child, it ran the same signal handler defined on its parent. You will see this behaviour only
when you fork the child and don't [exec](http://www.ruby-doc.org/core-2.1.2/Process.html#method-c-exec) in the child.

{% highlight ruby %}
Signal.trap('TERM') do
  puts "Will exit in 1 sec"
  sleep 1
  puts "Exiting.."
  exit 0
end

# some important code

p = fork do
  exec("while true; do echo hi; sleep 5; done")
end

Process.kill('TERM', p)
=> 1
{% endhighlight %}
