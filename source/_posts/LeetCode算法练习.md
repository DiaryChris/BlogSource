---
title: LeetCode算法练习
date: 2019-03-11 15:51:19
tags:
---



#### Two Sum

##### Approach 1: Brute Force

```c++
class Solution 
{
public:
    vector<int> twoSum(vector<int>& nums, int target) 
    {
        vector<int> result;
        for(int i = 0 ; i < nums.size() ; i++)
        {
            for (int j = i + 1 ; j < nums.size() ; j++)
            {
                if(nums[i] + nums[j] == target)
                {
                    result = {i, j};
                }
            }
        } 
        return result;
        throw "No solution";
    }
};
```


$$
T(n)=O(n^2)
\\
S(n)=O(1)
$$


##### Approach 2: One-pass Hash Table

```c++
class Solution 
{
public:
    vector<int> twoSum(vector<int>& nums, int target) 
    {
        vector<int> result;
        map<int, int> hash;
        for(int i = 0; i < nums.size(); i++)
        {
            int complement = target - nums[i];
            if(hash.count(complement))
            {
                result = {hash[complement], i};
                return result;
            }
            hash[nums[i]] = i;
        }
        throw "No solution";
    }
};
```


$$
T(n)=O(n) \\ S(n)=O(n)
$$



*Runtime: 12 ms, faster than 94.28% of C++ online submissions for Two Sum.*

*Memory Usage: 10.4 MB, less than 32.76% of C++ online submissions for Two Sum.*



#### Add Two Numbers

##### Approach 1: Elementary Math

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution 
{
    public:
        ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) 
        {
            ListNode* result = new ListNode(0);
            ListNode* head = result;
            
            while(l1 || l2)
            {
                if(l1)
                {
                    result->val += l1->val;
                    l1 = l1->next;
                }
                if(l2)
                {
                    result->val += l2->val;
                    l2 = l2->next;
                }
                if(result->val > 9)
                {
                    result->val = result->val % 10;
                    result->next = new ListNode(1);
                }
                else
                {
                    if(l1 || l2)
                    {
                        result->next = new ListNode(0);
                    }
                }
                result = result->next;
            }
            return head;
        }
};
```


$$
T(n)=O(max(m, n)) \\ S(n)=O(max(m, n))
$$



*Runtime: 40 ms, faster than 96.35% of C++ online submissions for Add Two Numbers.*

*Memory Usage: 19.1 MB, less than 61.79% of C++ online submissions for Add Two Numbers.*



#### Longest Substring Without Repeating Characters



##### Approach 1: Sliding Window

```c++
class Solution 
{
public:
    int lengthOfLongestSubstring(string s) 
    {
        int maxLength = 0;
        int lastStart = 0;
        map<char, int> charIndex;
        for(int i = 0; i < s.size(); i++)
        {
            if(charIndex.count(s[i]) && charIndex[s[i]] >= lastStart)
            {
                lastStart = charIndex[s[i]] + 1;
            }
            
            if(maxLength < i - lastStart + 1)
            {
                maxLength = i - lastStart + 1;
            }
            charIndex[s[i]] = i;
        }
        return maxLength;
    }
};
```

$$
T(n)=O(n) \\ S(n)=O(min(m, n))
$$



*Runtime: 36 ms, faster than 49.65% of C++ online submissions for Longest Substring Without Repeating Characters.*

*Memory Usage: 16.1 MB, less than 60.63% of C++ online submissions for Longest Substring Without Repeating Characters.*



##### Approach 2: Brute Force



```c++
class Solution 
{
public:
    int lengthOfLongestSubstring(string s) 
    {
        int maxLength = 0;
        set<char> substring;
        for(int i = 0; i < s.size(); i++)
        {
            for(int j = i; j < s.size() && !substring.count(s[j]); j++)
            {
                substring.insert(s[j]);
            }
            if(maxLength < substring.size())
            {
                maxLength = substring.size();
            }
            substring.clear();
        }
        return maxLength;
    }
};
```


$$
T(n)=O(n^2) \\ S(n)=O(min(m, n))
$$


*Runtime: 1148 ms, faster than 5.14% of C++ online submissions for Longest Substring Without Repeating Characters.*

*Memory Usage: 272 MB, less than 5.03% of C++ online submissions for Longest Substring Without Repeating Characters.*

