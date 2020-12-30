---
title: "プログラミングトレーニング(12/30/20)"
date: 2020-12-30T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["競技プログラミング"]
---

今日解いた問題。

<!--more-->

## (Medium) 1457. Pseudo-Palindromic Paths in a Binary Tree
-------

https://leetcode.com/problems/pseudo-palindromic-paths-in-a-binary-tree/

ノードを任意の順番で入れ替え、回文になるパスの個数を求める問題です。DFSで探索した後で、1から9までの値の個数を数え、奇数個になる値が1以下であればそのパスは入れ替えにより回文になります。

実装した後で気付きましたが、値が1から9までに限定されているので、ハッシュは使わずに9個の配列で実装すれば良かったです…。

```
class Solution {
public:
    int pseudoPalindromicPaths (TreeNode* root) {
        dfs(root);
        return res;
    }

private:
    void dfs(TreeNode* node) {
        mp[node->val]++;

        if (!node->left && !node->right) {
            int odd_cnt = 0;
            for (auto m : mp) {
                if (m.second % 2 == 0)
                    continue;
                odd_cnt++;
                if (odd_cnt > 1)
                    return ;
            }
            res++;
            return ;
        }
        
        if (node->left) {
            dfs(node->left);
            mp[node->left->val]--;
            if (mp[node->left->val] == 0)
                mp.erase(node->left->val);
        }
        if (node->right) {
            dfs(node->right);
            mp[node->right->val]--;
            if (mp[node->right->val] == 0)
                mp.erase(node->right->val);
        }
    }

    unordered_map<int, int> mp;
    int res = 0;
};
```

<br>

## (Medium) 340. Longest Substring with At Most K Distinct Characters
-------

https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/

部分文字列の問題は大体2ポインタで解けますね。この問題の場合は、k種類の文字列以下になる部分文字列の最大長なのでハッシュ（unordered_mapかunordered_set）を使って解きます。

```
class Solution {
public:
    int lengthOfLongestSubstringKDistinct(string s, int k) {
        unordered_map<char, int> mp;
        int l = 0, r = 0;
        int res = 0;
        
        while (r < s.size()) {
            mp[s[r]]++;
            if (mp.size() <= k) {
                res = max(res, r - l + 1);
            } else {
                while (mp.size() > k) {
                    mp[s[l]]--;
                    if (mp[s[l]] <= 0) {
                        mp.erase(s[l]);
                    }
                    l++;
                }
            }
            r++;
        }

        return res;
    }
};
```