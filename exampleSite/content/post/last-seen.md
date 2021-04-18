+++
draft = true
lastmod = 2021-04-18T14:05:00Z
publishdate = 2021-04-18T14:05:00Z
tags = ["removal-listener", "caches", "guava", "java"]
title = "Last Seen"

+++
## Use Case

Suppose your manager tells you - “Joe, do you remember where we store MaxMind data? Can you figure out when we saw an IP the last time around and send it on a stream? Gini will write a consumer to look at some patterns, also... don’t bombard him”.

## Thinking Process

Wait, what is MaxMind? MaxMind is a company that provides location data given an IP. Something like 205.101.100.247 -> Istanbul. 

Ok, need a code refresher now.

    LoadingCache<String, String> cache =
        CacheBuilder.newBuilder()
                    .expireAfterWrite(5, TimeUnit.MINUTES)
                    .build(new CacheLoader<String, String>() {
                        @Override
                        public String load(String key) {
                            // Query Table and then MaxMind
                            return UUID.randomUUID() + "_place";
                        }
                    });

We have a cache (essentially a key-value map) that stores IP and its location. 

After 5 minutes, the cache forgets the mapping. 

Coming back to "when did I see it last"? The obvious answer is whenever the cache forgets about it, right?

## What can I do about it?

Wonder if this cache has a hook that is invoked when it forgets something? 

It has built-in support for it - "removal listeners".

    LoadingCache<String, String> cache =
        CacheBuilder.newBuilder()
                    .expireAfterWrite(5, TimeUnit.SECONDS) // changed to SECONDS for local testing
                    .removalListener((RemovalListener<String, String>) notification -> {
                        System.out.printf("Removal listener called since %s with %s %s \n", 
                                          notification.getKey(), 
                                          notification.getValue(), 
                                          notification.getCause());
                        // Wrap key, value and time in an object and pass it along the stream for Gini.
                    })
                    .build(new CacheLoader<String, String>() {
                        @Override
                        public String load(String key) {
                            // Query Table and then MaxMind
                            return UUID.randomUUID().toString().split("-")[0] + "_place";
                        }
                    });
    
    // Testing
    System.out.println(cache.get("205.101.100.247")); // -> prints 26be9f3a_place (totally random place)
    System.out.println(cache.get("205.101.100.247")); // -> prints 26be9f3a_place (same random place)
    
    Thread.sleep(10_000);                             // sleep for 10 seconds
    
    System.out.println(cache.get("205.101.100.247")); // -> Removal listener called since 205.101.100.247 with 26be9f3a_place EXPIRED
                                                      // -> and prints 14e3c237_place (totally random place again)

The inline testing comments above should make the behavior of removal listener clear. 

Let me know if it doesn't. I'm @anirudhonezero. 

Thanks for reading along.

## Gotchas

1. Sometimes removal listener might not be called (implementation detail), this is because guava doesn't have clean-up threads. It relies on get, write to happen on the same segment of the expired key to happen. In our case, the last get call invokes the segment read and hence the removal listener. Reference - [https://stackoverflow.com/a/12157670](https://stackoverflow.com/a/12157670 "https://stackoverflow.com/a/12157670")


2. The last print statement should return the same place in a production environment.