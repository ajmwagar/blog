
---
title: Software Engineering for Systems Engineers (Mechanics, Grocery Shoppers, and Rocket Scientists)
date: 2024-12-05T00:00:00+00:00
draft: false
toc: false
tags:
  - basics
  - systems-engineering
  - software-engineering
  - code
  - software
  - career
  - ai
---

## Software vs Systems

While discussing the recent jump in Bitcoin prices since election night, a friend of mine today remarked that he wished he was a "Software Engineer" and understood crypto better. This friend works at an Aerospace company, and manages the creation of systems far more complex than a blockchain, especially Bitcoin's which doesn't have things like smart contracts, or Proof-of-Stake. 

"It's all systems, and you understand systems quite well." I quipped back. 

And he did. With almost a decade of experience on me, he had worked on many many projects, some of which I would dream to be able to work on, but don't have an aeronautical background.

It got me thinking, people approach technology, specifically software, as this out of reach, mysterious, almost magical concept. But the same people who are perplexed, or bewildered by software, many a time, have the problem solving, and systems skills to truly understand and appreciate software.

I know many mechanically inclined people. I've been on race teams where we've taken apart engines in the pits, and put them back together before the green flag the next morning. These same people often remark how magical software seems.

Funnily enough, many of the software engineers I know, struggle greatly with Systems Design. They can do Leetcode hard problems with their eyes closed all day long, but systems design seems to send them in a loop rather quickly.

I want to take step back, and talk about software at scale. Because at the end of the day, it's no different than a system like an engine in a car, or a rocket. In fact, I'd wager it's far far more simple than either of those.

> Software is just a complex system, where the components are made out of code, not metal.

## Where to start?

Let's go back to the engine metaphor. An internal-combustion engine is a rather simple concept. "Suck, Squeeze, Bang, Blow!" - can sum up the four states a cylinder goes through to make power.

1. *Suck* - Air comes in through the intake valve. Either naturally aspirated, or forced induction (turbo, supercharger, things that go stu-stu-stu-stu).
2. *Squeeze* - A fuel injector, sprays fuel into the air mixture. Shortly after, the piston starts an upstroke, compressing the air fuel mixture.
3. *Bang* - A spark, and then an explosion. The piston is forced back down, and power is made.
4. *Blow* - The piston comes back up again, forcing the spent mixture and exhaust gases out the exhaust valve.

That's it. 4-stroke engine. Obviously, we're leaving out cooling, and lubrication in our model, but for simplicities sake, that's okay.

When we talk about systems design in software. We often are trying to maximize the "throughput", or the efficient use of resources (storage/compute/memory).

Engines make power at the cost of fuel. Let's talk about the bottlenecks of this system. I already hinted at one, when we talk about a NA engine versus a turbo/super-charged one.

- More air, more fuel, more power. - If we want to increase the amount of fuel we can burn per combustion cycle, we need to increase the amount of air in the cylinder. This is usually done with what's called forced induction. A turbocharger, takes the exhaust gases to spin a turbine, which compresses air and rams it into the cylinder.

- There is also a matter of heat, and friction. Coolant and Oil is used to address these. 

But again, we're here to talk about software. Luckily for us, software is usually a lot more simple.

## A Trip to the Store... 

In software we talk a lot about **time complexity** and **space complexity**. These are references to how much more time or space a program/algorithm needs depending on how much you grow it's input. 

Let's go shopping. Say you have a grocery list. With N items. It can be 5, it can be 1000. 

Are you a perfect shopper? Going down each aisle only once, and getting all the items you need from it? Or are you a bit scatterbrained, and find yourself going back down each aisle for each item on your list?

Let's break down the difference. Here is some "pseudo-code", that describes our scatterbrained approach:

```
Enter Store.

For Item on Grocery List:
  - Find Aisle.
  - Put Item in Cart.

Checkout.

Leave Store.
```

This becomes an interesting example, because we can quickly talk about best-case, and worst-case scenarios.

Say this was the list:

### Item - Aisle
- Milk - Dairy
- Sourdough - Bread
- Beer - Beverages

See, in this scenario, we only visit each aisle once.

But if the list changes...

### Item - Aisle
- Milk - Dairy
- Sourdough - Bread
- Butter - Dairy
- Beer - Beverages
- Tortillas - Bread

We can repeat aisles needlessly.

Best case, this is what we call **linear time** where the list grows linearly, to the input. Or in "Big-O" notation, `O(n)`.
Worst case? Well things get a little worse. We'd be in **quadratic time** where we grow the amount of work quadratically to the input size. `O(n^2)`. 


### Optimizing...

If we wanted to optimize ourselves, we could change our shopping algorithm.

```
Enter Store.

For Item on Grocery List:
  - Make an Aisle List's if it doesn't already exist for this Aisle.
  - Put Item on Aisle's List.

For Aisle in Aisles:
    For Item on Aisle's List:
      - Find Item in Aisle.
      - Put Item in Cart.

Checkout.

Leave Store.
```

This changes our worst-case time complexity greatly. From `O(n^2)` to simply `O(n)`.

Best case or worst case, we only visit each aisle once, and only read the entire grocery list, one time.

Obviously, it's a bit of an unnatural process for a single person. But say you are in a rush but with a group. Breaking up the list into sub lists, and giving each list to a friend. You can shop "in parallel". Optimizing your time complexity to `O(n/p)`. With `p` being the number of people in your group.

That gets us to the next concept, parallelism, and concurrency.

## Many hands makes light work.

Whether we are splitting up the grocery list, or are a cylinder in an engine. We often deal with both "concurrency" and "parallelism".

What's the difference? 

*Parallelism* means you have many people/servers/things performing the same task, at the same time. Back to our grocery store metaphor, giving each person a list and having them shop at once, means you are in parallel.

*Concurrency*, fits better into our engine metaphor. Say you have an inline four motor. You may know that an engine like this has a firing order. It depends on the engine, but commonly 1-3-4-2 would be used.

Remember, each cylinder has 4-strokes (or states) it can be in, with an inline four, each cylinder is uniquely in it's own state, no matter what. Fixed to this rhythm with a crankshaft, we're describing a concurrent system. One cylinder makes power every stroke, working concurrently, but not in parallel.

In software, both parallelism and concurrency are used to make systems more efficient. Sometime at the same time.

---

Software is very adaptable, as we saw above, our algorithm can handle any sized grocery list as efficiently as possible. However, because of the wide range of inputs, there sometimes are situations where we may need to optimize further than simply "best-case" or "worse-case".

Say our grocery list was very biased towards beer and wine for a day (long weekend, camping trip, what have you). Your beer-wine aisle group member may take many times over what the dairy or bread group member.

How do we combat that? Is it worth combatting that? These are all questions that a software engineer needs to consider. 

Sometimes, it's worth leaving inefficiencies like this in our systems, because the frequency of them occurring is low enough that we can consider them "edge cases."

But I really despise waiting for other people to finish shopping. So let's tackle this biases efficiency a little more.

Here is our bias-affected parallel shopping algorithm.

```
Enter Store.

For Item on Grocery List:
  - Make an Aisle List's if it doesn't already exist for this Aisle.
  - Put Item on Aisle's List.

For Person in Group:
  - Assign Aisle to Person.

Each Person does at the same time:
  For Item on Aisle's List:
    - Find Item in Aisle.
    - Put Item in Cart.

Checkout.

Leave Store.
```

It's a pretty straight forward algorithm. But let's make it better.


```
Enter Store.

Let P = People in Group.
Let I = Items in Grocery List.

Let IP = I / P. # Items per person.

Let P_IDX = 0

For Item in Grocery List, Sorted by Aisle:
  - Take ceil(IP) items, assign to Group[P_IDX].
  - P_IDX = (P_IDX + 1) % P. # If the "%" is confusing, Google "Modulo", it'll change your life.
  - Skip to next item.
  - Stop when out of items.
  
Each Person does at the same time:
  For Item in Aisle's List for Aisle on Person's List:
      - Find Item in Aisle.
      - Put Item in Cart.

Checkout.

Leave Store.
```

Now please bear with the Psuedo code here...

This approach has three stages each with their own time complexity.

1. Sorting: `O(n*log(n))`
2. Assignment: `O(n)`
2. Execution: `O(I/P)` with `I` being Items on the Grocery List, and `P` being members of the group

The fun thing about Big-O notation is that it reduces. `O(n)` ~= `O(2n)`. Since it still grows linearly.

In our hyper optimized example here, the biggest time complexity, is sorting the list by aisle (`O(n*log(n))`). Now obviously, in real life, walking to each aisle, and grabbing the items from it will take the longest amount of time. But since we can change `I/P`. We could theoretically get to `O(1)` if we have enough people to each get one item.

Again, practically, versus theoretically, our approach would change. But splitting up the list to people, like above, is a practically good solution.

We could theoretically refine it further, by sending finished shoppers to help others with remaining tasks, and prioritizing the longest items (waiting at the deli, or getting an item that's locked up) could help reduce the overall shopping time. But your friends probably wouldn't put up with such a complex plan.

## Building Better Systems...

Without getting lost in metaphor. It's pretty easy to translate building software systems into day-to-day tasks, and items.

I haven't touched on true "distributed systems" or "fault tolerance" or "networking". But that's okay. These are just ways to describe complex problems with software/tech jargon. 

"Networking" is just a computer version of having a conversation, where you ask questions, or assign someone a grocery list to go find.

"Fault Tolerance" is just having to make a substitution for an item that is out at the store. Or having that person come back and say they couldn't find anything, and then making due from there.

"Distributed Systems" is just taking this approach, and having multiple shoppers goto different stores.

> Software is just Systems and Systems make up the world.

So the next time you hear about some magical algorithm or AI, just remember that at the end of the day, it's just some system someone came up with... that can likely be explained to you in plain english (that is, if more software engineer types had the soft skills to explain it to you...).

Even better, once you start thinking in systems you can't stop. I remember a long time ago I heard a cybersecurity professional tell me that once you start thinking about how to exploit a system regularly, you'll find yourself doing it non-stop. 

I find it's similar if you just simply ask "how does that work?" about your surroundings, or the apps on your phone. When you to see the world as logical, discrete systems, the world becomes small, and the chaos quells.
