---
title: Add annotations
description: >
  TK
menu:
  influxdb_cloud:
    name: Add annotations
    parent: Visualize data
weight: 101
---

An annotation is a note with an associated time range.
For a "single point" annotation, the start time and end time are the same.
A stream allows for logical, high level grouping of annotations.
Further annotation specification can be performed by adding "stickers" (also known as tags).
These stickers and streams can then be used to filter which annotations are returned;
specifying none returns all annotations, specifying only some stickers will return annotations from any stream with matching stickers, specifying
only a stream will return all annotations inside that stream, etc...

Adds an API to read/write annotations.

Each annotation belongs to a single stream.
Annotations also can be filtered by a time window, which is useful for visualization layers to only retrieve annotations that fit a displayed amount of time.

Streams can be described, as a stream is represented typically by a single word string.
Whenever an annotation is written to a stream, that stream is gets "updated" with the current time
(this seems like a placeholder for future use cases, as it's value is undetermined).

#### Add an annotation

1.

#### Remove an annotation
