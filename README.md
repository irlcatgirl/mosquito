# mosquito

Mosquito is a generic background job runner written specifically for Crystal. Significant inspiration from my experience with the successes and failings of the Ruby gem Sidekiq.

Mosquito currently provides these features:
- Delayed execution
- Scheduled execution
- Job Storage in Redis
- Crystal hash style `?` methods for parameter getters which return nil instead of raise
- Automatic rescheduling of failed jobs
- Progressively increasing delay of failed jobs
- Dead letter queue of jobs which have failed too many times

Current Limitations:
- Job failure delay, maximum retry count, and several other variables cannot be easily configured.
- Redis functions not all atomic. There is the potential for multiple instances or fibers running background jobs to interfere with each other, resulting in duplicate task executions. Missed executions are unlikely.
- Visibility into the job queue is difficult and must be done through redis manually.

![](https://cdn.shopify.com/s/files/1/0242/0179/products/amber1_1024x1024.png?v=1455409061)

## Project State

Updated 2018-01-08

> Sufficient working beta. No functionality is tested, but it all seems to be working in manual tests. Use in a production environment at your own risk, and please open issues and feature requests. 
>
> I'm using it in a production environment.

## Installation

Add this to your application's `shard.yml`:

```yaml
dependencies:
  mosquito:
    github: robacarp/mosquito
```

and run `shards install` from your application root.

## Usage

### Step 1: Require Mosquito in your application loader

```crystal
require "mosquito"

# Your jobs folder
require "jobs/*"

# In a second application target, or behind some other switching or multi-fiber interface, run tasks:
Mosquito::Runner.start
```

### Step 2: Define a queued job

```crystal
class PutsJob < Mosquito::QueuedJob
  params(message : String | Nil)

  def perform
    puts message
  end
end
```

### Step 3: Trigger that job

```crystal
PutsJob.new(message: "Hello Background Job World!").enqueue
```

### Success

```
> crystal run src/worker.cr
2017-11-06 17:07:29 -0700 - Mosquito is buzzing...
2017-11-06 17:07:51 -0700 - Queues: puts_job
2017-11-06 17:07:51 -0700 - Running task puts_job<mosquito:task:1510013271686:246> from puts_job
2017-11-06 17:07:51 -0700 - [PutsJob] ohai background job
2017-11-06 17:07:51 -0700 - task puts_job<mosquito:task:1510013271686:246> succeeded, took 0.0 seconds
```

## Periodic Jobs

Periodic jobs run according to a predefined period. Because the scheduler has no application context, they can have no inputs. By design, periodic jobs are meant to be self sufficient.

This periodic job:
```crystal
class PeriodicallyPutsJob < Mosquito::PeriodicJob
  run_every 1.minute

  def perform
    emotions = %w{happy sad angry optimistic political skeptical epuhoric}
    puts "The time is now #{Time.now} and the wizard is feeling #{emotions.sample}"
  end
end
```

Would produce this output:
```crystal
2017-11-06 17:20:13 -0700 - Mosquito is buzzing...
2017-11-06 17:20:13 -0700 - Queues: periodically_puts_job
2017-11-06 17:20:13 -0700 - Running task periodically_puts_job<mosquito:task:1510014013586:954> from periodically_puts_job
2017-11-06 17:20:13 -0700 - [PeriodicallyPutsJob] The time is now 2017-11-06 17:20:13 -0700 and the wizard is feeling skeptical
2017-11-06 17:20:13 -0700 - task periodically_puts_job<mosquito:task:1510014013586:954> succeeded, took 0.0 seconds
2017-11-06 17:21:14 -0700 - Queues: periodically_puts_job
2017-11-06 17:21:14 -0700 - Running task periodically_puts_job<mosquito:task:1510014074000:85> from periodically_puts_job
2017-11-06 17:21:14 -0700 - [PeriodicallyPutsJob] The time is now 2017-11-06 17:21:14 -0700 and the wizard is feeling optimistic
2017-11-06 17:21:14 -0700 - task periodically_puts_job<mosquito:task:1510014074000:85> succeeded, took 0.0 seconds
2017-11-06 17:22:15 -0700 - Queues: periodically_puts_job
2017-11-06 17:22:15 -0700 - Running task periodically_puts_job<mosquito:task:1510014135000:987> from periodically_puts_job
2017-11-06 17:22:15 -0700 - [PeriodicallyPutsJob] The time is now 2017-11-06 17:22:15 -0700 and the wizard is feeling political
2017-11-06 17:22:15 -0700 - task periodically_puts_job<mosquito:task:1510014135000:987> succeeded, took 0.0 seconds
```

## Integrating with Amber:

And add this file to your `src/` directory:

```crystal
require "amber"
require "mosquito"

require "../config/application"

require "./models/**"
require "./mailers/**"
require "./handlers/**"

require "./jobs/**"

Mosquito::Runner.start
```

Then simply `crystal run src/worker.cr`.

## Contributing

Contributions are welcome. Please fork the repository, commit changes on a branch, and then open a pull request.

## Contributors

- [robacarp](https://github.com/robacarp) robacarp - creator, maintainer
