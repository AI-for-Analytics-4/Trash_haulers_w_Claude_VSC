## Missed Trash Pickups

For this exercise, you'll be using Claude in VS Code to assist you. Claude should be treated as a **pair programmer**, not an automatic solution generator.

Get the data here: https://docs.google.com/spreadsheets/d/1O7aOHvppfeuzthEQ1pI9Y8cr5IlJOcvCzEYOLPxlM9g/edit?usp=drive_link

In this data question you will be working data of service request related to missed trash pickups from hubNashville, Metro Nashville government's comprehensive customer service system (https://hub.nashville.gov).

As part of Metro's contract with Red River Waste Solutions, failure to remedy an action or inaction will result in liquidated damages. One category of liquidated damages is related to chronic problems in any category of service at the same premises. A chronic problem is defined as more than one missed pickup for any address. The first missed pickup will not result in a fine; however, every subsequent missed pickup will result in a $200 fine.

Claude tends to be very eager to just do the work for you. You need to change the role that you give it and structure your requests so that adding code is explicitly disallowed unless invited.
For example, you can say

"I’m working in Python. I want to think through the approach to this problem.
Do not write any code yet. Explain the logic, assumptions, and possible approaches in plain English only."

**Part 1: Understanding and Planning (No Code Yet)**
Before writing any code, use Claude to help you think through the problem.

Ask Claude:  
* How would you identify repeat missed pickups at the same address?
* What assumptions need to be made about how addresses are standardized?  
* What edge cases could cause over- or under-counting fines?

Write a short paragraph summarizing the approach you plan to take.

**Part 2: Cleaning and Identifying Missed Pickups**
Now, begin coding. When you do this, make sure that you are reviewing the code that Claude is recommending. Note that pandas methods are often more efficient to using a custom function, so you may need to guide Claude in that direction.
1. Ask Claude to help you clean and standardize the address column.  
* Ask Claude to explain its approach before generating code.  
* Review and edit the code before running it.
2. Identify rows that represent missed pickups.
* Have Claude help you manually check a small subset of rows or addresses.

**Part 3: Calculating Fines (Base Rule)**
Using the rule:  
* First missed pickup at an address -> $0 fine  
* Every subsequent missed pickup -> $200 fine
1. Ask Claude to suggest an approach for calculating fines.
2. Implement the approach step-by-step.
3. Add at least one validation check (eg. inspect a single address manually).

Report:  
* Total amount of missed pickups  
* Total number of fined pickups  
* Total amount of damages

**Part 4: Challenge Rule**
Metro's actual contract specifies that a $500 fine applies if a missed pickup is the third or higher at the same address within a 6-month rolling window.

1. Ask Claude for multiple possible approaches to this problem, with an emphasis on efficiency. Claude may suggest inefficient solutions, so you may need to guide it towards rolling or window-based methods.  
2. Choose an approach and explain why.  
3. Implement the solution.  
4. Do some checks on the solution.  
