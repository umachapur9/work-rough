Question 1
hook+eecfc488-
139
HackerRank C
The manager of the Amazon warehouse has decided to make changes to the inventory. Currently, the inventory has n products, where the quality of the th product after quality checks is represented by the array element qualityli.
dentia
031
hook.com
The manager wens to create an optimal inventory, where the array of product quality flowe the
following property:
• All occurrences of each quality value must be contiguous.
HackerRank
In order to convert the inventory into an optimal inventory, the manager can do the following operation any number of times:
1. Choose two quality values x and y.
2. Replace every product with quality x to have quality y instead.
3. This operation costs num_replacements units of money, where num_replacements is the number of
@hook.c
products whose quality was changed.
393-4301-94174
Given n products and an array quality, find the minimum amount of money the manager has to spend to convert the inventory into an optimal inventory.
Hac
Note: The quality of a product can be negative indicating that the product is of poor quality.
Example:
Given n = 7, quality = [7, 7, 5, 7, 3, 5, 3).
Confidential
One of the optimal ways to convert is explained below:

7
7
Replace all products having quality 7
with quality 5
Spends 2 units of money
Replace all products having quality 3
with quality 7
Spends 2 units of money
7
7
7
7
7


Hence, the total amount spent is 4.
+
aecfc488-1393-430f-9417-92004a1
Function Description
Complete the function getMinAmount in the editor below. getMinAmounthas the following parameter(s): int qualityln: the quality of products
Hack
Returns
inventory.
int: the minimum amount of money the manager has to spend to convert the inventory into an optime 03G
HackerRan
Constraints
• 1sns2*105
• -10° s quality/il = 10°

Sample Case 1
Sample Input For Custom Testing
STDIN
FUNCTION
----.
11
10

quality[] size n = 11
6
10
-3
1
quality = [10,6,610,-3,1,1,4,-4,-1,1,7]
4
-1
1
-7

Sample Output 
4
Explanation
Initial inventory - 110, 6, 10, -3, 1, 1, 4, -4, -1, 1, -7]
Operation 1 (replace all products of quality 6 to 10): 10, 10, 10, -3, 1, 1, 4, -4, -7, 1, -71
Operation 2 (replace all products of quality 4 to 1): 10, 10, 10, -3, 1, 1, 1, -4, -1, 1, -71
Operation 3 (replace all products of quality -4 to 1): 10, 10, 10, -3, 1, 1, 1, 1, -7, 1, -71 
Operation 4 (replace all products of quality -1 to 1): 0, 10, 10, -3, 1, 1, 1, 1, 1, 1, -7]
Hence, the total amount spent = (1 + 1 + 1+1) = 4,


Right side panel:

#
# Complete the 'getMinamount' function beLow.
#
# The function is expected to return an INTEGER.
# The function accepts INTEGER_ARRAY quality as parameter.
#
17
18
19
def getMinAmount (quality):
# Write your code here
20
21
22
23
= main
fptr = open(os. environ[ OUTPUT PATH*], "w*)
24
quality_count - int(input() .strip())
25
26
27
quality - []
28
29
30
for _ in range(quality_count):
quality_item = int(input(). strip())
quality.append(quality_item)
31
32
result - getMinAmount(quality)
33
34
fptr write(str (result) + "In')
35
36
fptr.close()
