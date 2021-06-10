---
title: "プログラミングトレーニング(06/09/2021)"
date: 2021-06-09T00:00:00+09:00
bigimg: [{src: "/img/path.jpg", desc: ""}]
draft: false
comments: true
tags: ["競技プログラミング"]
---

競技プログラミングの練習メモです。

<!--more-->

## [LeetCode][Easy] 160. Intersection of Two Linked Lists
-------
https://leetcode.com/problems/intersection-of-two-linked-lists/

ハッシュを利用して time/space O(N + M) で解く方法で簡単に解けた。
```Python
class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> ListNode:
        hash_table = set()
        while headA:
            hash_table.add(headA)
            headA = headA.next
        
        while headB:
            if headB in hash_table:
                return headB
            headB = headB.next
        
        return None
```

Space O(1) で解く方法は全然思い付かなくて、回答をチラ見してあーなるほどなと理解した。これその場で思い付けるものだろうか。
```Python
class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> ListNode:
        nodeA = headA
        nodeB = headB
        while nodeA != nodeB:
            nodeA = headB if nodeA is None else nodeA.next
            nodeB = headA if nodeB is None else nodeB.next

        return nodeA
```

<br>

## [LeetCode][Medium] 19. Remove Nth Node From End of List
-------

2パスでやる方法は思い付いて解けた。ちょっと綺麗ではないけど。

```Python
class Solution:
    def removeNthFromEnd(self, head: ListNode, n: int) -> ListNode:
        node = head
        list_len = 0
        while node:
            node = node.next
            list_len += 1
        
        prev = None
        node = head
        for _ in range(list_len - n):
            prev = node
            node = node.next
        
        if prev == None:
            return node.next

        prev.next = node.next
        
        return head
```

```Python
```

<br>

## [LeetCode][Easy] 206. Reverse Linked List
-------

https://leetcode.com/problems/reverse-linked-list/

久しぶりに解いたあれって解けなかった。。。要復習だなこれ。

```Python
class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
        if head is None or head.next is None:
            return head

        node = self.reverseList(head.next)
        head.next.next = head
        head.next = None
        return node
```

<br>

## [LeetCode][Easy] 203. Remove Linked List Elements
-------

https://leetcode.com/problems/remove-linked-list-elements/

再帰コールで解けたけど、Space O(1) の while ループで解く方法が出来なくてこれもまた要復習。。

```Python
class Solution:
    def removeElements(self, node: ListNode, val: int) -> ListNode:
        if node is None:
            return None
        
        if node.val == val:
            return self.removeElements(node.next, val)
        
        node.next = self.removeElements(node.next, val)
        return node
```

```Python
```

<br>

## [LeetCode][Medium] 328. Odd Even Linked List
-------

https://leetcode.com/problems/odd-even-linked-list/

解法の考え方は正しかったけど、解けなかった。。

```Python
class Solution:
    def oddEvenList(self, node: ListNode) -> ListNode:
        if node is None or node.next is None:
            return node
        
        head = node

        odd = head
        even = head.next 
        even_head = even
        
        while even and even.next:
            odd.next = even.next
            odd = odd.next
            
            even.next = odd.next
            even = even.next
        
        odd.next = even_head
        
        return head
```

<br>
