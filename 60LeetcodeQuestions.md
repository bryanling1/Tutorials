# 60 Leecode problems to solve for coding interview

[Source](https://medium.com/@koheiarai94/60-leetcode-questions-to-prepare-for-coding-interview-8abbb6af589e)

# Table of contents
- [60 Leecode problems to solve for coding interview](#60-leecode-problems-to-solve-for-coding-interview)
- [Table of contents](#table-of-contents)
- [Linked List Cycle 2](#linked-list-cycle-2)
- [Remove duplicates from sorted list](#remove-duplicates-from-sorted-list)
- [Reverse a linked list](#reverse-a-linked-list)
- [Kth Largest Element in a Stream](#kth-largest-element-in-a-stream)
- [Sub array sum = k](#sub-array-sum--k)
- [Minimum depth of a binary search tree](#minimum-depth-of-a-binary-search-tree)
- [Construct Binary tree for preorder inorder traversal](#construct-binary-tree-for-preorder-inorder-traversal)
- [Longest increasing subsequence](#longest-increasing-subsequence)
- [House Robber 2](#house-robber-2)
- [Best time to buy and sell stock 2](#best-time-to-buy-and-sell-stock-2)
- [Word Break](#word-break)

# Linked List Cycle 2

- Do toroise and the hair to detect if there is a cycle
- Add a pointer to the `start` and `intersect` point
- Whene these 2 pointers intersect its the start of the loop

#  Remove duplicates from sorted list

- 2 Pointers
- Use a dummy starting node to help

```js
var deleteDuplicates = function(head) {
    if(!head || !head.next) return head;
    const dummy = new ListNode(0, head);
    let prev = dummy;
    
    while(head){
        if(head.next && head.val === head.next.val){
            while(head.next && head.val === head.next.val){
                head = head.next
            }
            prev.next = head.next;
        }else{
            prev = prev.next;
        }
        head = head.next
    }
    return dummy.next
   
};
```

# Reverse a linked list

```js
var reverseList = function(head) {
    if(!head) return null;
    if(!head.next){
    return head
  }

  let new_head = reverseList(head.next)

  head.next.next = head

  head.next  = null

  return new_head
};
```

# Kth Largest Element in a Stream

- Use a min heap, restrict length of heap to k

```js
var KthLargest = function(k, nums) {
    this.heap = new PriorityQueue((a, b) => a < b);
    this.k = k;
    for(let num of nums){
        this.add(num);
    }
};

/** 
 * @param {number} val
 * @return {number}
 */
KthLargest.prototype.add = function(val) {
        this.heap.push(val);
        if(this.heap.size() > this.k){
            this.heap.pop();
        } 
        return this.heap.peek();
};
```

# Sub array sum = k

```js
var subarraySum = function(nums, k) {
    let out = 0;
    const temp = {}
    let sum = 0;
    for(let i=0; i<nums.length; i++){
        sum += nums[i];
        if(sum === k ){
            out ++;
        }
        if(sum - k in temp){
            out += temp[sum-k];
        }
        
        if(sum in temp){
            temp[sum] ++
        }else{
            temp[sum] = 1;
        }
        
        
    }
    return out;
};

```

# Minimum depth of a binary search tree

- BFS is on average faster because our solution is higher up the tree

```js
var minDepth = function(root) {
    if(!root) return 0;
    return traverse(root);
};

const traverse = function(root){
    let count = 0;
    const queue = [root];
    while(queue.length > 0){
        let levelCount = queue.length;
        count ++;
        while(levelCount > 0){
            const current = queue.shift();
            if(!current.left && !current.right){
                return count
            }
            if(current.left){
                queue.push(current.left)
            }
            if(current.right){
                queue.push(current.right);
            }
            levelCount --;
        }
        
    }
    return count;
}
```

# Construct Binary tree for preorder inorder traversal

- Difficult part is keeping track of indeces

```js
var buildTree = function(preorder, inorder) {
    return traverse(preorder, inorder, 0, 0, inorder.length -1);
};

const traverse = function(preorder, inorder, preStart, inLeft, inRight){
    if(inLeft > inRight || preStart > preorder.length - 1){
        return null;
    }
    
    const current = new TreeNode(preorder[preStart]);
    let width = 0;
    while(inorder[inLeft + width] !== current.val){
        width ++;
    }
    current.left = traverse(preorder, inorder, preStart + 1, inLeft, inLeft + width - 1);
    current.right = traverse(preorder, inorder, preStart + width + 1, inLeft + width + 1, inRight);
    return current;
}
```

# Longest increasing subsequence 

- Question at each step: What is the largest subsequence at this point that includes this number
```js
var lengthOfLIS = function(nums) {
    const dp= new Array(nums.length).fill(1);
    return helper(nums, dp)
   };
   
   const helper = function(nums, dp){
       for (let j=1; j< nums.length ; j++){
           for(let i=0; i < j; i++){
                if(nums[j] > nums[i] && dp[i] + 1 > dp[j]){
                    dp[j] = dp[i] + 1;
                }
           }
       }
       return Math.max(...dp)
   }
```

# House Robber 2

- only different between `House Robber 1` is our end position for 
```js
var rob = function(nums) {
    if(nums.length === 1) return nums[0];
    return Math.max(helper(nums, 0, nums.length -1), helper(nums, 1, nums.length))
    
};

const helper = function(nums, start, end){
    let prev1_max = 0;
    let prev2_max = 0;
    let current_max = 0;
    for(let i=start; i < end; i++){
        if(nums[i] + prev1_max > current_max){
            current_max = prev1_max + nums[i]
        }
        prev1_max = prev2_max;
        prev2_max = current_max;
    }
    return current_max;
}
```

# Best time to buy and sell stock 2

```js
/**
 * @param {number[]} prices
 * @return {number}
 */
var maxProfit = function(prices) {
    if(prices.length === 1) return 0;
    
    let valley = prices[0];
    let peak = prices[0];
    let out = 0;
    
    for(let i=1; i<prices.length; i++){
        const price = prices[i];
        if(price < peak){
            out += peak - valley;
            valley = price;
            peak = price;
        }else{
            peak = price;
        }
    }
    
    if(peak > valley){
        out += peak - valley;
    }
    
    return out;
    
};
```

# Word Break

<a href="https://ibb.co/1MS5070"><img src="https://i.ibb.co/kJdfG5G/image.png" alt="image" border="0"></a>

```js
 var wordBreak = function(s, wordDict) {
    const dict = {};
     for(let word of wordDict){
         dict[word] = true;
     }
    const dp = new Array(s.length + 1).fill(false);
     dp[0] = true;
    for(let i=0; i<=s.length; i++){
        for(let j=i-1; j >= 0 ; j--){
            if(dp[j] && (s.substring(j, i) in dict)){
                dp[i] = true;
                break;
            }
        }
    }

     return dp[s.length]
    
};

```