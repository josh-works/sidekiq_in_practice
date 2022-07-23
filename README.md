

# Josh's notes for this course

## Friday, July 22

_Start: 4:30p_

I'd been having trouble deciding how/where to keep written notes for this. In many ways, I feel _very very dumb_ so I need a place where I can speak very freely about how difficult this is for me. I feel like everyone else thinks I know way more about this than I feel, so...

Since starting this course, I've dug more into Sidekiq for work at Homebot, like:

- [ws-39 rework Sentry logging for Sidekiq jobs]()
- [ws-38 move some sidekiq jobs out of system queue]()

and that ended up with some blog posts like:

- sidekiq basics
- https://josh.works/tags#sidekiq

I still feel like I know almost nothing about all this stuff, and I slightly smarter version of me should be an expert in it right now, based on how much time I've spent with it. So I'm finally plunging into some courses, seeing if for the love of all that is good in the world, I can wrap my head around Sidekiq.

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
$ foreman help start # reads
$ redis-server # in a different tab
$ redis-cli monitor $ in _another_ different tab
$ foreman start -t 10
```

https://stackoverflow.com/questions/6910378/how-can-i-stop-redis-server

After getting `foreman start -t 10` working, and watching the rather insane output from `redis-cli monitor`, and then watching it stop after a few seconds... is interesting. Redis seems _fast_

I see lots stuff in there from this `enqueuer` and `worker` (reading `Procfile` via `cat Procfile`)...

So the "lab" was something like this:

```ruby
Thread.new do
  while true
    workers = # figure this out
    num_processors_busy = # figure this out
    num_processors_total = # figure this out
    util_percent = # figure this out

    default_queue_latency_in_seconds = Sidekiq::Stats# Maybe in Sidekiq::API?

    retry_queue_latency_in_seconds = Sidekiq::  # Maybe in Sidekiq::API?

    Sidekiq.logger.warn(Sidekiq::CLI.r + "*" * 20 + Sidekiq::CLI.reset)
    Sidekiq.logger.warn(
      "Utilization (instantaneous): #{util_percent}% | " +
      "Default latency: #{default_queue_latency_in_seconds} | " +
      "Retries latency: #{retry_queue_latency_in_seconds}"
    )
    Sidekiq.logger.warn(Sidekiq::CLI.r + "*" * 20 + Sidekiq::CLI.reset)
    sleep 1
  end
end
```

OK. No idea what I am looking at. The answer, in case it makes sense to you:

```ruby
Thread.new do
  while true
    workers = Sidekiq::CLI.instance&.launcher&.manager&.workers
    num_processors_busy = workers&.count(&:job) || 0
    num_processors_total = workers&.size || 0
    util_percent = num_processors_busy.to_f / num_processors_total * 100

    default_queue_latency_in_seconds = Sidekiq::Stats.new.default_queue_latency.round # Maybe in Sidekiq::API?

    retry_queue_latency_in_seconds = Sidekiq::Queue.new("retry").latency # Maybe in Sidekiq::API?

    Sidekiq.logger.warn(Sidekiq::CLI.r + "*" * 20 + Sidekiq::CLI.reset)
    Sidekiq.logger.warn(
      "Utilization (instantaneous): #{util_percent}% | " +
      "Default latency: #{default_queue_latency_in_seconds} | " +
      "Retries latency: #{retry_queue_latency_in_seconds}"
    )
    Sidekiq.logger.warn(Sidekiq::CLI.r + "*" * 20 + Sidekiq::CLI.reset)
    sleep 1
  end
end
```

Makes no sense to me, but lets plunge onward. We got it running, i'll explore the API when it makes sense, hopefully soon. (I've played with these objects a bit, already, in other contexts. They still don't really make much sense to me.)

Re-reading chapter 0.md and chapter 1.md, pulling some quotes:

> All errors are wasted capacity: wasted work that could have been spent doing something productive. Keeping errors low means we keep our server capacity available for useful work.
>
> Useful error metrics to track for Sidekiq include:
>
> * Size of the retry queue
> * Size of the dead queue
> * Redis connection errors
>
> Error metrics should be as low as possible, so that capacity can be used for useful work.

- Homebot retry queueu: 47
- hb dead queueu: 9,999
- redis connection errors: 42

we currently have 3.5 million enqueued jobs,  55k marketing emails going out the door.

Nate says Sidekiq is some of the best written Ruby he's ever seen, so lets see what we're working with:

```shell
$ EDITOR=atom bundle open sidekiq
```

Followed along the rest of chapter_0. Re-skimming chapter 1, then calling it a night

> We can measure the resource efficiency of a background job processing system by dividing its job throughput (jobs per second) by its resource consumption (maybe in terms of vCPUs in the job fleet, or more helpfully, dollars spent). I prefer to think about this in terms of cost. We might track a metric here like "jobs per dollar": how many jobs we process per dollar of server spending.

6:30p, 2 hr session, but it involved a phone call, some breaks. Just chillin' in the office.

Next time, I'll pick up with `chapter_2.md`
