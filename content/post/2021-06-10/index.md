---
title: "プログラミングトレーニング(06/10/2021)"
date: 2021-06-10T00:00:00+09:00
bigimg: [{src: "/img/path.jpg", desc: ""}]
draft: false
comments: true
tags: ["競技プログラミング"]
---

競技プログラミングの練習メモです。

<!--more-->

## [LeetCode][Easy] 234. Palindrome Linked List
-------

https://leetcode.com/problems/palindrome-linked-list/

Space O(N) だけど、普通に解けた。

```Python
class Solution:
    def isPalindrome(self, head: ListNode) -> bool:
        vals = list()

        while head:
            vals.append(head.val)
            head = head.next

        l, r = 0, len(vals) - 1
        while l <= r:
            if vals[l] != vals[r]:
                return False
            l += 1
            r -= 1

        return True
```

Python だと回文チェックは以下みたいにかけるのか。

```Python
class Solution:
    def isPalindrome(self, head: ListNode) -> bool:
        vals = []
        while head:
            vals.append(head.val)
            head = head.next
        
        return vals == vals[::-1]
```

<br>

## [LeetCode][Easy] 21. Merge Two Sorted Lists
-------

https://leetcode.com/problems/merge-two-sorted-lists/

普通に解けた。

```Python
class Solution:
    def mergeTwoLists(self, l1: ListNode, l2: ListNode) -> ListNode:
        node = ListNode()
        ans = node
        
        while l1 and l2:
            if l1.val < l2.val:
                node.next = l1
                l1 = l1.next
            else:
                node.next = l2
                l2 = l2.next
            node = node.next
        
        node.next = l1 if l2 is None else l2
        
        return ans.next
```


<br>

## [LeetCode][Medium] 2. Add Two Numbers
-------

https://leetcode.com/problems/add-two-numbers/

コーナーケースの最後に carry を忘れないようにさえすれば、簡単。`divmod`が便利なので覚えておく。

```Python
class Solution:
    def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:
        carry = 0
        node = ListNode()
        ans = node
        
        while l1 or l2:
            s = carry
            s += l1.val if l1 else 0
            s += l2.val if l2 else 0
            carry, s = divmod(s, 10)
            node.next = ListNode(s)
            node = node.next
            
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None
        
        if carry:
            node.next = ListNode(1)
        
        return ans.next
```


<br>

## [LeetCode][Medium] 61. Rotate List
-------

そんなに悩まずに解けた。コードは綺麗ではないかも。

```Python
class Solution:
    def rotateRight(self, node: ListNode, k: int) -> ListNode:
        n = 0
        head = node
        tail = node
        while node:
            n += 1
            tail = node
            node = node.next
        
        if n == 0:
            return None
        
        k = k % n
        node = head
        for _ in range(n - k - 1):
            node = node.next

        tail.next = head
        head = node.next
        node.next = None
        
        return head
```

<br>

## [LeetCode][Medium] 708. Insert into a Sorted Circular Linked List
-------

https://leetcode.com/problems/insert-into-a-sorted-circular-linked-list/

普通に解けなかった。。回答見て理解したけど、コーナーケースが難しいだろこれ。

<br>
