# 60 Leecode problems to solve for coding interview

[Source](https://medium.com/@koheiarai94/60-leetcode-questions-to-prepare-for-coding-interview-8abbb6af589e)

# Table of contents
- [60 Leecode problems to solve for coding interview](#60-leecode-problems-to-solve-for-coding-interview)
- [Table of contents](#table-of-contents)
- [Linked List Cycle 2](#linked-list-cycle-2)
- [Remove duplicates from sorted list](#remove-duplicates-from-sorted-list)
- [Reverse a linked list](#reverse-a-linked-list)
- [Kth Largest Element in a Stream](#kth-largest-element-in-a-stream)

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