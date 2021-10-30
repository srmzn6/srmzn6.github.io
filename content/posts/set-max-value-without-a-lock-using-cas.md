---
title: "Set Max Value Without a Lock Using Cas"
date: 2021-10-24T17:04:46-04:00
draft: false
tags:
    - CAS
keywords:
    - CAS
    - Lock
---

**tl;dr** : I was looking for a way to set the maximum value of a variable but without using locks (because locks suck) this is my rant about it using CAS (compare-and-set operation).

We deal a lot with multi-threading and performance is something which we cannot part with. We needed a way to record the maximum latency for the application. The throughput is about 15-20K requests/second  and we needed to monitor and set the maximum latency value (since the start of the application) amongst other things, without affecting the performance.

here were a bunch of solutions that I found online that used locks, but locks although conceptually simple are performance bottleneck. Naturally, I turned to the Java concurrency package for reference and realized that [Doug Lea](http://en.wikipedia.org/wiki/Doug_Lea) had this algorithm which he used for [AtomicLong.getAndIncrement()](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/6-b14/java/util/concurrent/atomic/AtomicLong.java#AtomicLong.getAndIncrement%28%29) which used CAS operations. I used the same algorithm to set the max value and so far it appears to work.

Following is the code I used, with comments, hope it helps !

```      
       /**
	 * Try to update the max value without using a lock.
	 * This method is thread safe.
         *
	 * @param val the values to be updated to, if greater than given atomic.
	 * @param atomic the {@link AtomicLong} to be checked
	 * @return true if updated, false if not updated.
	 */
	public static boolean setMax(final long val, final AtomicLong atomic)
	{
	while (true)
	{
		final long currentMax = atomic.get();

		if (currentMax &gt;= diff)
		{
			/* if the current value is greater than diff we break */
			return false;
		}

		/*
		* Here we check whether other thread has already modified the
		* currentMax value, if yes setSuccess=false or else 
		* setSuccess=true in which case we the following operation 
		* will update the maxLatency value to the diff.
		*/
		final boolean setSuccess = atomic.compareAndSet(currentMax, diff);

		if (setSuccess)
		{
			/*
			* we managed to update the max value, no other thread changed
			* it, we are don here.
			*/
			return true;
		}

		/* some other thread got in the way, go back and do the drill again. */
	}

	}
```

