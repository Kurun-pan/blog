---
title: "プログラミングトレーニング(12/28/2020)"
date: 2020-12-28T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["競技プログラミング"]
---

今日解いた問題。

<!--more-->

## (Medium) 754. Reach a Number
-------

https://leetcode.com/problems/reach-a-number/

普通にアルゴリズムを思い付かなくて解けなかったです。Solution見て理解しましたが、AtCoderっぽい数学が少し絡んだ問題でした。

```
class Solution {
public:
    int reachNumber(int target) {
        target = abs(target);
        int k = 0;
        while (target > 0)
            target -= ++k;

        return target % 2 == 0 ? k : k + 1 + k % 2;

        /* 上記returnを分かり易く愚直に分解すると以下
        if (target % 2 == 0)
            return k;
        
        if ((target - (k + 1)) % 2 == 0)
            return k + 1;
        
        return k + 2;
        */
    }
};
```

<br>

## (Medium) 1602. Find Nearest Right Node in Binary Tree
-------

https://leetcode.com/problems/find-nearest-right-node-in-binary-tree/

キューを使ったBFSのアプローチで解けました。同じ深さのレベルという点に注意して、同じ深さのレベルの中でターゲットの値のノードが来たら、その一つ前のノードをリターンすれば良いだけですね。

```
class Solution {
public:
    TreeNode* findNearestRightNode(TreeNode* root, TreeNode* u) {
        queue<TreeNode*> q;
        q.push(root);

        while (!q.empty()) {
            int sz = q.size();
            TreeNode* prev = nullptr;
            for (int i = 0; i < sz; i++) {
                TreeNode* node = q.front();
                q.pop();

                if (node->left)
                    q.push(node->left);
                if (node->right)
                    q.push(node->right);

                if (prev && prev->val == u->val)
                    return node;
                prev = node;
            }
        }
        return nullptr;
    }
};
```