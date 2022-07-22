

# Josh's notes for this course

## Friday, July 22

_Start: 4:30p_

I'd been having trouble deciding how/where to keep written notes for this.

Since starting this course, I've dug more into Sidekiq for work at Homebot, like:

- [ws-39 rework Sentry logging for Sidekiq jobs]()
- [ws-38 move some sidekiq jobs out of system queue]()

and that ended up with some blog posts like:

- sidekiq basics
- https://josh.works/tags#sidekiq

I don't plan on pushing all this data in this repo anywhere, but storing my notes in a README is perfect.

So:

```shell
$ j sidekiq
# same as cd /Users/joshthompson/personal/sidekiq_in_practice/
$ git init
$ atom .
$ cp README.md readme_original_version.md
# write this message
$ ga . # thinks "i bet nate added this repo contents to the .gitignore, because he's thoughtful like that"
$ cat .gitignore
$ gc -m "setup repo or whatever"
```

### Lesson 1

I've already read this thing like twice, looked at the solution, and _still_ barely know what I'm looking at. It's daunting for lesson 1.

I read `lab/lab_1/README.md`, and `chapter_0.md`, `chapter_1.md`, and feel way over my head.

I fumble my way through getting the server running for lab 1. It's been a hell of a week, I'm tired.

```shell
cd lab/lab_1
foreman # nothing happened. _squints eyes_
foreman start # lots of logs print out, leftover messages from other sidekiq work I was doing for Homebot

# I think I need redis running... was it already?
ps aux | grep redis # yeah, redis-server seems up, on port :6379
```

reading notes in the `lab 1` `readme`:

> every lab assumes redis running on localhost:6379

my redis server is running on a diff port, so lets kill it:

```shell
pkill redis-server
```
He mentions `pkill -9 sidekiq`, lemme look and see if I have sidekiq stuff running:

```
ps aux | grep sidekiq
joshthompson     88025   0.0  0.5  4589992  80056 s007  S+   Tue02PM   2:32.85 sidekiq 6.5.1 email_basics [0 of 10 busy]
```

that `email_basics` reference is a diff rails app i spun up to test mailers, so I wonder what this all means.

```shell
$ foreman start # fails again cuz no redis running, in a huge and exciting stack trace
$ redis-server # open a new tab cuz this blocks what you just ran it in
$ foreman start
```

Now I'm seeing output from that Redis exercise - utilization counts and stuff.

I'm removing `lab` from `.gitignore` because I wish I had VCS on `logger_thread.rb` - i copied the solution into it, so i don't have the original.

I was getting super messy output from my sidekiq server, i did `pkill sidekiq` and `foreman start` and all is _finally_ looking groovy.

#### Good starting point:

```shell
$ pkill sidekiq
$ redis-cli shutdown

```

https://stackoverflow.com/questions/6910378/how-can-i-stop-redis-server
