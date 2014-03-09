---
layout: post
title: A silly review of DEC64
date: '2014-03-09T11:15:00+02:00'
tags:
- Javascript
- Floating Point
- Rant
disqus_id: a-silly-review-of-dec64
---

This is a rant about Douglas Crockford's DEC64 since I get so butthurt by it.

I also realize that he is much older, much more popular, and has a lot longer history with computing than I do. He actually remembers a lot of the things I have only read about. Still, I think there are technical and historical inaccuracies on his page that I would like to comment on so I can get it out of my system.

I was thinking of doing it when I heard his talk in Malmö about DEC64, but decided that he probably wouldn't actually publish it since he probably has more important matters to attend to.

Of course, as programmers, reinventing the wheel badly when we have more important things to attend to, is second nature. So we'll do a shot-by-shot review of his specification, webpage, and even some of the code.

 > It can precisely represent decimal fractions with 16 decimal places, which makes it well suited to all applications that are concerned with money.

Incidentally, binary floating point can also represent almost 16 (roughly 15.955) decimal places accurately. So the gains are roughly 0.045 decimal places. But if we round down, and use a currency like the Japanese Yen, 10^15 is still quite a bit of cash. About 1.5 times the GDP of Japan. It's relatively seldom that I handle piles of cash that large without thinking about the type I'm using to store it in. If I don't round down, I can still store the GDP of the world accurately down to a single JPY.

 > It can represent values as large as 3.6028797018963968E+143 or as small as 1.0E-127, which makes it well suited to most scientific applications. 

A reduced range compared to binary floating point isn't much of an issue, the range is asymmetric which annoys my sense of order, but shouldn't affect real programs.
 
 > It can provide very fast performance on integer values, eliminating the need for a separate int type and avoiding the terrible errors than can result from int truncation.

So can binary floating point, I have done a lot of integer work in Javascript and you can do it easily. On the other hand, I have never been hit by the 'terrible errors' that can result from int truncation. Actually, of the basic operators, only division can have rounding errors.

 > DEC64 is intended to be the only number type in the next generation of application programming languages.

Programmers like specialized types, and there are a reasons other than performance why the committee of the next generation of application programming languages didn't decide on a single numeric type. Actually, most languages that only have one numeric type and a committee (read Javascript) have decided on adding more types.


Representation
--------------

 > DEC64 represents numbers as 64 bit values composed of 2 two’s complement components: a 56 bit coefficient and an 8 bit exponent. The coefficient is in the high order end, and the exponent is in the low order end. The coefficient’s decimal point is between bits 8 and 7.

Each value can have multiple bit-patterns, this means that there can be denormals everywhere, this inhibits a fast hardware implementation.

I assume a fast hardware implementation is possible, since it's possible with IEEE 754. But it also means that even a simple comparison takes an amazing amount of instructions in software, and that the worst case of performance is hit a lot more often.

It also means that there can never be a (useful) flush-to-zero like mode that removes the cost of handling denormals.

 > The coefficient is in the range -36028797018963968 .. 36028797018963967. The exponent is in the range -127 .. 127. Numbers may not use an exponent containing -128. The value of a number is obtained from this formula: value = coefficient * 10 ^ exponent

Reducing the exponent to 8 bits is good I think, since it does enable slighly faster software implementations, and increases precision to the same level as binary floating-point.

 > Normalization is not required, and is usually not desired. Integers can have an exponent of 0 as long as the coefficient is less than 36 quadrillion. Addition of numbers with equal exponents could be performed in a single machine cycle.

I don't know what machine Crockford has, but his code can at best be 3 cycles on a Haswell, and we'll do the calculations later. But a super wide VLIW architecture like the Mill (check out <http://ootbcomp.com/>), could probably do it in one instruction bundle.

 > There are 255 possible representations of zero. They are all considered to be equal.

True, but a lot of other values have multiple representations too. Any number with leading and trailing zeroes does.

 > There is a special value called nan that has a coefficient of 0 and an exponent of -128. The result of division by zero is nan. nan is also the result of operations that produce results that are too large to be represented. nan is equal to itself.

I like infinity in floating-point, and they are the special values in IEEE 754 that make the most sense for most people. Keeping NaN but throwing away infinity is a weird design decision.

 > When an arithmetic operation has an input with an exponent of -128, the result will be nan. Applications are free to use the coefficient as they wish when the exponent is -128, since in that case the coefficient has no arithmetic significance. One possible use is to store object pointers in the coefficient. 

Didn't Crockford just say that DEC64 just had 1 NaN value, but now there's almost 2 ^ 56 more?

Implementation
--------------

 > DEC64 can be implemented efficiently in hardware or software. 

IEEE 754 binary arithmetic is implemented efficiently in hardware and software.

 > Conversion to and from textual representations is simple and straightforward and free of the complexities that binary floating formats must wrestle with to minimize the inevitable errors caused by the fundamental incompatibility of the binary and decimal systems. DEC64 instead uses an internal representation that is very compatible with the E notation.

This is true, and the main advantage of decimal floating-point, string conversion is very simple, an operation that is error-prone for the binary equivalent, especially on the x87 series of FPUs.

 > To convert an int to DEC64, shift it left 8 bits. To unpack a coefficient, shift it right 8 bits with sign extension. The exponent can be unpacked at no cost on x64 architecture because the least significant byte can be accessed directly.

This is true, it is slightly faster than the equivalent code for binary double-precision arithmetic. Except that we don't need to do that, since every AMD64 processor has hardware binary floating-point.

 > There is a fast path for addition of integers that takes only 5 instructions, the fast path for addition in hardware should take only 1 cycle when the two exponents are equal to each other and there is no overflow.

Okay, let's take a look at that magical single-cycle code,

```
; Add rdx to rax.
mov     cl,al         ; load the exponent of rax into cl
or      cl,dl         ; then or the two exponents together
jnz     slow_path     ; if both are zero, take the fast path
add     rax,rdx       ; add the coefficients together
jo      overflow      ; if there was no overflow, we are done
```

This is wrong in so many ways I'm going to have to make a list with like, three points in it. I'll assume he has the beefiest Intel processor around, the Haswell, since it is fast, relatively available, and faster than the fastest AMD.

Since Crockford was too lazy to read the optimization docs from Intel and do original research, I'm going to be lazy too, and just cite <http://www.agner.org/optimize/microarchitecture.pdf> instead, since Agner Fog is a lot better than me at this.

 1. Haswell can just issue four instructions in a single cycle, that is five. (10, page 135)
 2. `mov cl, al` isn't removed by register renaming (10.8, page 138), `or cl, dl` cannot be fused with `jnz` (9.6, page 124), and `add rax, rdx` cannot be fused with `jo` (9.6, page 124), which means that the longest dependent chain is `mov` -> `or` -> `jnz`, which takes 3 cycles if the branch predicts as not taken. (<http://www.agner.org/optimize/instruction_tables.pdf>)
 3. This only happens if the exponents are the same, which will only happen for code that only takes denormalized integers. But interestingly, if we used the correct datatype (an integer) then it only takes one cycle and doesn't risk a case that takes hundreds of cycles.

 > The fast path for multiplication in hardware takes the time it takes to do a 56*56 signed multiply when there is no overflow.

I don't see how this can end up well, considering the estimate for add was 3x as high. If the estimate for is just as bad, we'll estimate the cycle count at 9…

```
; Unpack the exponents in r8 and r9.
movsx r8,r1_b ; r8 is the first exponent
movsx r9,r2_b ; r9 is the second exponent

; Set flags in r0 if either operand is nan.
cmp r1_b,128 ; is the first operand nan?
sete r0_b ; r0_b is 1 if the first operand is nan
cmp r2_b,128 ; is the second operand nan?
sete r0_h ; r0_h is 1 if the second operand is nan

; Unpack the coefficients. Set flags in r1 if either is not zero.
sar r1,8 ; r1 is the first coefficient
mov r10,r1 ; r10 is the first coefficient
setnz r1_b ; r1_b is 1 if the first coefficient is not zero
sar r2,8 ; r2 is the second coefficient
setnz r1_h ; r1_h is 1 if the second coefficient is not zero

; The result is nan if one or both of the operands is nan and neither of the
; operands is zero.
or r1_w,r0_w ; is either coefficient zero and not nan?
xchg r1_b,r1_h
test r0_w,r1_w
jnz dec64_nan

mov r0,r10 ; r0 is the first coefficient
add r8,r9 ; r8 is the product exponent
imul r2 ; r2:r0 is r1 * r2

; The coefficient is in r0.
; The exponent is in r8.

mov r10,10 ; r10 is the divisor

; If the exponent is greater than 127, then the number is too big.
; But it might still be possible to salvage a value.

cmp r8,127 ; compare exponent with 127
jg pack_salvage

; If the exponent is greater than or equal to -127 and the top 9 bits of r0 are
; all the same, then we are ready to pack. If the top nine bits are all the
; same then the coefficient can survive the final left shift of 8 bits without
; a loss of significance. If the exponent is between -127 and 127, then it will
; fit too.

mov r11,r0 ; r11 is the coefficient
cmp r8,-127 ; compare the exponent with -127
setge r1_b ; r1_b is 1 if the exponent is not less than -127
sar r11,56 ; shift out all except the top 8 bits
adc r11,0 ; add the 9th bit and see if the result is 0
setz r1_h ; r1_h is 1 if the top bits are all the same
test r1_h,r1_b ; ready to pack?
jz pack_loop ; not yet
shl r0,8 ; shift the exponent into position
cmovz r8,r0 ; if the coefficient is zero, also zero the exponent
mov r0_b,r8_b ; mix in the exponent
```

Do I have to convince anyone that this does **not** take three cycles? No, didn't think so. So I'll leave it as an exercise, or you could check out some of the simpler ones, at <https://github.com/douglascrockford/DEC64>, and try to do those calculations yourself.

Motivation
----------

 > The idea of using powers of ten instead of powers of two is not new.

I agree, before computers, almost everyone used decimal in the western world. Even if there are of course counterexamples, hexadecimal abacuses and so on.

 > Floating point subroutines and interpretive systems for early machines were coded by D. J. Wheeler and others, and the first publication of such routines was in The Preparation of Programs for an Electronic Digital Computer by Wilkes, Wheeler, and Gill (Reading, Mass.: Addison-Wesley, 1951), subroutines A1-A11, pages 35-37 and 105-117. It is interesting to note that floating decimal subroutines are described here, although a binary computer was being used; in other words, the numbers were represented as 10^e f, not 2^e f, and therefore the scaling operations required multiplication or division by 10.

I'm too lazy to check it up, since I'm pretty sure the quote is correct and true.

 > The book Knuth cited may have been the first software book. It described some of the libraries and conventions of Maurice Wilkes’s EDSAC, one of the first generation of Von Neumann machines. Some of its subroutines used a numeric format that was very similar to DEC64.

First does not mean best. Also, both hardware and software may have changed slightly from EDSAC to Haswell, possibly.

 > Floating point was so important that support for it was moved into hardware for better performance. This led to the development of binary floating point because a shift could be implemented much more easily than a divide by 10.

There are numeric reasons as well, like a unique representation.
 
 > It was discovered that by biasing the exponent and moving it to the position just after the sign bit that floating point numbers could be compared with integer opcodes, a nifty optimization.

Yes, and it was quite important when we just had a few transistors around. Now it just allows geezers like me to be amused.

 > It was also discovered that because normalization always left a 1 bit in the most significant position of the significand, that that bit could be omitted, providing an additional bit of significance.

This bit isn't only useful, it also gives us the unique representation of a value that I love so much.

 > The Burroughs 5000 series had a floating point format in which an exponent of zero allowed the mantissa to be treated as an ordinary integer. DEC64 incorporates that brilliant idea.

The Burroughs 5000 did this to save encoding space and transistors, not because it was a brilliant idea. The Burroughs Large Systems were at least a decade ahead of the competition in many ways, but not everything about it was brilliant, there were a lot of manufacturing tradeoffs when building computers in 1961.

 > Languages for scientific computing like FORTRAN provided multiple floating point types such as REAL and DOUBLE PRECISION as well as INTEGER, often also in multiple sizes. This was to allow programmers to reduce program size and running time. This convention was adopted by later languages like C and Java. In modern systems, this sort of memory saving is pointless. By giving programmers a choice of number types, programmers are required to waste their time making choices that don’t matter. Even worse, making a bad choice can lead to a loss of accuracy or destructive bugs. This is a bad practice that is very deeply ingrained.

The Burroughs 6500 did this too, with single-precision and double-precision data tags. Yes, each word in memory was tagged with the type it contained!

Sure, we have almost infinite amounts of memory, but we have highly finite amounts of memory bandwidth. Also, the size of the L1 cache is teensy compared to main memory, and that is the fast part of memory. Haswell has 32kB of data cache per two threads and AMDs Jaguar (used in the Xbox One and Playstation 4) has 32kB per thread. That's just 512 numbers, using DEC64, does that still seem infinitely big?

 > Binary floating point trades away familiarity and decimal compatibility for performance.

No it doesn't, or yes it does, but not more so than decimal floating-point. You just have a false sense of security with decimal.

 > This made it unsuitable for business languages like COBOL.

It was unsuitable for business because it was floating-point, not because it was binary.

 > Decimal fractions cannot be represented accurately in binary floating point, which is a problem for programs that interact with humans, and is dangerous in programs that manipulate money. Exactness is required, so most business processing used BCD (Binary Coded Decimal) in which each digit is encoded in 4 bits.

And because it is fast to implement if you just have a few hundred transistors to play with and memory is faster than the core.

 > That created some inefficiency, but benefited from allowing a shift by 4 bits in place of the more complex divide by 10. For a time, mainframes could be ordered with optional floating point units for scientific computing, and optional BCD units for business computing.

Almost everything was optional when computers were still wired by hand. Having a timer was optional on the System/360.

 > The BASIC language eliminated much of the complexity of FORTRAN by having a single number type. This simplified the programming model and avoided a class of errors caused by selection of the wrong type. The efficiencies that could have gained from having numerous number types proved to be insignificant. 

It also eliminated a lot of other complexity, like variable names, scoping, and support for strings.

 > Business Basic was a dialect of BASIC that was developed by Basic/Four Corporation for its small business minicomputers. It used decimal floating point, much like the EDSAC, so the language could be used for both scientific and business applications. Business Basic could do everything that BASIC could do, and it could also handle money.

According to Wikipedia, Business Basic was a category of variants of BASIC. But sure, they could probably do whatever Dartmouth BASIC could do in 1964 and more.

 > Intel undertook an ambitious architecture that was marketed as iAPX432. A decimal number type was considered for the 432 but rejected in favor of conventional binary floating point.

Probably because they couldn't fit both a decimal floating point and a hardware garbage collector on the same chip. I'm sure they would have added it if they had the chance. For reference, the binary floating-point unit was probably over half of the GDP, a two-chip part of the CPU, which had 3 chips in total. Adding both a binary and a decimal floating-point would have made the iAPX 432 even more of a turd.

 > The 432 failed, but its floating point unit was salvaged and repackaged as the 8087, a numeric coprocessor for the 8086.

Surprisingly, the iAPX 432 was released in 1981, while the 8087 was released in 1980. I'm sure Intel knew iAPX 432 was a train wreck by then, but I'm also pretty convinced that the 8087 was more than salvaged and repacked. It likely was the reverse, the iAPX 432 used most of the design of the 8087.

 > The 8087’s number types became the basis of the IEEE 754 floating point standard. The standard was so successful that it destroyed the market for decimal computing.

Yes, it did destroy that market, because it existed in hardware already, in addition to actually having better performance and accuracy.

 > A later revision of IEEE 754 attempted to remedy this, but the formats it recommended were so inefficient that it has not found much acceptance. DEC64 is a better alternative.

Yes, decimal64 makes different trade-offs, but I haven't had a use for it, so I cannot say which one is the better format.

Summary
-------

I, personally, think DEC64 has few advantages over the binary floating-point we all have in our CPUs already. But I'm glad that the question has been raised before and swiftly destroyed on es-discuss, so I'm pretty sure this effort is dead in the water already.
