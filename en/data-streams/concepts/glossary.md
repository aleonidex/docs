---
title: Terms and definitions in {{ yds-full-name }}
description: In this article, you can find the terms and definitions for {{ yds-name }}.
---

# Terms and definitions in {{ yds-name }}

## Data stream {#stream-concepts}

A data stream is a named set of messages. Messages in {{ yds-name }} are written and read via streams. Data streams are created based on {{ ydb-full-name }} and are stored in databases.

## Shard {#shard}

To enable horizontal scaling, a stream is divided into shards which are units of concurrency. Each shard has a limited [throughput](#shard-thoughput).

{% note info %}

As for now, you can only reduce the number of shards in a stream by deleting and recreating a stream with a smaller number of shards.

{% endnote %}

You can configure a data stream to increase the number of shards as the write speed into the stream increases. For more information, see [autopartitioning in the {{ ydb-short-name }} documentation]({{ ydb.docs }}/concepts/topic#partitioning).

### Shard key {#partition-key}

A shard key is specified for each [message](#message) while writing it to a stream. Using the key hash, the message is mapped to a certain shard inside the stream.

{% note warning %}

When updating the number of stream shards, their distribution across the key hash space changes, too. Messages that were written before the number of shards was updated remain in the same shards and the same sequence. New messages are distributed over a new number of shards.

{% endnote %}

### Shard throughput {#shard-thoughput}

Each shard has a limited user-defined throughput. The maximum data write speed per shard is 1 MB/sec and the maximum data read speed is 2 MB/sec.

## Message {#message}

A message is the minimum atomic unit of user information.

It consists of a [body](#message-body) and additional system properties.

### Message body {#message-body}

The body of a message is a set of bytes. {{ yds-name }} does not interpret the message body in any way.

### Message sequence number {#sequence-number}

When writing data to a stream, each message is assigned a sequence number. Message sequence numbers are unique within a single shard and increase sequentially.

### Message retention period {#retention-time}

The message retention period is set for each stream. After it expires, messages are automatically deleted.

## Consumers {#consumers}

Consumers are applications that get data from {{ yds-name }} and process it. All consumers share the [total quota](limits.md) for data reads.

## Consumer groups {#extended-consumers}

In some cases, the common [quota]({{ link-console-quotas }}) model for all consumers is too restrictive, so each consumer gets a guaranteed read speed. Consumers with a guaranteed read speed are called _consumer groups_.
