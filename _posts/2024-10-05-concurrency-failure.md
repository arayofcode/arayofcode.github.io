---
layout: post
title: Concurrency Limits and Performance
---

## Lessons learnt from building a system with unbound concurrency

I read [this blog](https://netflixtechblog.medium.com/performance-under-load-3e6fa9a60581) by Netflix Engineering team this morning, which reminded me of a challenge I faced on, working on a project in my first role. The blog shares Netflix's approach to managing system performance and availability under heavy workload through adaptive concurrency limits. Their algorithm dynamically adjusts the number of concurrent requests their system would handle, finding the right number that maximised their throughput while maintaining low latency. Reading the article helped me understand why my application failed, and thought it would be good to share what went wrong, and how we could've improved it.

[Disclaimer: Much of the background I'll share has been paraphrased from the blog to provide better context.]

## Background on Scalability and Availability

You build a system with concurrent core service that can handle multiple requests. Given workload can be high, if number of incoming requests are more than what the service can handle, the excess requests begin failing. To mitigate this, you add a message queue as well. The queue ensures that even when each request takes different time process, it would still be handled. This seems reasonable, but isn't enough.

The core service can handle a limited number of requests based on the resource capacity. Moreover, the time taken to process each request is variable too. With an unbound message queue, it is possible to reach a situation where number of incoming requests is more than the number of handled requests. Your message queue grows further, leading to an increase in latency until the core service runs out of resources and crashes, or the requests begin timing out. If unchecked, this potentially impacts everything downstream.

For the message queue, you can experiment with rate limiting, or dead-letter queue (I only know that's a solution, but never used one before). For concurrency, you can consider scaling up, horizontally, or diagonally, however, this comes with its own costs. This is why at some point, finding the right concurrency limit is needed. We were stuck here too.

## The Challenge

We faced a similar challenge while building a security scanning solution, expected to perform scans of tens of thousands of repositories. Our core service, a PHP script running on a Google Cloud Instance, would clone repositories, run a shell command to scan, then push scan results to another service, which processed those final results. It was a naive approach where we brought in concurrency by running those scans as background jobs. The code that did it was very similar to this:

```php
shell_exec('scanner clone_path &')
```

The ampersand ( & ) at the end pushed the process to background and that was it. We didn't add any limitation on how many background scans it would create. This was catastrophic: the first couple of runs failed because our instance resources were fully utilised and it crashed. The mess-up: a simple for-loop, going through all repos and running those scans. Because there was no wait, we created a lot more jobs than what our instance could handle, and it failed.

```php
foreach ($repo_list as $repo) {
  $path = clone_repo($repo)
  shell_exec('scanner clone_path &')
  ...
}
```

The fix was simple: limit the number of concurrent scans and scale up. However, finding the right number was challenging. A single scan would take ~2 minutes for a repo on an average case. A good chunk of repos, it would go 6-7 minutes, and occasionally timeout after ten minutes. You choose the wrong number, and suddenly the scan time would be a couple days in the worst case. Not having much idea of how to do it, plus working under deadlines, we went to hit and trials to determine the right concurrency limit.

## Learnings

With enough hit-and-trial, we settled on a number that reduced our scan size to ten hours. This seemed acceptable as the scans only needed to run once every few days. We used this solution for about 1.5 years before switching to an enterprise tool that offered this as one of their features. Despite the replacement, it was a great learning experience building this tool. I enjoyed reading Netflix team's blog as it shared an approach I hadn't thought of. However, for our specific tool, given scale was nearly constant (number of repos wouldn't really change that much), a simpler way to implement dynamic concurrency limit would've been to monitor the instance resources. Basically, run the main service (a script) as a cron job that checks if instance has resources available under certain limit. If available, scan another repo otherwise do nothing.

In hindsight, an better approach would've been to perform scans through Google Kubernetes Engine, which could've ensured presence of no data once scans finished. This could've scaled a lot better than how we did it, but it comes with a trade-off: for each scan, you need to clone the repository. Using the instance and cloning each repo there meant subsequent scans would take less time as we could use git pull .