---
layout: post
title: Oh Elasticsearch!
date: 2016-01-25 18:27 -0800
categories: tech
tags: linux,ELK,elasticsearch,logstash
---

Working on fixing an issue with an ELK stack today and I realized that the folks at [Elastic](http://elastic.co) have some pretty interesting names for the shards in the database.

So far I've seen:

- Peregrine
- Obliterator
- Amiko Kobayashi
- Cassie Lang
- Vakume
- Mist Mistress
- Arides
- Roughhouse
- Omega the Unknown
- Martinex


There were a few more; Elasticsearch generates a new shard name every time the service is restarted. I'm almost tempted to restart the service a few more times, just to see what it'll come up with. 

Also, it'd be great if the commands to check the integrity of an index were, you know, simpler.