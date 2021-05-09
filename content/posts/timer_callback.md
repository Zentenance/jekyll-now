---
title: "Timer Callback"
date: 2021-05-09
draft: false
tags: ['Embedded Software', 'C/C++']
---

This is an interview question for embedded software engineer. 

> ISR gets called with a certain unsigned integer value every 1ms.   
> Write the ISR callback to return average of values during 1 sec  

The timer callback gets invoked from ISR. 

First of all we need to calculate the average as fast as possible because this is called in ISR context. Instead of calculating sum of 1000 values during past 1 sec every time, we can cache the previous sum to calculate the average. The small key idea is when we add current value, we remove the oldest value from the cache. 

If this is an interview question, you would ask to an interviewer what the ISR returns when the ISR has not received 1000 value yet. The example below the ISR returns 0 in that case.

```c
#define BUCKETS_FOR_ONE_SEC	(1000)

// we should do memset(value, 0, sizeof(unsigned int) * BUCKETS_FOR_ONE_SEC); 
unsigned int value[BUCKET_FOR_ONE_SEC]; 
unsigned int sum = 0;
unsigned int count = 0;
bool ready = false;

void timer_callback(unsigned int val) {
	
	sum -= value[count];
	sum += val;
	value[count] = val;
	
	if (++count >= BUCKETS_FOR_ONE_SEC) ready = true;
	count %= BUCKETS_FOR_ONE_SEC;

	return ready ? sum / BUCKETS_FOR_ONE_SEC : 0;
}
```

