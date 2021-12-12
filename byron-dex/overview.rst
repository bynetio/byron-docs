Overview
========

Our DEX model is based on **Order Book Pattern (OBP)**
combined with a **2-phase commit pattern**. The model
distinguishes the commit and execution phases. In the
committing phase, actors submit swap orders with an
expectation of a particular amount of an opposite coin. The
execution phase matches orders and performs them. Successful
execution locks money in the script for the beneficiary.

Orders
------

Orders are the core of OBP. Order is represented as UTxO,
which contains all information needed to perform a swap on
it. There may be different types of orders. For now, we
implemented the following:

-  Limit order (sell order) - the simplest type of order. It
   is given by swappers. It contains the tokens that the
   swapper wants to replace and they can be used only when
   an appropriate payout is created for the owner of the
   order (UTxO, which can only be used by that person).
-  Liquidity order - prepared with liquidity providers in
   mind. It contains information about which two tokens (in
   specific their quantities) will be exchanged between each
   other, and the amount of the fee. For example: we have a
   liquidity order between 100A and 200B, with a commission
   of 1%. In the beginning, we create it as UTxO, and add
   100A to it. If someone wants to receive 100A, they will
   have to create a new liquidity order, this time between
   202B (101% of 200) and 100A, and will have to create 202B
   back. If someone wants to get this 202B, they will have
   to create a liquidity order between 101A and 202B and put
   101A in it. This works until the original owner cancels
   the order - then he receives all the funds stored in this
   UTxO. The more swaps were made on this liquidity order,
   the more profit the person submitting it will receive.

Actors
------

We have 3 types of actors:

-  **swappers** - people wishing to exchange one token for
   another. They want the exchange to take place as soon as
   possible and with the best exchange rate.

-  **liquidity providers** - people with funds willing to
   provide liquidity. They want to earn as much as possible
   on their investment.

-  **performers** - people (or bots) executing orders. They
   look for orders that match and choose the ones from which
   they can earn. For example, Alice wants to trade 10A for
   9B and Bob wants to trade 10B for 9A. If we take these
   two orders and execute them in one transaction, Alice
   gets 9B, Bob gets 9A, and the performer gets 1A and 1B.
   Performers want to be able to easily analyze existing DEX
   orders and be able to combine as many orders as possible
   into one transaction.

Liquidity Swamps
------------------------------------------

Liquidity Swamps are a sets of discrete Liquidity Orders,
where each order has a slightly different rate. The
distribution of subsequent orders can be determined on the
basis of the *x \* y = k* curve, but there is nothing to
prevent the use of other curves as well. Every Liquidity
Order is an independent exchange offer, but all of them
together simulate the operation of the classic Liquidity
Pool. The Liquidity Provider can add their funds in swamps
of liquidity instead of through one or more Liquidity Orders
with identical rates. Thus, his funds earn better in an
environment of ever-changing cryptocurrency prices.

Imagine a traditional Liquidity Pool. It is some kind of
system containing large amounts of two different tokens (A
and B) and allowing for pulling them out as long as certain
rules are adhered to. For example, we can make it so the
product of the quantities of both coins can never decrease,
and each swap must additionally draw 0.3% less than it would
like (a commission). If we analyze such a pool, it turns out
that each consecutive swap of coin A for coin B is less and
less profitable. If we were to establish that each swap
could move only 100 coin A (and as much B as the math of
this pool requires), we could **divide the entire pool into
a set of independent exchange offers.**

Not every offer is equally profitable - only those on the
top will be close to the market rate. However, when the
global market price of coins fluctuates, the A to B swap
offers will disappear faster than B to A (or vice versa).
The situation will stabilize when the most profitable offer
remains close to the market price.
