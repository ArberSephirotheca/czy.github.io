# MIT 6.824 - Lecture 1 Introduction

**MIT 6.824 Lecture 1 Note**
<!--more-->
# WHAT is Distribution System
* Multiple computer works together to serve a same service.

## WHY Distribution System
* **Parallelism**.
* **Fault tolerance**:
  * Avaliability - Under certain failures, we can still provide service. etc(replicate servers, one fail, and one still running).
  * Recoverability.
  * Non-volatile storage - HHD SSD.
  * Replication
* **Physical** (bank transfer between two different regions etc.).
* **Security/Isolation** (separate code on different machine).

## Consistency
* Put/Get:
  * given the danger that user may get different version of data.
* Strong consistency:
  * version control of get.
  * put replicas as far as possible to prevent data loss from natural disaster.
* Weak consistency:
  * Hardware is weakly ordered with respect to a synchronization model if and only if it appears sequentially consistent to all software that obey the synchronization model.

## MapReduce
* Map function:
  * `Map(k,v)`, where `k` is filename and `v` is content.
  * Map function split `v` into word.
  * For each word w emit `(w, "1")`.
* Reduce function:
  * `Reduce(k, v)`.
  * `emit(len(v))`.
