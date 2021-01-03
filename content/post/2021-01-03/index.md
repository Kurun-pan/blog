---
title: "プログラミングトレーニング(01/03/2021)"
date: 2021-01-03T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["競技プログラミング"]
---

今日解いたLeetCodeの問題です。

<!--more-->

## (LeetCode) Easy 266. Palindrome Permutation
-------

https://leetcode.com/problems/palindrome-permutation/

文字の順番を自由に入れ替えて回文になるか判定する問題です。文字列中の各文字をカウントし、奇数個数になる文字が1個以下であれば回文に出来るので愚直にハッシュを利用して簡単に解けます。

```
class Solution {
public:
    bool canPermutePalindrome(string s) {
        unordered_map<char, int> mp;
        for (auto c : s)
            mp[c]++;
        
        int count = 0;
        for (auto m :mp) {
            if (m.second % 2 == 1)
                count++;
            if (count > 1)
                break;
        }
        return (count <= 1);
    }
};
```

なお、上記のようにハッシュを使わなくても、C++の場合は`bitset`を利用したビット操作でコンパクトに実装が出来ます。文字の種類はASCIIで256個あるので、256bitのフィールドを用意し、その各ビットに文字を割り当て、該当する文字のビットを反転（NOT）することで、その文字が奇数個あるか偶数個あるかを判別することが出来ます。

```
class Solution {
public:
    bool canPermutePalindrome(string s) {
        bitset<256> b;
        for (char c : s)
            b.flip(c);
        return b.count() < 2;
    }
};
```

<br>

## (LeetCode) Medium 1379. Find a Corresponding Node of a Binary Tree in a Clone of That Tree
-------

https://leetcode.com/problems/find-a-corresponding-node-of-a-binary-tree-in-a-clone-of-that-tree/

これはEasy問題でいい気がしますが、単にDFSかBFSの探索問題です。なんの捻りもないです。

```
class Solution {
public:
    TreeNode* getTargetCopy(TreeNode* original, TreeNode* cloned, TreeNode* target) {
        if (!original)
            return nullptr;
        
        if (original == target)
            return cloned;
        
        auto left = getTargetCopy(original->left, cloned->left, target);
        if (left)
            return left;
        return getTargetCopy(original->right, cloned->right, target);
    }
};
```
