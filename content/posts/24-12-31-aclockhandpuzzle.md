+++
categories = ["puzzles"]
title = 'A Clock Hand Puzzle'
date = 2024-12-31T12:33:33-05:00
draft = false
+++

I used to not like analog clocks because they unecessarily made it harder to tell time in a world where digital clocks are a reality. Now, I appreciate them a lot more for all the mathematical fun that they present.

So, here's a very simple puzzle I thought of while looking at one.

## The Puzzle
It is 3:00 right now on an analog clock. How much longer do I have to wait to see the minute and the hour hands cross each other?

## The Solution
Lets call the position of the minute hand and the hour hand be $x$,$y$, respectively. If we let the ranges be $x,y \in [0,60)$ then we can conveniently express $y$ as

$$y = \frac{5x}{60} + 15$$

Then, the problem becomes as simple as solving the systems of equations of

$$
\begin{cases}
    y = x \\
    y = \frac{x}{12}+15
\end{cases}
$$

Which is solved with

$$
\begin{align}
    x &= \frac{x}{12} + 15 \\
    11x &= 180 \\
    x &= 180/11 
\end{align}
$$

$$\implies x = 16.\overline{36}$$

Thus, we will see them cross for the first time at 3:16, which makes sense!


