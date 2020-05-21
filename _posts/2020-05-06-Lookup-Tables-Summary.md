---
layout: post
title: "When to use a lookup table"
date: 2020-05-06
--- 

# Lookup tables, what are they?

A lookup table is a data structure that allows for fast access to data that might otherwise have to be calculated. They are especially useful when the data takes a long to be calculated but requires relatively little storage space once found. Some real world examples include the *Table of Logarithms* from the 20th century, or a relatively frequency table for characters.

# When should you use one?

Generally, a lookup table should be used when the space requirement is not as important as the time requirement. Generally, memory is more abundant than time but that is not always the case. An example would be some program that needs to quickly know how many sensors have a high voltage (Digital One), it would most likely be beneficial to use a lookup table as opposed to iterating over the sensor data, even if you were to iterate relatively fast.

# Example table

This is just the table of number of set bits in a given 4 bit number (0x0 - 0xF):
```c

int table[15] = { 0, 1, 1, 2, 1, 2, 2, 3, 1, 2, 2, 3, 2, 3, 3, 4};    
              //{ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, F}
```

# A note

Lookup tables are generally faster than crunching the numbers, however, there may be limitations to memory access speed depending on where the table is being stored. In general, lookup tables stored in RAM are very fast, but wasteful of useful memory. If your access speed is faster than whatever algorithm generates a single value, then use the table, if not then maybe generation is the better choice.