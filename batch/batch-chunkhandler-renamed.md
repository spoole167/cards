---
id: batch-chunkhandler-renamed
tier: 1
tier_label: Won't Build
title: Spring Batch ChunkHandler Renamed to ChunkRequestHandler
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: batch
---

ChunkHandler renamed to ChunkRequestHandler. Fails to compile on Boot 4.0.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/BatchRenamingUsage.java:[4,65]
  error: cannot find symbol
    symbol: class ChunkHandler
    location: package org.springframework.batch.integration.chunk
```

## What Changed {.what-changed}

<code>org.springframework.batch.integration.chunk.ChunkHandler</code> has been renamed to <code>ChunkRequestHandler</code>.

## Why {.why-changed}

The rename of <code>ChunkHandler</code> clarifies its role in the request/response model of remote chunking.

## The Fix {.diffs}

```diff-card
# // ChunkHandler rename
@@removed
import org.springframework.batch.integration.chunk.ChunkHandler;
// ...
ChunkHandler handler = ...;
@@added
import org.springframework.batch.integration.chunk.ChunkRequestHandler;
// ...
ChunkRequestHandler handler = ...;
```

## How To Fix {.fixes}

#### Rename ChunkHandler to ChunkRequestHandler.

Update the import and all references. Only the name changed; the interface contract is the same.

## Scope Check {.scope-check}

Search for <code>ChunkHandler</code> (not <code>ChunkRequestHandler</code>) across all Java/Kotlin sources. Affects remote chunking setups.


## Further Info {.further-info}

ChunkHandler belongs to the remote-chunking integration module. Verified in the same spring-break test module as the companion JobStep.setJobLauncher removal: the two renames fail in the same compile run but are otherwise unrelated.

## Links {.footer-links}

- [spring-break module: batch-chunkhandler-renamed](https://github.com/spoole167/spring-break/tree/main/batch-chunkhandler-renamed)

- [Spring Batch 6.0 Migration Guide](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-6.0-Migration-Guide)

