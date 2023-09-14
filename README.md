# Kusto Detective - Rookie - The rarest book is missing

## Introduction
Welcome to the actual first challenge. Let’s just say that it is expected that you have practiced since the last task. These tasks were released with a 2 week break in between each other. Considering the level of the query for the first task, this one takes it even further. Not to be worried, but if you are solving this by yourself, you’re in for more and more brain twisters.

The query itself makes totally sense once you get the correct answer, but going from a logic reasoning and transferring into a query that does what you want takes practice. I will try to make these walkthroughs more progressive than what the tasks may feel like.

Copy the query in the inbox text and run through it like we did in the onboarding case. It may take some time.

## The Story

"I want to add an image here?" - Sorry, the image is missing.

Let's dive into the story by reading the provided screenshot. How would you proceed from here? For me, the first step is to formulate a strategy by understanding the story and determining what to look for. Here are some keywords to extract from the story:

- 325,000 rare books
- The rarest book in the world, "De Revolutionibus Magnis Data," published in 1613 by Gustav Kustov.
- Each book has a number of pages and weight, with RFID stickers.
- RFID is collected for each shelf, indicating which books are present and the total weight.

In essence, the story revolves around a misplaced book without its RFID tag. Can you find it? My initial thought is to identify the shelf with the largest weight gap between the expected weight and the actual weight. If a book has been misplaced, the shelf's total weight will be higher than the sum of the RFID-tagged books.

My approach would be to calculate the expected weight on each shelf, compare it with the actual weight, sort it in descending order, and the answer should be the top result.

Let's move on to the hints to gather more information that could lead us in the right direction.

## Hints
### First hint
"I wanna to add image here? "
Take a more in depth look of the book of interest. I find this as a self explanatory thing to do when starting out. Find out what kind of data you are dealing with, and see if anything stands out with the element in question. Also, you might remember a few details that can trigger any lead when doing so.

### Second hint
"I wanna to add image here? "
This hint proves valuable as it leads me into some queries and keywords that was interesting. The first element locks down on shelf 5853 using where. The mv-expand function takes the whole list of RFID’s and creates a new row for each with their original belonging values residing in the original entry. From here we have every book with its shelf and the shelves total weight. Moving on the hint does a lookup. A lookup is similar to a join (it basically means to combine two tables), but has a few differences:

like non repeating columns from the $right table,
limited to leftouter (default) and inner (kind of join)
assumes $left is the larger table (opposite with join)
automatically broadcasts the $right table to the $left table
Microsoft has more info to dig deeper into.

So, what is lookup doing here? Well, it “joins” or looks up all the information from Books and broadcasts it based on rf_id from $right to $left. And then reducing numbers of columns shown by project. The columns you put after this will be the columns that are being part of the output.

### Third hint
"I wanna to add image here? "
Nothing is perfect, ey? Dust?

## Solution
Based on everything we have gone through now, we have a query from the hints that has taken us pretty far. We’ve been forced to try out a few different functions that seem to do a lot of what we are looking for. The query we are left with is:

```kql
Shelves 
| where shelf == "5853"
| mv-expand rf_id = rf_ids to typeof(string) 
| lookup Books on rf_id 
| project shelf, author, book_title, total_weight, weight_gram
```

What was our goal? We need to calculate the total weight of each book for each shelf and compare it to the actual weight that the shelf is claiming to have. How do we move forward? A better question is, how do we calculate on columns? Google this and see what you can figure out before moving forward!

There are a few ways to do calculations on columns’ values. The solution would be to use summarize for this. summarize groups the rows based on what columns you provide after by. This way we can calculate and name the calculated value and make it into its own column.

To get the calculated weight we should sum() the weight_gram (each books weight) and we also need to create a column with the shelves claimed weight by shelf. Then move on to another type of calculating on columns in KQL - extend. extend does calculation and adds it into a new extended column. Subtract claimedWeight by bookWeight. Here’s the solution!

```kql
Shelves 
| mv-expand rf_id = rf_ids to typeof(string)
| lookup Books on rf_id 
| project shelf, author, book_title, total_weight, weight_gram
| summarize bookWeight = sum(weight_gram), claimedWeight = min(total_weight) by shelf
| extend difference = (claimedWeight - bookWeight)
| order by difference desc
```

This gives us a very interesting shelf number indeed!
