---
layout: post
title: "Everyday Useful Bit hacks"
date: 2020-05-06
--- 

# What are Bit hacks and why you should care

Bit hacks are pieces of Boolean logic that low level programmers can use to significantly increase the speed of certain operations.
For example, suppose have a port with multiple devices and need to have different operations for each on an interrupt. The naïve solution would be to 
loop over every bit in the interrupt status flag, which takes a lot of time, or you can mask out just the bits you need for each device as you need them. This way, you do not waste precious CPU cycles iterating over bits you probably do not need. Or if you need to count the number of set bits (without using a [lookup table]({{ 2020-05-06-When-To-Use-A-Lookup-Table | prepend: site.baseurl | replace: '//', '/' }})). These examples are used very often in the real world when speed and efficiency matter, they also have the added bonus of making you look like an **elite coder haxor** to your friends when you show off your projects. So without further preamble, here are some of my favorite and most used bit hacks, as well as some situations they are useful in.

# Bit hack number one: Counting in modulo N
Consider this scenario, you want to run the method ```run_after_three()``` after some event has occurred three times. This event could be anything, a button press, a failed response from a device, or number of times of times an accelerometer has measured an increase in velocity. For this simple example our event will be reaching the bottom of the main loop. Let;s look at this approach first
```C
int main()
{
    int count = 0;
    while(1)
    {
        if(count % 3 == 0)
        {
            run_after_three();
        }
        count++;
    }
    ...
}
```
This approach has some issues:
 1. The modulo operation is very slow
 2. Count will eventually overflow
 3. run_after_three() will fire on the first run.

 We can tackle the modulo operation first. Many of you may remember this operation from your school days as "Divide a by b and give me the remainder" so in this case, we divide count by 3 and spit out the remainder, and if its 0, the count is divisible by three and so must have increased three times. However, there is a major issue here, namely that computers are not all that good at dividing numbers, especially with odd sizes like this. Computers are very good at adding one, subtracting one and shifting bits left or right. Multiplication is relatively fast, since it can be broken done into adding one many times:
 ```
 2 * 3 = 2 + 2 + 2 = (2 * 1) + (2 * 1) + (2 * 1) = (1 + 1) + (1 + 1) + (1 + 1)
 ```
 Division does not regress cleanly into subtracting multiple ones like this. The simplest form of division relies on iterative subtraction via addition, and must keep tract of the previous operations results. Because of this the modulo operator takes a lot of time. But Fear not! we can still see if the number has been incremented three times by simply looking at the bottom two bits. For example:
 ```C
int main()
{
    int count = 0;
    while(1)
    {
        if(count & 0x11 == 1)
        {
            run_after_three();
        }
        count++;
    }
    ...
}
```
The reason this works is that binary values sort of naturally reset when counting up
```
0 = 0000 0000
1 = 0000 0001
2 = 0000 0010
3 = 0000 0011
4 = 0000 0100 <-- notice the two least significant bits are the same as 0;
5
6
7 = 0000 0111 <-- back the three on the LSBs
```
This solves issues 1 and 3, as we are now only taking one cycle to calculate the anded value and one to compare it to 1 and see if we have a "3". 
However, this still leaves issue number two the fact that our count will eventually over flow.
Depending on your system, this could be unimportant or a massive failure point, depending on if the count ever reaches that mythical value MAX_INT (or MAX_UINT if you're being clever). 
We can avoid this by restricting our count to only 0-3, once again by using some properties of binary math. Basically, we want to have the reset happen *without* setting the next bit. 
We can accomplish this by anding the number with the maximum value we would like it to obtain after addition. (**NOTE:** This only works when all bits in the given max value are set, 3 = 11, 7 = 111, F=1111 etc, there are other solutions for other values but they are far more involved). So we can write this solution:

 ```C
int main()
{
    int count = 0;
    while(1)
    {
        if(count == 3)
        {
            run_after_three();
        }
        count = (count + 1) & 0x03;
    }
    ...
}
```

This effectively allows count to increment by one until it reaches the value 0100, at which point it is set equal to 0000 instead of 0100. This allows us to count in modulo 3, and makes our risk of overflow zero.

# Bit hack number two: Using all of your bytes and nibbles
Let's make a slight addition to our program from our hack, now let's say I want to run another method ```run_after_seven()``` So what do we do? should we make about variable to store the count for this and go with it? What happens when we need another contrived method to run after some count? Obviously we can not just keep adding variables, eventually we will run out of memory. And what about access? Should we make an integer array to hold all of our counts and keep track of elements on a method basis? My answer is no, we should just us the count we already have, as it has tons of space.
Remember that our count is a variable of type int. That means it is 2 bytes, or 4 nibbles. Theses two counts can be contained in 4 bits each, as one is three and the other is seven.
So, since we are resetting the value without relying on the normal reset from incrementing, we have a lot of free space. Currently we are incrementing like this:
```
(0000 0000  0000 0000 + 0000 0000  0000 0001) & 0000 0000  0000 0011 =
(0000 0000  0000 0001)

(0000 0000  0000 0001 + 0000 0000  0000 0001) & 0000 0000  0000 0011 =
(0000 0000  0000 0010)

(0000 0000  0000 0010 + 0000 0000  0000 0001) & 0000 0000  0000 0011 =
(0000 0000  0000 0011)

(0000 0000  0000 0011 + 0000 0000  0000 0001) & 0000 0000  0000 0011 =
(0000 0000  0000 0000)
```
As you can see, we never touch any bit beyond bit 1. That means we have three nibbles that are completely untouched. We can use these to store more information.
We can do this by using the shift operations >> and << (left and right shift). These are used to move bits left or right by the specified amount, for example  0100 >> 1 = 0010 and 0100 >> 2 = 0001 (Usually, the empty bits are filled with 0s when performing a shift). This operation can be performed in an if state non destructively to isolate values, as well in an assignment to move values to specific points. There is a catch though, now that each nibble has a meaning beyond empty space we need to employ **friendly coding** to it, as well as bit masking to isolate values

 ```C
int main()
{
    int count = 0;
    while(1)
    {
        if((count & 0x03) == 3)
        {
            run_after_three();
        }
        if(((count & 0x70) >> 4) == 7)
        {
            run_after_seven();
        }
        count |= (count + 1) & 0x03;
        count |= (count + (1 << 4)) & 0x70;
    }
    ...
}
```
You might be thinking "Huh where did all of those hex values come from? why are you using 4 for the shift? what in world am I getting into?" Once again have no fear, it is all very simple when you get down to it.
All we are doing here is masking out bits we are not interested in and then shifting into the correct location. 4 is the number of bits in a nibble, so that is how far left and right to shift and get the data for the second nibble.
Shifting 1 left by four produces 0001 0000, and shifting any value right by four produces 0000 xxxx. 
So, the statement count |= count | ((count + ( 1 << 4))) & 0x70 resolves down to 
```
(xxxx xxxx) | ((xxxx xxxx + 00001 0000) & 0111 0000) = (xxxx xxxx) | ((xxxx+1) xxxx & 0111 0000) = (xxxx xxxx) | (0xxx+1 0000) = 0xxx+1 0000
```
We can see how this allows us to count up to seven in the second nibble and reset it at seven.
The statement ```if ((( count & 0x70) >> 4) == 7)``` simply masks out the lower nibble and shifts the second nibble down.
 (**NOTE:** You do not have to shift in the if statement, you could simply compare to 0x70 after masking)

 # Bit Hack number three: counting bits

 There are two major ways to count bits, using a lookup table or looping through the number. Lookup tables are O(1) time but take up some memory, looping through a number can be reduced to O(log(n)) with Brian Kernighan's algorithm. 
 This algorithm exploits the fact that logical ands of N and N-1 always turn off the right most bit of N, meaning you can use a count variable and a loop like this:
 ```c
 int countSetBits(int n)
 {
     int count = 0;
     while(n)
     {
         n &= (n - 1);
         count++;
     }
     return count;
 }
 ```
 This will very quickly count the number of set bits, as it usually reduces in by a large amount, except in the worst case of all bits being set.

 So, back to our contrived example, let's say that each of our functions returns some value on an error, and we store whether we have seen an error at bit 3 in each of their respective nibbles, and we want to exit if both of them get raised.
 We can easily implement this with Brian Kernighan's, also we area going to rename "count" to "status" since it is now doing more than counting.
  ```C
int errorCount(int n)
{
    int count = 0;
     int count = 0;
     while(n)
     {
         n &= (n - 1);
         count++;
     }
     return count;
}

int main()
{
    int status = 0;
    while(1)
    {
        if(errorCount(status) >= 2)
        {
            return ERROR;
        }
        if((status & 0x03) == 3)
        {
            if (run_after_three() == ERROR)
            {
                status |= 0x08
            }
        }
        if(((status & 0x70) >> 4) == 7)
        {
            if (run_after_seven() == ERROR)
            {
                status |= 0x80;
            }
        }
        status |= (status + 1) & 0x03;
        status |= (status + (1 << 4)) & 0x70;
    }
    ...
}
```

Obviously this is not a very graceful solution, but this is an extremely contrived example.
It does show that you can construct fairly robust software with these bit hacks and techniques, and at a low level this software would be much fast than software that utilized the naïve approaches.

# Conclusion
 I hope you have enjoyed this quick peek into the world of useful bit hacks, these are ones that I find myself using the most, all though there are many many others.