---
title: "Are Static Variables Thead Safe"
date: "2023-06-29"
tags: ["multithreading", "static variables"]
author: "Swebert Correa"
---

Multithreading in itself is hard. But adding static variables into the mix, and
we have chaos! One must always have a keen eye when dealing with them.

## What is a static variable

An ordinary variable is limited to the scope in which it is defined, while the
scope of the static variable is throughout the program. When a variable is
defined as static, memory is reserved for it at compile time.

## What is multithreading

Normally a program gets a single unit of execution assigned to it. But when one
wants to perform multiple tasks concurrently, multiple execution units, or
threads, are spawned and assigned to the task. This paradigm is helpful when
one deals with a lot of blocking code, like having sleeps, network calls, I/O
interactions, etc.

## Creating the problem...

Let's say we want to create a function to print the values from 1 through 10.
However, there is also some blocking routine that the function must perform.
Let's create a simple function that will:
- increment the static variable `x`
- sleep for 1 second
- print the value of `x`

```C
#include <stdio.h>
#include <unistd.h>

void * increment_and_print() {
    static int x = 0;

    /* calculation done on X */
    x++;

    /* some blocking routine */
    sleep(1);

    printf("x = %d\n", x);

    return NULL;
}

int main() {
    for (int i = 0; i < 10; i++) {
        increment_and_print();
    }

    return 0;
}
```

The _problem_ with this code is that it takes 10 seconds to run. We could just
create some threads to concurrently run this code, so that the time taken would
just be 1 second, right?

## Possible solution?

Creating threads in a loop, and assigning them to run the function should solve
the problem, right? Each thread runs concurrently, so the time taken to run the
code should be about 1 seconds.

```C
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>

void * increment_and_print() {
    static int x = 0;

    /* calculation done on X */
    x++;

    /* some blocking routine */
    sleep(1);

    printf("x = %d\n", x);

    return NULL;
}

int main() {
    pthread_t thread[10];

    for (int i = 0; i < 10; i++) {
        pthread_create(&thread[i], NULL, increment_and_print, NULL);
    }

    for (int i = 0; i < 10; i++) {
        pthread_join(thread[i], NULL);
    }

    return 0;
}
```

## Why is mixing them generally a bad idea

While the solution seems indifferent at first glance, careful inspection might
unveil a bug. The issue is that while the first thread is sleeping, the second
thread increments `x` and goes to sleep. Now when the second thread is sleeping,
the third one increments `x` and goes to sleep. This saga continues until all
the threads are sleeping. Since all the threads incremented the value of `x` and
went to sleep, the value of `x` now is 10. So instead of having the values 1
through 10 being printed out, we'll get 10 printed out ten times!
