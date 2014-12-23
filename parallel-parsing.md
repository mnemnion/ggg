Parallel parsing.


Let's assume we have a bunch of SIMDs and a bunch of cores. Like at least 32 cores, more like 64. 

So we can stripe through a long region of memory using a small rule, very quickly. Any other task we can do in parallel is appreciably faster. 

A lot of this will probably be wrong.

## Literals

We make a small kernel on the SIMDs for each literal terminal. We divide the string into regions, and for each literal we try to match it within that region. If we start on a byte that could be part of the literal, we back up exactly enough to check; if we're matching when we hit the end of a region, we continue. 

We want to partition fairly carefully, because any literal crossing a region boundary is matched twice. This isn't a problem but it is extra work. 

## Regular Expressions

