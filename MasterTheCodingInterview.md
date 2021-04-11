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
- [Sorting](#sorting)
  - [Bubble Sort](#bubble-sort)
  - [Selection Sort](#selection-sort)
  - [Insertion Sort](#insertion-sort)
  - [Merge Sort](#merge-sort)
  - [Quick Sort](#quick-sort)
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

