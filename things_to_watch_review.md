# bad c functions

* strcpy, use strncpy instead. Ensure its nul-terminated
* strcmp, use strncmp instead
* Ensure snprintf return value is used properly

# Memory Allocation

* What are things that are allocated now?
* Are they coming from pooled data structures
* How much is the lifetime?
* How thread-local they are? Same thread handling or one thread allocing another dealloc'ing.
    * Thread local allocations may be faster in Pooled-allocators.

# General

* Use `__thread` instead of `pthread_key_create`

# Naming convention

* Is there some sanity in naming types and variables

# Logging

* ARM-ID duplication

# CPP

* const in arguments, fn-decl
