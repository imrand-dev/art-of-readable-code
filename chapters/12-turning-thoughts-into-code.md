## Chapter 12: Turning Thoughts into Code

> You do not really understand something unless you can explain it to your grandmother. — Albert Einstein

When explaining a complex idea to someone, it’s easy to confuse them with all the little details. It’s a valuable skill to be able to explain an idea “in plain English,” so that someone less knowledgeable than you can understand. It requires distilling an idea down to the most important concepts. Doing this not only helps the other person understand but also helps you think about your ideas more clearly.

### Describing Logic Clearly

Here is a snippet of code from a web page in PHP. This code is at the top of a secured page. It checks whether the user is authorized to see the page, and if not, immediately returns a page telling the user is not authorized:

```php
$is_admin = is_admin_request();
if ($document) {
    if (!$is_admin && ($document['username'] != $_SESSION['username'])) {
        return not_authorized();
    }
} else {
    if (!$is_admin) {
        return not_authorized();
    }
}
```

Let’s start by describing the logic in plain English.

There are two ways you can be authorized:

- you are an admin
- you own the current document (if there is one)

Otherwise, you are not authorized.

Here is an alternative solution inspired by this description:

```php
$is_admin = is_admin_request();
if ($is_admin) {
    // permission granted
} elseif ($document && ($document['username'] == $_SESSION['username'])) {
    // permission granted
} else {
    return not_authorized();
}
```

### Applying This Method to Larger Problems

Imagine we have a system that records stock purchases. Each transaction has four pieces of data:

- `time` (a precise date and time of the purchase)
- `ticker_symbol` (e.g., GOOG)
- `price` (e.g., $600)
- `number_of_shares` (e.g., 100)

For some strange reason, the data is spread across three separate database tables, as illustrated here. In each database, the `time` is the unique primary key.

![stock-price](https://i.ibb.co/37qkgJh/Untitled.png)

Now we need to write a program to join the three tables back together. This step should be easy because the rows are all sorted by `time`, but unfortunately some of the rows are missing. You want to find all the rows where all three times match up, and ignore any rows that can’t be lined up, as shown in the previous illustration.

Here is some Python code that finds all the matching rows:

```py
def printStockTransactions():
    stock_iter = db_read("SELECT time, ticker_symbol FROM ...")
    price_iter = ...
    num_shares_iter = ...

    # Iterate through all the rows of the 3 tables in parallel.
    while stock_iter and price_iter and num_shares_iter:
        stock_time = stock_iter.time
        price_time = price_iter.time
        num_shares_time = num_shares_iter.time

        # If all 3 rows don't have the same time, skip over the oldest row
        # Note: the "<=" below can't just be "<" in case there are 2 tied-oldest.
        if stock_time != price_time or stock_time != num_shares_time:
            if stock_time <= price_time and stock_time <= num_shares_time:
                stock_iter.NextRow()
            elif price_time <= stock_time and price_time <= num_shares_time:
                price_iter.NextRow()
            elif num_shares_time <= stock_time and num_shares_time <= price_time:
                num_shares_iter.NextRow()
            else:
                assert False  # impossible
            continue

        assert stock_time == price_time == num_shares_time
        
        # Print the aligned rows.
        print "@", stock_time,
        print stock_iter.ticker_symbol,
        print price_iter.price,
        print num_shares_iter.number_of_shares

        stock_iter.NextRow()
        price_iter.NextRow()
        num_shares_iter.NextRow()
```

This example code works, but there’s a lot going on with how the loop skips over unmatched rows. Some warning flags might have gone off in your head: *Could this miss any rows? Might it read past the end-of-stream for any of the iterators?*

So how can we make it more readable?

Let’s step back and describe in plain English what we’re trying to do:

- We are reading three row iterators in parallel.
- **Whenever the rows' times don't line up, advance the rows so they do line up.**
- Then print the aligned rows, and advance the rows again.
- Keep doing this until there are no more matching rows left.

Looking back at the original code, the messiest part was the block dealing with “advance the rows so they do line up.” To present the code more clearly, we can extract all that messy logic into a new function named `advanceToMatchingTime()`.

```py
def printStockTransactions():
    stock_iter = ...
    price_iter = ...
    num_shares_iter = ...

    while True:
        time = advanceToMatchingTime(stock_iter, price_iter, num_shares_iter)
        if time is None:
            return

        # Print the aligned rows.
        print "@", time,
        print stock_iter.ticker_symbol,
        print price_iter.price,
        print num_shares_iter.number_of_shares

        stock_iter.NextRow()
        price_iter.NextRow()
        num_shares_iter.NextRow()
```

### Applying the Method Recursively

```py
def advanceToMatchingTime(stock_iter, price_iter, num_shares_iter):
    # Iterate through all the rows of the 3 tables in parallel.
    while stock_iter and price_iter and num_shares_iter:
        stock_time = stock_iter.time
        price_time = price_iter.time
        num_shares_time = num_shares_iter.time

        # If all 3 rows don't have the same time, skip over the oldest row
        if stock_time != price_time or stock_time != num_shares_time:
            if stock_time <= price_time and stock_time <= num_shares_time:
                stock_iter.NextRow()
            elif price_time <= stock_time and price_time <= num_shares_time:
                price_iter.NextRow()
            elif num_shares_time <= stock_time and num_shares_time <= price_time:
                num_shares_iter.NextRow()
            else:
                assert False  # impossible
            continue

        assert stock_time == price_time == num_shares_time
        return stock_time
```

But let’s improve that code by applying our method to `advanceToMatchingTime()` as well. Here’s a description of what this function needs to do:

- Look at the times of each current row: if they're aligned, we're done.
- Otherwise, advance any rows that are "behind."
- Keep doing this until the rows are aligned (or one of the iterators has ended).

This description is a lot clearer and more elegant than the previous code. One thing to notice is that the description never mentions `stock_iter` or other details specific to our problem. This means we can also rename the variables to be simpler and more general. 

```py
def advanceToMatchingTime(row_iter1, row_iter2, row_iter3):
    while row_iter1 and row_iter2 and row_iter3:    
        t1 = row_iter1.time
        t2 = row_iter2.time
        t3 = row_iter3.time

        if t1 == t2 == t3:
            return t1

        tmax = max(t1, t2, t3)

        # If any row is "behind," advance it.
        # Eventually, this while loop will align them all.
        if t1 < tmax: row_iter1.NextRow()
        if t2 < tmax: row_iter2.NextRow()
        if t3 < tmax: row_iter3.NextRow()

    return None  # no alignment could be found
```

### Summary

This chapter discussed the simple technique of describing your program in plain English and using that description to help you write more natural code. This technique is deceptively simple, but very powerful. Looking at the words and phrases used in your description can also help you identify which subproblems to break off.

But this process of “saying things in plain English” is applicable outside of just writing code. For example, one college computer lab policy states that when a student needs help debugging his program, he first has to explain the problem to a dedicated teddy bear in the corner of the room. Surprisingly, just describing the problem aloud can often help the student figure out a solution. This technique is called “rubber ducking.”