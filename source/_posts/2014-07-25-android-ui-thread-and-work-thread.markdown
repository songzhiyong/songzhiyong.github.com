---
layout: post
title: "Android UI Thread &amp; work Thread"
date: 2014-07-25 10:37
comments: true
categories: android
---
Using `wait()` and `notifyAll()` to make UI thread and work thread work one by one

```java
new Thread(new Runnable() {
          @Override
          public void run() {
            synchronized (lock) {
              for (int i = 0; i < 100; i++) {
                final int index = i;
                if (i % 5 == 0) {
                //on some condition, update UI on UI thread with handler
                  handler.post(new Runnable() {
                    @Override
                    public void run() {
                      synchronized (lock) {
                        Log.i(TAG, "ui" + index);
                        lock.notifyAll();
                      }
                    }
                  });
                  try {
                    lock.wait();
                  } catch (InterruptedException e) {
                  }
                }
                Log.i(TAG, "work" + i);
              }
            }
          }
        }).start();
```