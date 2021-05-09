---
title: "Backtracking I"
date: 2021-05-09
draft: false
tags: ['Algorithms']
---

**Backtracking** is a general algorithm for finding all (or some) solutions to some computational problems, I will summarize general approach to backtracking questions with popular examples.

> The backtracking algorithm traverses this search tree  [recursively](https://en.wikipedia.org/wiki/Recursion_(computer_science)) , from the root down, in  [depth-first order](https://en.wikipedia.org/wiki/Depth-first_search) . At each node _c_, the algorithm checks whether _c_ can be completed to a valid solution. If it cannot, the whole sub-tree rooted at _c_ is skipped (_pruned_). Otherwise, the algorithm (1) checks whether _c_ itself is a valid solution, and if so reports it to the user; and (2) recursively enumerates all sub-trees of _c_. The two tests and the children of each node are defined by user-given procedures. Therefore, the _actual search tree_ that is traversed by the algorithm is only a part of the potential tree. **The total cost of the algorithm is the number of nodes of the actual tree times the cost of obtaining and processing each node.** This fact should be considered when choosing the potential search tree and implementing the pruning test.  

Key takeaway is the backtracking is basically DFS.

## Subsets
This is basic question which we can apply backtracking. Here is the LeetCode question.

([Subsets - LeetCode](https://leetcode.com/problems/subsets/) Given an nteger array of **unique** numbers, find all possible unduplicated subsets. For example, with `nums = [1,2,3]`, the unduplicated subsets are `[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]`

![](/images/backtracking-i/9684023D-622F-480D-B0C7-618992397540.jpg)

C++ implementation of the algorithm is the below. The `res` is the integer array to hold the result. The time complexity of the implementation is `N x pow(2, N)` because each element could be present or absent.
```c

vector<int> res;

    void backtrack(int start, vector<int>& cur, vector<int>& nums) {
        res.push_back(cur);
        
        for (int i = start; i < nums.size(); i++) {
            cur.push_back(nums[i]);
            backtrack(i + 1, cur, nums);
            cur.pop_back();
        }
    }
```

## Subsets with duplicates
[Subsets II - LeetCode](https://leetcode.com/problems/subsets-ii/) - Given the integer array that may contain duplicates, find all possible subsets. The result must not contain duplicate subsets.

![](/images/backtracking-i/EA54CA73-4B74-4114-BDAE-2CAE6803B3E6.jpg)

The key idea to remove the duplicates is we skips already visited elements using the condition;
```
start < i && nums[i] == nums[i - 1]
```
If we come across the dilates when `start < i` we already put the element during past DSF. The implementation is like the below; 
```c
vector<int> res;

    void backtrack(int first, vector<int>& cur, vector<int>& nums) {
        res.push_back(cur);
        
        for (int i = start; i < nums.size(); i++) {
			  if (i > start && nums[i] == nums[i - 1]) continue;
            cur.push_back(nums[i]);
            backtrack(i + 1, cur, nums);
            cur.pop_back();
        }
    }
```

## Permutation
[Permutations - LeetCode](https://leetcode.com/problems/permutations/) Given distinct integer array, find the all possible permutation. 

All permutation of  `[1,2,3]` is `[1,2,3], [1,3,2,], [2,1,3], [2,3,1], [3,1,2], [3,2,1]`.This is also variation of backtracking. We need to find the subsets which has the same length of the given integer array. To apply the `backtrack()` we need to sort first. 
```c
vector<int> res;
vector<int> used(nums.size(), 0);

    void backtrack(vector<int>& cur, vector<int>& nums) {
        if (cur.size() == nums.size() {
			  res.push_back(cur);
			  return;
		  }
        
        for (int i = 0; i < nums.size(); i++) {
            if (used[i]) continue;
            used[i]++;
            cur.push_back(nums[i]);
            backtrack(i + 1, cur, nums);
            cur.pop_back();
            used[i]--;
        }
    }
```

## Permutation with duplicates
[Permutations II - LeetCode](https://leetcode.com/problems/permutations-ii/) Given integer arrays which may have duplicates and sorted, find the all possible unique permutations

Given `[1,1,2]` we have unique permutation, `[1,1,2],[1,2,1],[2,1,1]` 

```c
vector<int> res;
vector<int> used(nums.size(), 0);

    void backtrack(vector<int>& cur, vector<int>& nums) {
        if (cur.size() == nums.size() {
			  res.push_back(cur);
			  return;
		  }
        
        for (int i = 0; i < nums.size(); i++) {
            if (used[i] || (!used[i] && nums[i] == nums[i - 1))) continue;
            used[i]++;
            cur.push_back(nums[i]);
            backtrack(i + 1, cur, nums);
            cur.pop_back();
            used[i]--;
        }
    }
```


![](/images/backtracking-i/DBB2053C-F802-41D3-AF74-C5180D28C994.jpg)