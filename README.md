Code Question 1
As part of your Day 1 Orientation at Amazon, you've been invited to participate in a programming challenge. Please represent your team by completing the following challenge:
Given an array of binary digits, O and 1, sort the array so that all zeros are at one end and all ones are at the other. Which end does not matter. To sort the array, swap any two adjacent elements. Determine the minimum number of swaps to sort the array.
Example arr=[0,1,0,1l
With 1 move, witching elements 1 and 2, yields [0,0,1,l, a sorted array.
Function Description
Complete the function minMoves in the editor below.
minMoves has the following parameters): int arrin): an array of binary digits
Returns
int: the minimum number of moves necessary
Constraints
• 1≤n≤ 105
• arrlil is in the set (O,1)


Input Formatfor Custom Testing
Input from stain will be processed as follows and passed to the function.
The first line contains an integer n, the size of the array arri Each of the next n lines contains a binary digit as an integer, amfil.
7 Sample Case O
Sample Input
SIDIN
Function
8
1
1
1
1
arr[i] size n = 8
arr - ［1, 1,1,1,0, 0,0,0］
Sample Output
Explanation
The array is already sorted, so no moves are necessary.

Sample Case 1
Sample Input
STDIN
----
8
Function
arr[i] size n = 8
→ arr - [1, 1, 1, 1, 0, 1, 0, 1]
Sample Output
3
Explanation
Perform the following minimal sequence of 3 moves 1 1 1 1 0 101→11111001→11 111010→11111100 to sort the array, Bold/red is the value at the new position.


Right side panel:

#!/bin/python3

import math
import os
import random
import re
import sys



#
# Complete the 'minMoves' function below.
#
# The function is expected to return an INTEGER.
# The function accepts INTEGER_ARRAY arr as parameter.
#

def minMoves(arr):
    # Write your code here

if __name__ == '__main__':
    fptr = open(os.environ['OUTPUT_PATH'], 'w')

    arr_count = int(input().strip())

    arr = []

    for _ in range(arr_count):
        arr_item = int(input().strip())
        arr.append(arr_item)

    result = minMoves(arr)

    fptr.write(str(result) + '\n')

    fptr.close()




