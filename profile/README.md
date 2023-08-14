# The rant

None of the *industry standard* solutions for logging are not fit for human consumption
and definitely not for Kubernetes.

What is wrong with Fluent (Bit)

* Pipeline dies with DNS resolution error https://github.com/fluent/fluent-bit/issues/5654 https://github.com/fluent/fluentd/pull/2808
* Retry logic is not enabled by default and has no effect for the DNS cornercase

What is wrong with ElasticSearch

* It was designed for webshop search, not for logging
* Once JSON parsing is enabled *all* keyspace gets indexed
* It's not possible to mix data types for same JSON keys,
  field that gets indexed as integer causes subsequent messages with other
  datatypes to fail
* Memory usage skyrockets due to all fields being indexed,
  limiting memory usage causes search to be practically useless

What is wrong with Graylog:

* Graylog journal inherently introduces bottleneck in the logging system
  that needs to be then scaled again.
* Graylog ingestor gladly accepts all messages, some of which can then be
  rejected during submission to Elastic. This goes on until Graylog journal is full.
* No usable Prometheus metrics

What is wrong with Kibana:

* Kibana starts up Chromium to render screenshots by default,
  there is no way to turn it off

Overall all of the above:

* Do not have Prometheus metrics builtin
* Even via suitable exporter there is no way to get usable metrics
  * How many messages failed to parse as JSON
  * How many messages failed to insert to ElasticSearch
  * No alerting rules
* Are not fire and forget, die for whatever random reasons
* Are a long endeavour to get running even as a MVP
  
# Solution

* Something that works for 80% of the cases out of the box in Kubernetes environment
* Has metrics and Prometheus operator alerting rules built-in
* Best effort compliance with ECS, because well Elastic itself is unable
  to make it's components conform to ECS
  https://github.com/elastic/ecs-logging-python/issues/48#issuecomment-1274328772


# Guidance for application developers

* Follow the ECS schema for most part

# Guidance for application administrators

* Don't keep debug, notice and info levels enabled by default
