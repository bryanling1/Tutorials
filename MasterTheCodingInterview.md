# Master the Coding Interview: Course

# Table of contents
- [Palindromes](#palindromes)
# Palindromes

- Often appear as string sub problems
- 3 Methods:
  - 2 pointers starting from ends
  - 2 pointers starting from middle
  - Left to right pointers on 2 strings

# Linked Lists
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