# Master the Coding Interview: Course

# Table of contents
- [Master the Coding Interview: Course](#master-the-coding-interview-course)
- [Table of contents](#table-of-contents)
- [Palindromes](#palindromes)
- [Linked Lists](#linked-lists)
  - [Reversing a linked list](#reversing-a-linked-list)
  - [M to N reversal](#m-to-n-reversal)
  - [Flattening doubly linked list with children](#flattening-doubly-linked-list-with-children)
  - [Cycle Detection](#cycle-detection)
        - [Floyd's Tortoise and Hare Algorithm](#floyds-tortoise-and-hare-algorithm)
- [Stacks and Queues](#stacks-and-queues)
  - [Minimum Brackets to Remove](#minimum-brackets-to-remove)
  - [Implement a Queue class using stacks](#implement-a-queue-class-using-stacks)
- [Recursion](#recursion)
  - [Divide and Conquer](#divide-and-conquer)
  - [Kth Largest Element](#kth-largest-element)
    - [Hoare's Quickselect Algorithm](#hoares-quickselect-algorithm)
  - [Binary Search](#binary-search)
    - [Start And End of Target in a Sorted Array](#start-and-end-of-target-in-a-sorted-array)
- [Sorting](#sorting)
  - [Bubble Sort](#bubble-sort)
  - [Selection Sort](#selection-sort)
  - [Insertion Sort](#insertion-sort)
  - [Merge Sort](#merge-sort)
  - [Quick Sort](#quick-sort)
- [Trees](#trees)
  - [Binary Tree](#binary-tree)
  - [Binary Search Tree](#binary-search-tree)
  - [PreOrder, InOrder, PostOrder](#preorder-inorder-postorder)
  - [Searching every elment O(n)](#searching-every-elment-on)
    - [Breadth First Search (BFS)](#breadth-first-search-bfs)
    - [Depth First Search (DFS)](#depth-first-search-dfs)
  - [Binary Tree Approach](#binary-tree-approach)
  - [Maximum Depth of Binary Tree](#maximum-depth-of-binary-tree)
  - [Level Order Of Binary Tree](#level-order-of-binary-tree)
  - [Right Side View Tree](#right-side-view-tree)
  - [Number of Nodes in Complete Tree](#number-of-nodes-in-complete-tree)
    - [Complete Tree](#complete-tree)
    - [Full tree](#full-tree)
    - [Idea](#idea)
  - [Validate Binary Search Tree](#validate-binary-search-tree)
  - [Contraints](#contraints)
    - [Idea](#idea-1)
- [Heaps](#heaps)
  - [Heap insertion](#heap-insertion)
  - [Heap Deletion](#heap-deletion)
  - [Implementation](#implementation)
- [2D arrays](#2d-arrays)
  - [Number of Islands](#number-of-islands)
    - [Constraints](#constraints)
    - [Idea](#idea-2)
  - [Space and Time complexity](#space-and-time-complexity)
    - [Time complexity](#time-complexity)
- [General tips to solving problems](#general-tips-to-solving-problems)
# Palindromes

- Often appear as string sub problems
- 3 Methods:
  - 2 pointers starting from ends
  - 2 pointers starting from middle
  - Left to right pointers on 2 strings

# Linked Lists

- Reasigning values always envolves some sort of temp variable
- Make sure that switching values you check for edge cases where the value is `null`
- 99% of the time the **iterative** of approach has a better space and time complexity of the **recursive** one
## Reversing a linked list
```js
const reverseList = function(head) {
    let currentNode = head
    let prev = null

    while(currentNode){
        let temp = currentNode.next
        currentNode.next = prev
        prev = currentNode
        currentNode = temp
    }
    return prev
};
```
Recursive:
```js
const reverseList = function(head){
  if(!head.next){
    return head
  }

  let new_head = reverseList(head.next)

  head.next.next = head

  head.next  = null

  return new_head
}
```
## M to N reversal

Given a linked list and numbers m and n, return it back with only positions m to n in reverse.

**Main Idea**
- We want to keep track of:
  - `m-1`: start
  - `m`: start of reversal
  - `n`: end of reversal
  - `n+1`: tail
  - `positions`: Current position

To get to starting position:
```js
let currentPos = 1, currentNode = head, start = head;

while(currentPos < m){
    start = currentNode;
    currentNode = currentNode.next;
    currentPos ++ 
}
```

Reverse:
```js
let newList = null, tail = currentNode;
while(currentPos >= m && currentPos <=n){
    const next = currentNode.next;
    currentNode.next = newList;
    newList = currentNode;
    currentNode = next;
    currentPos ++;
}
```
Apply to original list
```js
start.next = newList;
tail.next = currentNode;
```

Return the conditional head
```js
if(m>1) return head;
return newList;
```

## Flattening doubly linked list with children

<a href="https://ibb.co/QKTs4tR"><img src="https://i.ibb.co/qrcbvVf/image.png" alt="image" border="0"></a>

- Recursive has O(n) space complexity

- Merging bottom is the same as merging from top, so iterably is the best approach

- To merge, we need the **head**, **tail** (head.next), the **childHead** (head.child), and the **childTail**

```js
const flatten = function(head){
//check for if head is null
if(!head) return head;

let currentNode = head;

//iterate through
while (currentNode !== null){
  //no child? advance
  if(currentNode.child === null){
    currentNode = currentNode.next;
  }else{
    //we found a child, we want to start merging
    //we need to find the **childTail**
    let tail = currentNode.child;
    while(tail.next !== null){
      tail = tail.next
    }
    //we have our 4 values, now we can set them
    tail.next = currentNode.next
    if(tail.next !== null){
      tail.next.prev = tail;
    }
    currentNode.next = currentNode.child;
    currentNode.next.prev = currentNode;
    currentNode.child = null;
  }
}
}
```

## Cycle Detection 

<a href="https://ibb.co/7CNPwD4"><img src="https://i.ibb.co/Rb4fxKy/image.png" alt="image" border="0"></a>

##### Floyd's Tortoise and Hare Algorithm
- 2 pointers (tortoise and hare)
- **tortoise** moves at 1 step per iteration
- **hair** moves at 2 steps per iteration
- You have detected a cycle when the 2 pointers end up at the same node
- If **hair** is null  or **hair.next** is null, you do not have a cycle (found the tail)

<a href="https://ibb.co/4t9yWgw"><img src="https://i.ibb.co/c3M5JXK/image.png" alt="image" border="0"></a>
# Stacks and Queues
## Minimum Brackets to Remove

```ts
 var minRemoveToMakeValid = function(s) {
    const stack = [];
    const res = s.split('');
    for(let i=0; i<s.length; i++){
        const char = s.charAt(i);
        if(char !== "(" && char !== ")" ){
            continue;
        }

        if(char === "("){
            stack.push(i)
        }else{
            //")"
            if(stack.length > 0){
                stack.pop()
            }else{
                res[i] = '';
            }
        }
    }

    for(let i=0; i<stack.length; i++){
        res[stack[i]] = '';
    }

    return res.join('');

};
```
- set `.split()` elements to `''` will remove them when we `.join()`

## Implement a Queue class using stacks

<a href="https://ibb.co/GPDbMPC"><img src="https://i.ibb.co/h8Ntd82/image.png" alt="image" border="0"></a>
 - Continue popping out of `stack2` until it is empty, push `stack` into `stack2` and continue to pop

```js
class QueueWithStack{
  constructor(){
    this.in=[];
    this.out=[];
  }

  enqueue(val){
    this.in.push(val)
  }

  dequeue(){
    if(this.out.length === 0){
      while(this.in.length){
        this.out.push(this.in.pop())
      }
    }
    return this.out.pop()
  }

  peek(){
    if(this.out.length === 0){
      while(this.in.length){
        this.out.push(this.in.pop())
      }
    }
    return this.out[this.out.length - 1]
  }

  empty(){
    if(this.out.length === 0 && this.int.length === 0) return true
    return false
  }
}
```

# Recursion
- Normal recursion space: O(N)
- Tail recursion space: O(1)

<a href="https://ibb.co/Xj6dTyP"><img src="https://i.ibb.co/S5H1pQg/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/svcvxJ8"><img src="https://i.ibb.co/mCdCZFf/image.png" alt="image" border="0"></a>

## Divide and Conquer

1. Multi-branched recursion
2. Breaks a problem into multipel smaller but same sub-problems
3. Combines the solutions of sub-problems into the solution for the original problem (putting them all together)


## Kth Largest Element

Given an unsorted array, return the kth largest element. It is the kth largest element in sorted order, not the kth distinct element.

### Hoare's Quickselect Algorithm

Quicksort, but we only sort the side in which the target index is greater or larger than

```js

var findKthLargest = function(nums, k) {
    const indexToFind = nums.length - k;
    quickSelect(nums, 0, nums.length -1, indexToFind);
    return nums[indexToFind]
};

const quickSelect = function(array, left, right, indexToFind){
    if(left < right){
        const pivot = innerSort(array, left, right);
        if(pivot === indexToFind){
            return array[pivot]
        }else if(pivot < indexToFind){
            quickSelect(array, pivot + 1, right, indexToFind)
        }else{
            quickSelect(array, left, pivot - 1, indexToFind)
        }
    }
}

const innerSort = function(nums, start, end){
    let i = start;
    let j = start;

    while(j < end){
        if(nums[j] < nums[end]){
            let temp = nums[j];
            nums[j] = nums[i];
            nums[i] = temp;
            i++;
            j++;
        }else{
            j++;
        }
    }

    const temp = nums[end]
    nums[end] = nums[i]
    nums[i] = temp

    return i;
}
```
- Average time complexity is `O(n)` (n + n/2 + n/4 ... = 2n)
- Worst case is `(O(n^2))` when the array is in reverse sorted order

## Binary Search

```js
const binarySearch = function(array, target){
  let left = 0;
  let right = array.length;

  while(left <= right){
    const mid = Math.floor((left + right)/2);
    const foundVal = array[mid];
    if(foundVal === target){
      return mid;
    }else if(foundVal < target){
      left = mid + 1;
    }else{
      right = mid - 1;
    }
  }

  return -1;
}
```
### Start And End of Target in a Sorted Array

Given an array of interegers **sorted** in ascending order, return the starting and ending index of a given target value in an array, i.e [x, y]. 

Your solutin should run in logN time.

We see that the given array is **sorted** so we should expect some sort of use of **Binary Search**

```
[1, 3, 3, 5, 5, 5, 8, 9] t=5 --> [3, 5]

[1, 2, 3, 4, 5, 6] t=4 --> [3, 3]

[1, 2, 3, 4, 5] t=9 --> [-1, -1]

[] t=9 --> [-1, -1]
```

**Idea**
We want to find the middle point of the **block** of our target value with binary search. 

Then, partition the left and right sides, and do another binary search to find where it ends.

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var searchRange = function(nums, target) {
    //Find the middle of the block
    const mid = binSearch(nums, 0, nums.length - 1, target);
    if(mid === -1){
        return [-1, -1];
    }
    //Find the left side
    let start = mid;
    let end = mid;
    let temp1; temp2;

    while(start !== -1){
      temp1 = start;
      start = binSearch(nums, 0, start - 1, target);
    }

    start = temp1;

    while(end !== -1){
      temp2 = end;
      end = binSearch(nums, end + 1, nums.length - 1, target);
    }

    end = temp2;

    return [start, end]
};

const binSearch = function(array, left, right, target){
    let left = 0;
    let right = array.length - 1;

    while(left <= right){
        const mid = Math.floor((left + right)/2);
        const val = array[mid];
        if(val === target){
            return mid
        }else if(val < target){
            left = mid + 1;
        }else{
            right = mid - 1;
        }
    }
    return -1;
}
```


# Sorting

<a href="https://ibb.co/WDncfG0"><img src="https://i.ibb.co/XyXztjD/image.png" alt="image" border="0"></a>

## Bubble Sort

Swapping pairs 

<a href="https://ibb.co/KsbdQGb"><img src="https://i.ibb.co/xgXWcsX/image.png" alt="image" border="0"></a>

## Selection Sort

Find the smallest element, place to next in the list

## Insertion Sort

<a href="https://ibb.co/ccbxZgZ"><img src="https://i.ibb.co/xSm3BCB/image.png" alt="image" border="0"></a>

Good for small data and when the list is almost sorted

## Merge Sort

<a href="https://ibb.co/gtRcqpX"><img src="https://i.ibb.co/BTCYWx7/image.png" alt="image" border="0"></a>

```js
const innerSort = (arr1, arr2) => {
    let p1=0;
    let p2=0;
    let out = [];

    while(p1 < arr1.length && p2 < arr2.length){
        if(arr1[p1] < arr2[p2]){
            out.push(arr1[p1]);
            p1 ++;
        }else{
            out.push(arr2[p2]);
            p2 ++;
        }
    }

    if(p1 < arr1.length){
        out = out.concat(arr1.slice(p1))
    }

    if(p2 < arr2.length){
        out = out.concat(arr2.slice(p2))
    }

    return out;
}

const mergeSort = (arr) =>{
    if(arr.length < 2){
        return arr
    }

    const mid = Math.floor(arr.length / 2);
    const arr1 = mergeSort(arr.slice(0, mid));
    const arr2 = mergeSort(arr.slice(mid));

    return innerSort(arr1, arr2);
}
module.exports = {
    innerSort,
    mergeSort,
};

```

## Quick Sort

Pick a pivot point

<a href="https://ibb.co/1724c6x"><img src="https://i.ibb.co/pbXkqyp/image.png" alt="image" border="0"></a>

```js
const quickSort = function(array, left=0, right=array.length - 1){
    if(left < right){
        const pivot = innerSort(array, left, right);
        quickSort(array, pivot + 1, right)
        quickSort(array, left, pivot - 1)
    }
}

const innerSort = function(nums, start, end){
    let i = start;
    let j = start;

    while(j < end){
        if(nums[j] < nums[end]){
            let temp = nums[j];
            nums[j] = nums[i];
            nums[i] = temp;
            i++;
            j++;
        }else{
            j++;
        }
    }

    const temp = nums[end]
    nums[end] = nums[i]
    nums[i] = temp

    return i;
}
```

# Trees

## Binary Tree

Each parent either has 0, 1, 2, children

## Binary Search Tree

- Left child is smaller than the current node

- Right child is greather than the current node

- Unbalanced tress have a O(n) in worst cases, turns into a linked list

- No O(1) operationsa

## PreOrder, InOrder, PostOrder

**InOrder:** It returns the values in a BST in order
**PreOrder:** Parent, left-child, right-child
  - Useful for recreating a tree from a PreOrdered array
**PostOrder:** left-child, right-child, parent

## Searching every elment O(n)
### Breadth First Search (BFS)

- Top to bottom, left to right
- Shortest Path
- Closer Nodes
- High memory requirement because we have to store the list of children nodes
  - Use a queue for this

**Use Cases**
- Answer not far from the roof the tree
- If the tree is very deep and solutions are rare;
  - Space complexity of O(Width of tree)
- Finding the shortest path

```js
class BinarySearchTree{
  constructor(){
    this.root = null;
  }

  insert(value){...}

  lookup(value){...}

  remove(value){...}

  breadthFirstSearch(){
    let currentNode = this.root;
    let list = [];
    let queue = [];
    queue.push(currentNode);

    while(queue.length > 0){
      currentNode = queue.shift();
      list.push(currentNode.value);
      if (currentNode.left){
        queue.push(currentNode.left)
      }
      if (currentNode.right){
        queue.push(currentNode.right);
      }
    }
    return list;
  }
}
```

### Depth First Search (DFS)

- Visit every child then go back up
- Lower memory requirement
- Good at answering "Does Path Exist?"
- Can be slow

**Use Cases**
- If the tree is very wide (BFS would use too much memo)
- If solutions are frequent but located deep in the tree
  - Space complexity of O(Height of tree)
- Determining if a path exists between two nodes


## Binary Tree Approach

1. Do I need to traverse the tree (98% of the time)
2. How? (BFS or DFS)
3. If DFS, which traversal type if we care about the values
   
## Maximum Depth of Binary Tree

1. Furthest leaf node, so we know its DFS
  
```js
function recurise(node, count=0){
  if(!node){
    return count;
  }

  count++;

  return Math.max(recurse(node.left, count), recurse(node.right, count));
}
``` 

## Level Order Of Binary Tree

Essentially implement BFS but each level has its own array.

**Idea**
- identify tree level by adding a counter
- Count the number of elements in the next level by looking at the children
```js
/**
 * Definition for a binary tree node.
 * function TreeNode(val, left, right) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.left = (left===undefined ? null : left)
 *     this.right = (right===undefined ? null : right)
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number[][]}
 */
var levelOrder = function(root) {
    if(!root){
        return [];
    }
    
    let currentNode = root;
    let list = [];
    let queue = [];
    
    
    queue.push(currentNode);

    while(queue.length > 0){
        let elementsInLevel = queue.length;
        let count = 0;
        const currentLevelItems = [];
        while(count < elementsInLevel){
            currentNode = queue.shift();
            currentLevelItems.push(currentNode.val)
            count ++;
            if (currentNode.left){
              queue.push(currentNode.left)
              }
            if (currentNode.right){
              queue.push(currentNode.right);
            }
        }
        list.push(currentLevelItems)
    }
    return list;
};
```

## Right Side View Tree

Given a binary tree, imagine you're standing to the right of the tree. Retrun an array of the values of the nodes you can soo ordered from top to bottom.

We can use the similar solution from the last question using BFS.

**DFS Idea**
- Prioritize finding the right most values
- We need to someone keep track of the level of our nodes
- We can use

```js
var rightSideView = function(root) {
    const out = []
    dfs(root, 0, out)
    return out;
};


const dfs = function(root, level, result){
    if(!root){
        return
    }
    if (level >= result.length){
        result.push(root.val);
    }
    if(root.right){
        dfs(root.right, level+1, result)
    }
    if(root.left){
        dfs(root.left, level+1, result)
    }
}
```

## Number of Nodes in Complete Tree

Given a complete binary tress, count the number of nodes.
<a href="https://ibb.co/HTDqWjM"><img src="https://i.ibb.co/pjvdtSs/image.png" alt="image" border="0"></a>

We can obviously do DFS, but since it is a complete binary tree we can optimize
### Complete Tree
- Every level is completely full, except for the last level which all extres nodes need to be pushed to the left as far as they can be.

### Full tree
- Every node has either 2 or 0 children

### Idea
- Utilise the fact that the tree is complete
- Divide the question into 2 problems: 
  1. Get to the last level
     - We can observe that the number of nodes at the nth level is 2^(n-1) - 1
     - We can get the height of three by traversing down the left side all the way (since the tree is complete)
  2. Count the number of nodes in the last level

<a href="https://imgbb.com/"><img src="https://i.ibb.co/ns1FNSK/image.png" alt="image" border="0"></a>
     - Binary search the last level
     - We know that the **minimum** is 1, **maximum** is 2^(n-1)
     - If we use indecies to label the nodes, If we find the last node in the last level (i) the total number of nodes in that level is i + 1
     - We `ceil` the mid value instead of `floor` because our midpoint is **inclusive** (Looking for a position)
     - We resetting the left and right pointers, set `left` to `mid` or `right` to `mid - 1`
       - Since we know that if we set `right` pointer, everying to the right of it **including itself** isn't a node
      - Stop when the `left` and `right` pointer are equal 
    - To traverse to the node we are searching for, we can use an approach similar to **binary search**
     - Calculate the `mid` value and round up
     - If the value of `mid` is less than our equal to `search index`, we go to the right. Then reposition the pointers and repeat
   
```js
/**
 * Definition for a binary tree node.
 * function TreeNode(val, left, right) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.left = (left===undefined ? null : left)
 *     this.right = (right===undefined ? null : right)
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number}
 */
const countNodes = function(root){
  if(!root){
    return 0;
  }
  //First get the number of levels
  let levels = 0;
  let temp = root;
  while(temp){
    levels ++;
    temp = temp.left;
  }

  //Set are left and right pointers
  const lengthOfLast = 2**(levels - 1);
  let left = 0;
  let right = lengthOfLast - 1;

  while(left < right){
    const mid = Math.ceil((right + left) / 2);
    const result = ithNodeInLastLevel(root, mid, lengthOfLast);

    if(result){
      left = mid;
    }else{
      right = mid - 1;
    }
  }

  return left + lengthOfLast;

}

const ithNodeInLastLevel = function(root, i, length){
    let temp = root;
  let left = 0;
  let right = length - 1;

  while(left < right){
    const mid = Math.ceil((right + left) / 2);
    if(mid <= i){
      //go right
      left = mid;
        temp = temp.right;
        
    }else{
      //go left
      right = mid - 1;
      temp = temp.left;
    }
  }

  return temp ? true : false;
}
```


## Validate Binary Search Tree

Given a binary tree, determine if it is a valid binary search tree

<a href="https://ibb.co/WkMbRf8"><img src="https://i.ibb.co/rxzCXH8/image.png" alt="image" border="0"></a>

## Contraints

- Are there duplicate values in the tree? If so, how do we handle them?
  
<a href="https://ibb.co/CMgLxmt"><img src="https://i.ibb.co/8dqkFNB/image.png" alt="image" border="0"></a>

### Idea
- Update the boundaries when we traverse down the tree
- Go `right` update `min`, go `left` update `max` with current value;

```js
const isValidBST = function(root){
  if(!root) return true;

  return dfs(root, -Infinity, Infinity)
}

const dfs = function(root, min, max){
  if(node.val <= min || node.val >= max){
    return false;
  }

  if(node.left){
    if(!dfs(node.left, min, node.val)){
      return false;
    }
  }

  if(node.right){
    if(!dfs(node.right, node.val, max)){
      return false;
    }
  }

  return true;
}

```
- We can't just `return dfs(node.left, min, node.val)` when we check the children because we need to make sure we check both the left and right children.

# Heaps

- Resembles a **complete BTS**
- Max Heap and Min Heap
- Root node has the Max/Min value
- We can represent with an array in BFS form
- **Parent:** `floor((index - 1)/2)`
- **Left:** `2*index + 1`
- **Right:** `2*index + 2`
- Priority Queue is essentailly a heap
  
## Heap insertion

1. Insert element at end of the array
2. keep swapping up until into the right place
  

## Heap Deletion
1. Remove the top value
2. Take the last element in the array and move to the top then perk down
3. If its a `max heap`, with swap with the value that is **greater** than the top. If they are both greater, go with the one with the **larger** value as to maintain the heep structure

## Implementation

```js
class Priority Queue(
  constructor( comparator = (a,b) => a > b){
    this._heap = [];
    this._comparator = comparator;
  }

  size(){
    return this._heap.length;
  }

  isEmpty(){
    return this.size() === 0;
  }

  peek(){
    return this._heap[0];
  }

  _parent(index){
    return Math.floor((index - 1)/2);
  }

  _leftChild(index){
    return index * 2 + 1;
  }

  _rightChild(index){
    return index*2 + 2;
  }

  _swap(index1, index2){
    const temp = this._heap[index1];
    this._heap[index1] = this._heap[index2];
    this._heap[index2] = temp;
  }

  _compare(i,j){
    return this._comparator(this._heap[i], this._heap[j])
  }

  push(value){
    this._heap.push(value);
    this._siftUp();
    return this.size();
  }

  _siftUp(){
    let nodeIndex = this._size() - 1;
    while(nodeIndex > 0 && this._compare(nodeIndex, this_parent(nodeIndex))){
      this._swap(nodeIndex, this._parent(nodeIndex));
      nodeIndex = this._parent(nodeIndex);
    }
  }

  pop(){
    if(this.size() > 1){
      this._swap(0, this.size() - 1);
    }
    const poppedValue = this._heap.pop();
    this._siftDown();
    return poppedValue;
  }

  _siftDown(){
    let nodeIndex = 0;
    while(
      (this._leftChild(nodeIndex) < this.size() &&
      this._compare(this._leftChild(nodeIndex), nodeIndex))) ||
      (this._rightChild(nodeIndex) < this.size() &&
      this._compare(this._rightChild(nodeIndex), nodeIndex))) 
    ){
      const greaterNodeIndex = this._rightChild(nodeIndex) < this.size() && this.compare(this._rightChild(nodeIndex), this._leftChild(nodeIndex)) ? 
                            this._rightChild(nodeIndex) : this.leftChild(nodeIndex);
      this._swap(greaterNodeIndex, nodeIndex);
         nodeIndex = greaterNodeIndex;
  }

)
```
- `_` means `private`

# 2D arrays

## Number of Islands

Given a @D array containing only 1's (land) and 0's(water), count the number of islands.
Islands are attacked horizontally and vertically

<a href="https://ibb.co/JQMSgKj"><img src="https://i.ibb.co/7460LXt/image.png" alt="image" border="0"></a>

### Constraints

Are the outsides water? Yes

### Idea
- Have a counter, every time we hit a 1, if it is a new island, increment the counter
- We can start by searching **sequantly** until we hit a 1
- Then traverse that 1 until all the corners are 0
- We can then **switch all the values to 0** of that island and search again (to prevent reading over the same values)
- We can then traverse with BFS and DFS in up, right, down, left directions

Using BFS:
```js
const directions = [
  [-1, 0],
  [0, 1],
  [1, 0],
  [0, -1]
]

const numberOfIslands = function(matrix){
  if(matrix.length === 0) return 0;

  let islandCount = 0;

  for(let row=0; row<matrix.length; row++){
    for(let col=0; col<matrix[0].length; col++){
      if(matrix[row][col] === 1){
        //Found a island
        islandCount ++;
        matrix[row][col] = 0;
        const queue = [];
        queue.push([row, col]);
        
        while(queue.length){
          const currentPost = queue.shift();
          const currentRow = currentPos[0];
          const currentCol = currentPos[1];
          
          for(let i=0; i<directions.length; i++){
            const currentDir = directions[i];
            const nextRow = currentRow + currendDir[0]
            const nextCol = currentCol + currentDir[1]
            //make sure the new coordinates are real in our matrix
            if( nextRow < 0 || nextRow >= matrix.length || nextCol < 0 || nextCol >= matrix[0].length){
              continue;
            }

            if(matrix[nextRow][nextCol] === 1){
              queue.push([nextRow, nextCol]);
              //flip the value
              matrix[nextRow][nextCol] = 0;
            }
          }
        }
      }
    }
  }

  return islandCount;
}
```

## Space and Time complexity

### Time complexity
- BFS
  - Sequential part is O(n)
  - Search is also O(n)
  - Therefore O(n + n) == O(2n), since our inner BFS switches everything to 0, so there is no overlap
  - Space complexity is related to the queue
    - In worst case, the grid is filled with 1s
    - Then our space complexity is O(max(rows, columns)) as diagnoal ( have not visited those directions yet so they will be in queue);
- DFS
  - Time: O(n)
  - Space: Since we traverse to the bottom, in our worst case (when the entire matrix is all 1s) our stack spack os O( rows * columns) which yeilds a **worse space complexity* than BFS
# General tips to solving problems

Step
1. Ask questions about constraints to the problem
2. Write out a best case, worst case, and edge cases


- Write down absolutelty everything that we know during a small step
  - Arrays
    - Length of the array
    - If certain values only appear to the left or right
    - If the question is in **sorted order** we probably need **binary search**