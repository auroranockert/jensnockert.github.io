---
layout: post
title: Exploding dice
date: '2012-07-26T00:01:24+02:00'
tags: 
tumblr_url: http://blog.aventine.se/post/28007546498/exploding-dice
mathjax: true
---
It is too warm to do any actual work in Sweden today, so I instead sat in my apartment, playing Classic Battletech with myself, and after a while; I thought that since I am an adult and can have any imaginary friends I like, that I wanted to incorporate some Mechwarrior (the accompanying role-playing game to Classic Battletech) characters to create a more interesting environment to lose time in.

One of the characteristics of the Mechwarrior game is that it uses *2d10* (two ten-sided dice) for almost anything, but with the special rule that any 10 is rolled again, adding to the total. For example, if I roll a *4* and a *7*, my result is *11*, but if I roll a *6* and a *10*, I get to roll once more, maybe a result of *3*, for a total of *19*. You can continue rolling dice for a long while, in theory, an infinitely long while.


But what is the mean value of such a roll?
---------------------------------------------------------------------------------

We'll reduce the *2d10* case to a simple *d10* (one ten-sided die), because the two dice are independent of each other. So, to calculate the mean, we just have to sum up each possible value, multiplied by its probability, simple.

$$\frac{1 + … + 9}{10} + \frac{11 + … + 19}{100} + \frac{21 + … + 29}{1000} + …$$

Except that sum is not really that simple, and not really that general, we can simplify it a bit more, each addend for example, if we are using \\(n\\)-sided dice, and we are in the \\(i\\)-th explosion.

$$\frac{k \times n \times (n - 1) + n(n  1) / 2}{n^(k + 1)} = \frac{(n - 1) \times (k + 1 / 2)}{n^k}$$

Then we can build the sum,

$$\sum_{k = 0}^{\infty}{\frac{(n - 1) \times (k + 1 / 2)}{n^k}}$$

and we can calculate some prefix sums for \\(n\\) = 10, to some arbitrary precision,

<table>
	<tr>
		<th>\(i\)</th>
		<th>\(\sum\)</th>
	</tr>
	<tr>
		<td>0</td>
		<td>4.50000</td>
	</tr>
	<tr>
		<td>1</td>
		<td>5.85000</td>
	</tr>
	<tr>
		<td>2</td>
		<td>6.07500</td>
	</tr>
	<tr>
		<td>3</td>
		<td>6.10650</td>
	</tr>
	<tr>
		<td>4</td>
		<td>6.11055</td>
	</tr>
	<tr>
		<td>5</td>
		<td>6.11105</td>
	</tr>
	<tr>
		<td>6</td>
		<td>6.11110</td>
	</tr>
</table>

and we see that it seems to tend towards \\(\frac{55}{9}\\), or \\(\frac{10}{9}\\) times better than a normal die. In the general case, for \\(n > 1\\),

$$\frac{n(n + 1)}{2(n - 1)}$$

or \\(\frac{n}{n - 1}\\) times better than the regular, non-exploding die.


Comparing to D20
---------------------------------------------------------------------------------

Another common system, with similar qualities is D20, which uses a single *d20* dice, but with the special property that a *20* always succeeds. Let us compare the two systems, based on which TN (target-number) that you need to succeed.

In both systems, the lowest roll possible, always fails.

<table>
	<tr>
		<th>TN</th>
		<th>d20</th>
		<th>2d10</th>
	</tr>
	<tr>
		<td>2</td>
		<td>95.0%</td>
		<td>99.0%</td>
	</tr>
	<tr>
		<td>3</td>
		<td>90.0%</td>
		<td>99.0%</td>
	</tr>
	<tr>
		<td>4</td>
		<td>85.0%</td>
		<td>97.0%</td>
	</tr>
	<tr>
		<td>5</td>
		<td>80.0%</td>
		<td>94.0%</td>
	</tr>
	<tr>
		<td>6</td>
		<td>75.0%</td>
		<td>90.0%</td>
	</tr>
	<tr>
		<td>7</td>
		<td>70.0%</td>
		<td>85.0%</td>
	</tr>
	<tr>
		<td>8</td>
		<td>65.0%</td>
		<td>79.0%</td>
	</tr>
	<tr>
		<td>9</td>
		<td>60.0%</td>
		<td>72.0%</td>
	</tr>
	<tr>
		<td>10</td>
		<td>55.0%</td>
		<td>64.0%</td>
	</tr>
	<tr>
		<td>11</td>
		<td>50.0%</td>
		<td>55.0%</td>
	</tr>
	<tr>
		<td>12</td>
		<td>45.0%</td>
		<td>47.0%</td>
	</tr>
	<tr>
		<td>13</td>
		<td>40.0%</td>
		<td>39.8%</td>
	</tr>
	<tr>
		<td>14</td>
		<td>35.0%</td>
		<td>33.4%</td>
	</tr>
	<tr>
		<td>15</td>
		<td>30.0%</td>
		<td>27.8%</td>
	</tr>
	<tr>
		<td>16</td>
		<td>25.0%</td>
		<td>23.0%</td>
	</tr>
	<tr>
		<td>17</td>
		<td>20.0%</td>
		<td>19.0%</td>
	</tr>
	<tr>
		<td>18</td>
		<td>15.0%</td>
		<td>15.8%</td>
	</tr>
	<tr>
		<td>19</td>
		<td>10.0%</td>
		<td>13.4%</td>
	</tr>
	<tr>
		<td>20</td>
		<td>5.0%</td>
		<td>11.8%</td>
	</tr>
	<tr>
		<td>21</td>
		<td>5.0%</td>
		<td>10.0%</td>
	</tr>
	<tr>
		<td>22</td>
		<td>5.0%</td>
		<td>8.4%</td>
	</tr>
	<tr>
		<td>23</td>
		<td>5.0%</td>
		<td>7.0%</td>
	</tr>
	<tr>
		<td>24</td>
		<td>5.0%</td>
		<td>5.7%</td>
	</tr>
	<tr>
		<td>25</td>
		<td>5.0%</td>
		<td>4.6%</td>
	</tr>
	<tr>
		<td>26</td>
		<td>5.0%</td>
		<td>3.7%</td>
	</tr>
	<tr>
		<td>27</td>
		<td>5.0%</td>
		<td>3.0%</td>
	</tr>
	<tr>
		<td>28</td>
		<td>5.0%</td>
		<td>2.4%</td>
	</tr>
	<tr>
		<td>29</td>
		<td>5.0%</td>
		<td>2.0%</td>
	</tr>
	<tr>
		<td>30</td>
		<td>5.0%</td>
		<td>1.7%</td>
	</tr>
</table>

The most interesting parts about the table, is around TN 13 where the automatic success for a single exploding die, which means that the probabilities go a bit funny there, something to watch out for when designing a game with exploding die. The same thing happens higher up in the table as well, but there it doesn't make much difference.


Conclusion
---------------------------------------------------------------------------------

In the end, if you want your rolls to be average more often, fail less often, but with a quite exciting system for heroic success, use the exploding dice system. If you want a system that is easer to calculate the probabilities on, use a single-die system with automatic success.

I calculated everything by hand, so be a bit wary about my calculations. I only did some quick checking with numerics the result ended up similar.