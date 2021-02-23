---
title: Software Engineering Laws
tags:
   - Software Engineering
   - Code
   - Laws
---

Latest week a college shared the following [article](https://www.netmeister.org/blog/software-engineering-laws.html) at work that i found very interesting aggregating the info here in order not to forget.

## Conway's Law

Also known as: "You will ship your org chart."

> "Any organization that designs a system will produce a design whose structure is a copy of the organization's communication structure."

You may think you can avoid it via cross-functional standup meetings and stakeholder updates and decision matrices, but eventually and inevitably conflicting or diverging priorities will lead to equally conflicting or divergent processes and outcomes.

## Brooks's Law

From The **Mythical Man-Month** Book:

> "Adding manpower to a late software project makes it later."

When you realize you're not making the progress you thought you would and your management tries to reallocate resources from another part of the org, you'll end up not only delaying the project, but you'll likely ship a more brittle, more complex product

## Zawinski's Law

> "Every program attempts to expand until it includes a web server. Those programs which cannot so expand are replaced by ones which can."

For web services, it's "...until it requires a user account and collects all users' data"; for physical devices, it's "...until it includes an insecure wifi access point with a default password you can't change. And a web server".

## Parkinson's Law

> "Work expands so as to fill the time available for its completion."

The primary project management lessons here is that if you don't set a deadline, then the project will never be completed. This is why iterating on a minimum viable product within fixed timelines is important.

And, of course, we can and should adjust this law for data, processing power, RAM, ...:

> "Data/CPU/memory usage expands to use up all available storage space/bandwidth/cycles/RAM."

## Pareto's Fallacy

The [Pareto Principle](https://en.wikipedia.org/wiki/Pareto_principle) is easy to misinterpret, especially by management. This often leads to Pareto's Fallacy:

> "When you're 80% done, you think you only have 20% left."

The critical part that's overlooked here is that those 20% will require 80% of your time.

## Sturgeon's Revelation

> "90% of everything is crud."

Yes, that includes your products.

## The Peter Principle

> "In a hierarchy, every employee tends to rise to their level of incompetence. Thus, in time, every post tends to be occupied by an employee who is incompetent to carry out its duties."

Dunning and Kruger send their regards...([ref](https://en.wikipedia.org/wiki/Peter_Principle))

## Eagleson's Law

> "Any code of your own that you haven't looked at for six or more months might as well have been written by someone else."

6 months is rather optimistic.

## Greenspun's 10th Rule of Programming

This can be generalized to the Universal NIH-Rule: "any custom developed system contains an ad hoc, informally-specified, bug-ridden, slow implementation of half of the industry standard you refused to adopt".

## The Iceberg Fallacy

> "The cost of development of a new software product is the only ~25% of the total cost of ownership management sees and budgets for."

## The LGTM Dilemma

> "If you want to quickly ship a 10 line code change, hide it in a 1500 line pull request."

Also known as The Bikeshedders' Blind Spot.
