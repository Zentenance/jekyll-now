---
title: "Ring buffer in C"
date: 2021-05-08T21:25:47-07:00
draft: false
---

## Overview
> Implement a ring buffer in C.   

I got asked this questions already two times during screening interview with FANNG and failed two times. The ring buffer structure is also known as a circular queue. The different names come from two different usages. In embedded systems, in order to receive and send data to a peripheral device such as UART, a ring buffer is common to cache the data. The same structure is used for a certain RTOS to send and receive an event and data between tasks.

## Structure
It can be implemented using a linked list or an array. The diagram below is the ring buffer with size 4 using array. The four variables got involved in working with the ring buffer. They are **capacity**, **length**, **read index** and **write index**.
  
![](/Ring_buffer_in_C/Blank%20Diagram%201%20Page%201.png)

* **capacity**: capacity of ring buffers. Here it is 4.
* **length**: length of the buffers holding datas. Here it is 2
* **read index**: point to the start buffer to read
* **write index**: point to the start buffer to write

## Implementation
This is the `struct Queue` written in C.

```c
struct Queue {
	char *buf;

	size_t capacity;
	size_t length;
	size_t read;
	size_t write;
};

int init_queue(struct Queue *q, size_t capacity);
int enqueue(strcut Queue *q, char *data, size_t len);
int dequeue(struct Queue *q, char *data, size_t len);

int main() {
	Queue myqueue;
	assert(0 == init_queue(&myqueue, 4);

	return 0;
}

int init_queue(struct Queue *q, size_t capacity) {
	q->buf = (char*)malloc(capacity);
	if (!q->buf) return -1;
	q->capacity = capacity;
	q->len = q->read = q->write = 0;
	return 0;
}
```
  
We update four variables in cases;
1. When `init_queue` gets called, we update the **capacity** as total allocated buffer size.
2. In `enqueue()` , we do `memcpy()` the buffer to the data and update **read** index to `read += len`
3. In `dequeue()` , we do `memcpy()` the data to the buffer and update **write** index to `write += len`
4. In case 3 and 4, we update the **length** to `length += len` and `length -= len` respectively. 

Now tricky part comes up. How do we do `memcpy()` the data, and update **read** and **write** index, when the data need to be copied  all the way to end of the ring and continue to be copied from the beginning. This is the case with calling `enqueue(&myqueue, data,  3)` when the ring buffer is holding the data in the picture below.

![](/Ring_buffer_in_C/Blank%20Diagram%202%20Page%202.png)

Here is my two cents.

> When the problem looks complicating to tackle down, don’t panic.   
> Divide it to small sub problems you are familiar with.  
 
We just need to do `memcpy()` two times, first one for the data till the end of the queue and second one for the data from the beginning wile carefully updating the **write** index, and source and target pointers for `memcpy()`. Let’s do this.

```c
// first copy till the end
size_t len_to_end = MIN(len, q->capacity - q->write);
memcpy(q->buf + write, data, len_to_end);

// second copy from the begining of the buf with the remainings
if (len > len_to_end)
	memcpy(q->buf, data + len_to_end, len - len_to_end);

// update the write index and length of ring buffer
write = (write + len) % q->capacity;
q->len += len;  
```

This is it. To address the case to read circle around from the ring buffer, we implement the similar logic in `dequeue()`.

Before wrapping up, the last thing nice to have is helper functions below.

```c
static bool is_full(struct Queue *q) {
	return q->capacity == q->length;
}

static bool is_empty(struct Queue *q) {
	return q->read == q->write;
}

static size_t get_empty_buffers_length(struct Queue *q) {
	return q->capacity - q->length;
}
```

## Summary and dicussion
We implemented a ring buffer using four variables. Actually we don’t need all of **write**, **read** and **length** to implement the ring buffer. For example with **read** and **length**, we can come up with **write** index. With **write** and **read** we can calculate the **length** of ring buffer. 

Because **length** is referenced by `enqueue()` and `dequeue()` , mutex is required to allow multithreads to access the ring buffer. By removing **length** and having **write** and **read** only , we can possibly have lock-free implementation. For lock-free implementation details, Juho wrote a nice blog post - [I’ve been writing ring buffers wrong all these years](https://www.snellman.net/blog/archive/2016-12-13-ring-buffers/). 

