# Math is the Answer 96.86% of the Time
###### *A [BigDonRob](https://retroachievements.org/user/BigDonRob "BigDonRob") writeup*

The rest is as easy as pi. Here is a collection of useful math tips, tricks, and cheats that will help you turn a lot of logic into simple Delta = X - 1, Mem = X scenarios.


### Table of Contents:
* [Add Source with Modifications ( * )](#add-source-with-modifications--)
* [Advanced Bitcounting](#advanced-bitcounting)
* [N != 0 (0/1)](#n--0-01)
* [N < X (0/1)](#n--x-01)
* [N >= X (0/1) MANUAL](#n--x-01-manual)
    * Also covers: N > X
* [N >= X || M + N >= X (0/1) with Remember/Recall](#n--x--m--n--x-01-with-rememberrecall)
    * Also covers: N > X || M + N > X (0/1)
* [N == X (0/1) MANUAL](#n--x-01-manual-1)
* [N == X (0/1) with Remember/Recall](#n--x-01-with-rememberrecall)
* [N == X1 || N == X2 (0/1) with Remember/Recall](#n--x1--n--x2-01-with-rememberrecall)

## Add Source with Modifications ( * )

$${\color{orange}Delta}$$ / $${\color{orange}Mem}$$ checks are always needed, even if they aren't always framed exactly as $${\color{orange}Delta}$$ / $${\color{orange}Mem}$$ in the logic. For the best way to ensure accurate single frame timing with a measurable count, we can do a lot with $${\color{red}Add \space Source}$$ and $${\color{red}Sub \space Source}$$. 

The most obvious and simple use of this would be checking that a score was above or below something, and is now at or less than/greater than the target.

If your score is nice and simple, all you really need is
```
Delta  < Target
Mem   >= Target
```
But, with a lot of games, you will find the score is stored as Binary Coded Decimals, ASCII (0x30,0x31,0x32, etc.), or you have to track the score in a way that is not as simple.

The basic math here is $${\color{red}Add \space Source}$$ with Modifications. ( * )
```
Add Source Delta Score(000X)
Add Source Delta Score(00X0) *  0xA
Add Source Delta Score(0X00) *  0x64
Add Source Delta Score(X000) *  0x3E8
           Value 0           <  XXXX
Add Source Mem   Score(000X)
Add Source Mem   Score(00X0) *  0xA
Add Source Mem   Score(0X00) *  0x64
Add Source Mem   Score(X000) *  0x3E8
           Value 0           >= XXXX
```

Works great, it can be measured if you need it to be, and locks things down to the single frame it becomes true.

Now, a special note about BCD, you can't just slap a $${\color{orange}Delta}$$ on it, because BCD is already a type. What you can do is individually map the deltas with math and use BCD for the actual $${\color{orange}Mem}$$ check.
```
Add Source Delta Lower4 Score(000X)
Add Source Delta Upper4 Score(00X0) *  0xA
Add Source Delta Lower4 Score(0X00) *  0x64
Add Source Delta Upper4 Score(X000) *  0x3E8
           Value 0                  <  XXXX
           BCD   16-bit Score       >= XXXX
```

[⬆️ Return to Top](#math-is-the-answer-9686-of-the-time)

## Advanced Bitcounting

So, that is the basics of BIG numbers, but what happens when we want to count super specific data, i.e. individual bits and bitcounts?

A common example of this happens in Pokémon games. Catch them all. Have X of these Y types/groups, etc. When these are stored as actual $${\color{green}Bits}$$, we have 3 options at our disposal.

1. $${\color{red}Add \space Source}$$ BitCount

Adds a 1 for every $${\color{green}Bit}$$ turned on in an 8-bit memory region. Works with $${\color{orange}Delta}$$. Clean and simple.

2. $${\color{red}Add \space Source}$$ BitX

Adds a 1 for a specific $${\color{green}Bit}$$ being turned on. If you need to count $${\color{green}Bits}$$ 2, 5, and 7, you just add $${\color{green}Bits}$$ 2, 5, and 7. Easy enough!

3. Advanced Bitcounting:
```
Add Source Bitcount
Sub Source BitX
```

This one is a little more complicated to wrap your head around, and is more of an optimization than a requirement. But let's say the $${\color{green}Bits}$$ you need are [0,1,3,4,5,6,7]. You CAN just add all 7 of those $${\color{green}Bits}$$. 7 lines of code never hurt anything. 
```
Add Source Delta Bit0
Add Source Delta Bit1
Add Source Delta Bit3
Add Source Delta Bit4
Add Source Delta Bit5
Add Source Delta Bit6
Add Source Delta Bit7
           Value 0    = 6
Add Source Mem   Bit0
Add Source Mem   Bit1
Add Source Mem   Bit3
Add Source Mem   Bit4
Add Source Mem   Bit5
Add Source Mem   Bit6
Add Source Mem   Bit7
           Value 0    = 7
```

BUT, what about when you have 500 things to track that are in clumps of 5 here, 7 there, 3 there? A simple rule here is that if you are adding 5 or more $${\color{green}Bits}$$ from a single 8-bit address, you can add the $${\color{green}Bitcount}$$, and subtract the $${\color{green}Bits}$$ you don't need.
```
Add Source Delta Bitcount
Sub Source Delta Bit2
           Value 0    = 6
Add Source Mem   Bitcount
Sub Source Mem   Bit2
           Value 0    = 7
```

Same effect, significantly fewer lines. Here's how it works. By adding the $${\color{green}Bitcount}$$, you are adding ALL of the $${\color{green}Bits}$$ individually, by subtracting $${\color{green}Bit2}$$, you are negating that addition for $${\color{green}Bit2}$$ ONLY.

If $${\color{green}Bit2}$$ is off, its net is +0 - 0 = 0.
If $${\color{green}Bit2}$$ is on, its net is +1 - 1 = 0.

[⬆️ Return to Top](#math-is-the-answer-9686-of-the-time)

## N != 0 (0/1)

So, you just need to know if a thing is on or off. You have a counter for X things. But all you need to know is if it is 0 or not.
```
Add Source Delta Thing1 / Delta Thing1
Add Source Delta Thing2 / Delta Thing2
Add Source Delta Thing3 / Delta Thing3
           Value 0     = 2
Add Source Mem   Thing1 / Mem   Thing1
Add Source Mem   Thing2 / Mem   Thing2
Add Source Mem   Thing3 / Mem   Thing3
           Value 0     = 3
```

0/0 does not break the universe. At least in the RA toolkit. It just returns 0. So this math is a standard way to turn everything into 1 or 0. Kill at least 1 of every enemy? Check for the $${\color{orange}Deltas}$$ to equal total - 1 and the $${\color{orange}Mem}$$ to equal the total.

[⬆️ Return to Top](#math-is-the-answer-9686-of-the-time)

## N < X (0/1)

What if you have a "Best Score" that defaults to 0xFFFF before you start? Well, you can't use $${\color{orange}Mem}$$ / $${\color{orange}Mem}$$, since it will be 1 for every value. Integer math becomes the answer. If X is divided by X + k, the result is 0.

1 / 2 = 0
1 / 63435 = 0

If we leverage this into a $${\color{red}Sub \space Source}$$, we can make a simple counter for "post a score on all X of these"
```
Sub Source Delta Score1 / 0xFFFF
Sub Source Delta Score2 / 0xFFFF
Sub Source Delta Score3 / 0xFFFF
           Value 3     = 2
Sub Source Mem   Score1 / 0xFFFF
Sub Source Mem   Score2 / 0xFFFF
Sub Source Mem   Score3 / 0xFFFF
           Value 3     = 3
```

This is another weird one, but if the default is 0xFFFF, then the only time you will get a value of 1 is before a lower score is set.

1 - 1 = 0
1 - 0 = 1

[⬆️ Return to Top](#math-is-the-answer-9686-of-the-time)

## N >= X (0/1) MANUAL

N > X (0/1) [Just use X +1 with the formula]
```
+ / x
- / 2x
- / 3x
- / 5x
+ / 6x
- / 7x
etc.
```
```
+ / x
- / x (primes)
+ / x (distinct 2-prime products)
- / x (distinct 3-prime products)    ← 30x, 42x, etc.
+ / x (distinct 4-prime products)    ← 210x, 330x, etc.
```

You have a counter that goes from 1 to 10. All you want to know is that the player has 3 or more.
```
Add Source Delta Thing / 3
           Value 0     = 0
Add Source Mem   Thing / 3
           Value 0     = 1
```

That works, right? Yes, but only until the Thing reaches 6. Now you suddenly have a return of 2 instead of 1. But what if we negate the extra 1 at 6 and higher?
```
Add Source Delta Thing / 3
Sub Source Delta Thing / 6
           Value 0     = 0
Add Source Mem   Thing / 3
Sub Source Mem   Thing / 6
           Value 0     = 1
```

So at 6, 7, or 8, we end up with + 2 - 1 = 1. Perfect. But we need to account for 9, as well.
```
Add Source Delta Thing / 3
Sub Source Delta Thing / 6
Sub Source Delta Thing / 9
           Value 0     = 0
Add Source Mem   Thing / 3
Sub Source Mem   Thing / 6
Sub Source Mem   Thing / 9
           Value 0     = 1
```

Now we end up with a proper + 3 - 1 - 1 = 1. If our cap is 10, we're golden. If our cap is less then 15, we end up with + 4 - 2 - 1 = 1. This math can be expanded as many times as necessary to get accurate 0/1 checks.

### Also covers: N > X

Just use X + 1 with the formula above.

[⬆️ Return to Top](#math-is-the-answer-9686-of-the-time)

## N >= X || M + N >= X (0/1) with Remember/Recall

N > X (0/1) [Just use C - X - 1 with the formula]
```
Add Source N1
Remember   0      + (C - X)
Remember   Recall / C
Remember   Recall * C
Add Source N2
Remember   Recall + (C - X)
Remember   Recall / C
Remember   Recall * C
etc.
Remember   Recall / C
Measured   Recall = Target
```

For this formula, C needs to be greater than the max possible N value - X. An easy default is 0xFF for 8-bit, 0xFFFF for 16-bit, etc.

With this formula, we greatly simplify the logic whenever $${\color{blue}R/R}$$ is available. If N >= X, the sum becomes >= C, making the Recall / C simplify it to 1/0. Multiplying by C keeps the current count accurate when adding the next item in line.

This can be expanded to allow multiple values to be added, or subbed, or manipulated any way you want to find out if the end result becomes 0 (< X) or 1 (>= X), just make sure C is greater than the max possible SUM of maxN * Entries.

A specific use case is having 3 individual $${\color{green}Bits}$$ that are obtained items that give + 1, +2, +3 to a certain stat. What if you want a single 0/1 for having the + 2 OR +3 for each of these items?
```
Add Source Delta  BitX(+2)
Add Source Delta  BitY(+3)
Remember   Value  0         + 2
Remember   Recall           / 3
Remember   Recall           * 3
Add Source Delta  BitXX(+2)
Add Source Delta  BitYY(+3)
Remember   Recall           + 2
Remember   Recall           / 3
           Recall           = 1
Add Source Mem    BitX(+2)
Add Source Mem    BitY(+3)
Remember   Value  0         + 2
Remember   Recall           / 3
Remember   Recall           * 3
Add Source Mem    BitXX(+2)
Add Source Mem    BitYY(+3)
Remember   Recall           + 2
Remember   Recall           / 3
Measured   Recall           = 2
```

Whew!!! A lot of math involved. But how does it work?
```
+ 0 + 0 + 2     = 2
2 / 3           = 0

+ 1 + 0 + 2     = 3
3 / 3           = 1
+ 1 + 1 + 2     = 4
4 / 3           = 1

1 * 3           = 3

+ 0 + 0 + 2 + 3 = 5
5 / 3           = 1

+ 1 + 0 + 2 + 3 = 6
6 / 3           = 2
+ 1 + 1 + 2 + 3 = 7
7 / 3           = 2
```

### Also covers: N > X || M + N > X (0/1)

Just use C - X - 1 with the formula above.

[⬆️ Return to Top](#math-is-the-answer-9686-of-the-time)

## N == X (0/1) MANUAL

```
+ / 1x
- / 1(x + 1)
- / 2x
+ / 2(x + 1)
- / 3x
+ / 3(x + 1)
- / 5x
+ / 5(x + 1)
+ / 6x
- / 6(x + 1)
- / 7x
+ / 7(x + 1)
+ / x
- / (x + 1)
- / x (primes)
+ / (x + 1) (primes)
+ / x (distinct 2-prime products)
- / (x + 1) (distinct 2-prime products)
- / x (distinct 3-prime products)    ← 30x, 42x, etc.
+ / (x + 1) (distinct 3-prime products)
+ / x (distinct 4-prime products)    ← 210x, 330x, etc.
- / (x + 1) (distinct 4-prime products)
```

Let's start with a simple use case. You have something that has a range of 0 through 7, and you want to add 1 for the number being 6, and only 6.
```
Add Source Delta Thing / 0x6
Sub Source Delta Thing / 0x7
           Value 0     = 0
Add Source Mem   Thing / 0x6
Sub Source Mem   Thing / 0x7
           Value 0     = 1
```

Your math becomes
```
+ 0 - 0 = 0
+ 1 - 0 = 1
+ 1 - 1 = 0
```

Now, how do we scale this for BIG ranges?
We'll use an X of 0x77 and a maximum value of 0xFF
```
Add Source Delta Thing1 / 0x77
Sub Source Delta Thing1 / 0x78
Sub Source Delta Thing1 / 0xEE
Add Source Delta Thing1 / 0xF0
           Value 0      = 0
Add Source Mem   Thing1 / 0x77
Sub Source Mem   Thing1 / 0x78
Sub Source Mem   Thing1 / 0xEE
Add Source Mem   Thing1 / 0xF0
           Value 0      = 1
```

Under 0x77 becomes 0
0x77 becomes 1
Over 0x77 becomes 0
```
+ 0 - 0 - 0 + 0 = 0
+ 1 - 0 - 0 + 0 = 1
+ 1 - 1 - 0 + 0 = 0
+ 2 - 1 - 1 + 0 = 0
+ 2 - 2 - 1 + 1 = 0
```

[⬆️ Return to Top](#math-is-the-answer-9686-of-the-time)

## N == X (0/1) with Remember/Recall

```
Add Source C
Sub Source N1     ^ X
Remember   0
Remember   Recall / C
Remember   Recall * C
Add Source C
Sub Source N2     ^ X
Remember   Recall
Remember   Recall / C
Remember   Recall * C
etc.
Remember   Recall / C
Measured   Recall = Target
```

For this formula, C needs to be greater than the max possible XOR case. An easy default is 0xFF for 8-bit, 0xFFFF for 16-bit, etc.

If you have $${\color{blue}Remember/Recall}$$ available to use, this is hands down much easier to work with than the manual version if your RANGE is much greater than your target. It will collapse the expanding line count of the manual version to a maximum of 5 per entry. This saves you space anytime the maximum possible value divided by your target value is greater than 3.

 While you can technically use $${\color{blue}Remember/Recall}$$ + C to save a line, that allows for an underflow and should be avoided.

You have your constant, then you subtract the XOR from it. If N == X, that subtraction will be 0 and the division returns 1. If N != X, the subtraction will always be greater than 0, making the division collapse into 0.

[⬆️ Return to Top](#math-is-the-answer-9686-of-the-time)

## N == X1 || N == X2 (0/1) with Remember/Recall

If we want to expand the above logic to allow for multiple X values, such as have an Ice Sword OR a Fire Sword equipped on all 3 characters, we can use the same process, but we must repeat the SAME lines (Values * Entries).
```
Add Source C
Sub Source N1     ^ X1
Remember   0
Remember   Recall / C
Remember   Recall * C
Add Source C
Sub Source N1     ^ X2
Remember   Recall
Remember   Recall / C
Remember   Recall * C
Add Source C
Sub Source N2     ^ X1
Remember   Recall
Remember   Recall / C
Remember   Recall * C
Add Source C
Sub Source N2     ^ X2
Remember   Recall
Remember   Recall / C
Remember   Recall * C
Add Source C
Sub Source N3     ^ X1
Remember   Recall
Remember   Recall / C
Remember   Recall * C
Add Source C
Sub Source N3     ^ X2
Remember   Recall
Remember   Recall / C
Measured   Recall = 3
```

[⬆️ Return to Top](#math-is-the-answer-9686-of-the-time)
