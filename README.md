# catnip

[![CircleCI](https://circleci.com/gh/mewna/catnip.svg?style=svg)](https://circleci.com/gh/mewna/catnip)
[![powered by potato](https://img.shields.io/badge/powered%20by-potato-%23db325c.svg)](https://mewna.com/)
![GitHub tag (latest by date)](https://img.shields.io/github/tag-date/mewna/catnip.svg?style=popout)


A Discord API wrapper in Java. Fully async / reactive, built on top of
[vert.x](https://vertx.io). catnip tries to map roughly 1:1 to how the Discord 
API works, both in terms of events and REST methods available.

catnip is part of the [amyware Discord server](https://discord.gg/yeF2HpP)

Licensed under the [BSD 3-Clause License](https://tldrlegal.com/license/bsd-3-clause-license-(revised)).

## Installation

[Get it on Jitpack](https://jitpack.io/#com.mewna/catnip)

Current version: ![GitHub tag (latest by date)](https://img.shields.io/github/tag-date/mewna/catnip.svg?style=popout)

### Can I just download a JAR directly?

No. Use a real build tool like [Maven](https://maven.apache.org/) or [Gradle](https://gradle.org/).

### Javadocs?

[Get them here.](https://mewna.github.io/catnip/docs)

## Features

- Automatic sharding
- Very customizable - you can write [extensions](https://github.com/mewna/catnip/blob/master/src/main/java/com/mewna/catnip/extension/Extension.java)
  for the library, as well as [options](https://github.com/mewna/catnip/blob/master/src/main/java/com/mewna/catnip/CatnipOptions.java)
  for most anything you could want to change. See `EXTENSIONS.md` for more.
- Modular - REST / shards can be used independently. See `MODULAR_USAGE.md` for more.
- Customizable caching - Can run with [no cache](https://github.com/mewna/catnip/blob/master/src/main/java/com/mewna/catnip/cache/NoopEntityCache.java),
  [partial caching](https://github.com/mewna/catnip/blob/master/src/main/java/com/mewna/catnip/cache/CacheFlag.java),
  or [write your own cache handler](https://github.com/mewna/catnip/blob/master/src/main/java/com/mewna/catnip/cache/EntityCacheWorker.java).
  See `CACHING.md` for more.
- Asynchronous cache accesses.
- You can disable individual events.
- You can disable all events, and handle gateway events directly.
- Customizable ratelimit/session data handling - wanna store your 
  [sessions/seqnums](https://github.com/mewna/catnip/blob/master/src/main/java/com/mewna/catnip/shard/session/SessionManager.java) 
  and [REST ratelimit data](https://github.com/mewna/catnip/tree/master/src/main/java/com/mewna/catnip/rest/ratelimit)
  in Redis, but [gateway ratelimits](https://github.com/mewna/catnip/blob/master/src/main/java/com/mewna/catnip/shard/ratelimit/Ratelimiter.java)
  in memory? You can do that!
- [Customizable shard management](https://github.com/mewna/catnip/blob/master/src/main/java/com/mewna/catnip/shard/manager/ShardManager.java)

## Basic usage

This is the simplest possible bot you can make right now:

```Java
final Catnip catnip = Catnip.catnip("your token goes here");
catnip.on(DiscordEvent.MESSAGE_CREATE, msg -> {
    if(msg.content().startsWith("!ping")) {
        msg.channel().sendMessage("pong!");
    }
});
catnip.connect();
```

catnip returns `CompletionStage`s from all REST methods. For example,
editing your ping message to include time it took to create the
message:

```Java
final Catnip catnip = Catnip.catnip("your token goes here");
catnip.on(DiscordEvent.MESSAGE_CREATE, msg -> {
    if(msg.content().equalsIgnoreCase("!ping")) {
        final long start = System.currentTimeMillis();
        msg.channel().sendMessage("pong!")
                .thenAccept(ping -> {
                    final long end = System.currentTimeMillis();
                    ping.edit("pong! (took " + (end - start) + "ms)");
                });
    }
});
catnip.connect();
```

You can also create a catnip instance asynchronously:

```Java
Catnip.catnipAsync("your token here").thenAccept(catnip -> {
    catnip.on(DiscordEvent.MESSAGE_CREATE, msg -> {
        if(msg.content().startsWith("!ping")) {
            msg.channel().sendMessage("pong!");
        }
    });
    catnip.connect();
});
```

Also check out the [examples](https://github.com/mewna/catnip/tree/master/src/main/example/basic) for Kotlin and Scala usage.

### Modular usage

catnip supports being used in REST-only or shards-only configurations. The nice thing about catnip
is that using it like this is **exactly the same** as using it normally. The only difference is
that to use catnip in REST-only mode, you don't call `catnip.connect()` and use 
`catnip.rest().whatever()` instead. 

### Custom event bus events

Because vert.x is intended to be used in clustered mode as well as in a single-node configuration,
emitting events over the built-in event bus requires registering a codec for the events that you
want to fire. If you have an event class `MyEvent`, you can just do something to the effect of
```Java
eventBus().registerDefaultCodec(MyEvent.class, new JsonPojoCodec<>(this, MyEvent.class));
```
where `JsonPojoCodec` is `com.mewna.catnip.util.JsonPojoCodec` and is safe to use.

## Useful extensions

- `catnip-voice` - Voice support for your catnip bot. 
  https://github.com/natanbc/catnip-voice
- `catnip-utilities` - Some extensions for typesafe commands, event waiters, reaction menus, 
  and more. https://github.com/queer/catnip-utilities 
- `discordbotlist-stats-catnip` - Wrapper aiming at combining all Discord Bot Lists AND Wrappers into one artifact.
  Automatically handles pushing bot stats to bot lists. https://github.com/burdoto/discordbotlist-stats#using-with-catnip--

## Why write a fourth Java lib?

- JDA is very nice, but doesn't allow for as much freedom with customizing the internals;
  it's more / less "do it this way or use another lib" in my experience.
- I didn't want ten billion events for every possible case. catnip maps more/less 1:1 with the
  Discord API, and any "extra" events on top of that need to be user-provided via extensions or
  other means. I guess really I just didn't want my lib to be as "high-level" as other libs are.
- I wanted to try to maximize extensibility / customizability, beyond just making it modular. Things
  like being able to intercept raw websocket messages (as JSON), write custom distributed cache handlers,
  ... are incredibly useful.
- I like everything returning `CompletionStage`s instead of custom classes. I do get why other libs
  have them, I just wanted to not.
- I wanted modular usage to be exactly the same more / less no matter what; everything
  should be doable through the catnip instance that you create.
- I wanted to make a lib built on vert.x.
- To take over the world and convert all Java bots. :^)

### Why vert.x?

- vert.x is nice and reactive and async. :tm:
- You can use callbacks or Rx (like we do), but vert.x also provides support for reactive streams
  and kotlin coroutines.
- There's a *lot* of [vert.x libraries and documentation](https://vertx.io/docs/) for just about
  anything you want.
- The reactive, event-loop-driven model fits well for a Discord bot use-case.
