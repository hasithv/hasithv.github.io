+++
categories = ["puzzles"]
tags = ["probability"]
title = 'A Simple Boarding Puzzle'
date = 2024-08-12T00:00:38-05:00
draft = true
+++

## The Puzzle
_Inspired by true events_

Alice is assigned to be the 56th passenger to board a full plane with 60 seats. However, a panic causes all the passengers--including Alice--to arrange themselves radomly in line to board.

As Alice was originally 56th, she decides that she would be happy as long as passengers with the assigned spots 57, 58, 59, and 60 are not in front of her. What is the probability that Alice will be happy?

## Solution
The only passengers we need to consider are Alice and passengers 57, 58, 59, and 60--since for each arrangement of these 5 passengers, there is an equal amount of ways to arrange all other 55 passengers.

Thus, the probability will simply the number of ways we can arrange these 5 passengers such that Alice is in front of passengers 57, 58, 59, and 60 divided by the total number of ways to arrange these 5 passengers:

$$ 4!/5! = 1/5$$
