---
title: "プログラミングトレーニング(01/11/2021)"
date: 2021-01-11T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["競技プログラミング"]
---

先週一週間で解いた問題です（簡単で復習は不要かなと思ったものを除く）。

<!--more-->

## (LeetCode) Medium 1506. Find Root of N-Ary Tree
-------

https://leetcode.com/problems/find-root-of-n-ary-tree/

問題文がやたら分り辛い気もしますが、やることはルートのノードを見つけるだけです。基本的な解き方は、全てのノードの子ノードをチェックし、誰からも参照されなかったノードがルートであるというシンプルな理論です。

```
class Solution {
public:
    Node* findRoot(vector<Node*> tree) {
        set<Node*> seen;
        for (auto t : tree)
            for (auto c : t->children)
                seen.insert(c);
        for (auto t : tree)
            if (seen.find(t) == seen.end())
                return t;
        return nullptr;
    }
};
```

追加で、スペースオーダーをO(1)にするためにもう少し工夫することを考えてみます。各ノードの値はユニークであると定義されていますので、各ノードの値の総計を求め、それから各ノードの子ノードの値を引いて残った値がルートノードの値ということになります。

```
class Solution {
public:
    Node* findRoot(vector<Node*> tree) {
        int sum = 0;

        for (auto t : tree) {
            sum += t->val;
            for (auto c : t->children)
                sum -= c->val;
        }

        for (auto t : tree)
            if (t->val == sum)
                return t;

        return nullptr;
    }
};
```

さらに工夫して1パスで解くことを考えてみます。ユニークな値を一つ求めるというこの手の問題では、XORが有効的です（同じ値が2回出現したらXORの値がゼロになるので、一度しか出現していない値だけが最後に残る）。

```
class Solution {
public:
    Node* findRoot(vector<Node*> tree) {
        long long res = 0;

        for (auto t : tree) {
            res ^= (long long) t;
            for (auto c : t->children) {
                res ^= (long long) c;
            }
        }
        return (Node*) res;
    }
};
```

<br>

## (LeetCode) Easy 1237. Find Positive Integer Solution for a Given Equation
-------

https://leetcode.com/problems/find-positive-integer-solution-for-a-given-equation/


`f(x, y) = z` を満たす(x, y)の組み合わせを見つける問題です。x, yの取り得る値の範囲を全探索すれば解けます。問題の制約上、バイナリサーチも利用可能です。

```
class Solution {
public:
    vector<vector<int>> findSolution(CustomFunction& customfunction, int z) {
        vector<vector<int>> res;
        for (int x = 1; x < 1000; x++)
            for (int y = 1; y < 1000; y++) {
                if (z == customfunction.f(x, y)) {
                    res.push_back({x, y});
                    break;
                }
            }
        return res;
    }
};
```

もう少し少し探索範囲を枝刈りすることを考えます。`f(x,y) == z`を満たすx, yについて、xもしくはyが1増加すると必ずzも最低でも1増加する制約があるため、yは上限の方から見ていき、`f(x,y) == z`を満たすyがあったとして、そこからxが+1されたら次のyの候補は必ず前回のyの値よりも小さくなるため、基本的にyは減少方向に見ていけば良いです。


```
class Solution {
public:
    vector<vector<int>> findSolution(CustomFunction& customfunction, int z) {
        vector<vector<int>> res;
        int y = 1000;
        for (int x = 1; x <= 1000; x++) {
            while (y > 1 && customfunction.f(x,y) > z)
                y--;

            if (customfunction.f(x,y) == z)
                res.push_back({x, y});
        }
        return res;
    }
};
```

<br>

## (LeetCode) Medium 240. Search a 2D Matrix II
-------

https://leetcode.com/problems/search-a-2d-matrix-ii/

縦横それぞれの方向で数値が照準になっている2次元配列の中からターゲットの値が存在するかどうかをチェックする問題です。単純に全探索するとTLEになりますが、昇順であることを利用してO(M + N)で解くことが出来ます。左下からスタートして、ジグザクにスキャンする感じです。

```
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int M = matrix.size();
        int N = matrix[0].size();
        
        int j = 0;
        for (int i = M - 1; i >= 0; i--) {
            while (j < N && matrix[i][j] <= target) {
                if (matrix[i][j] == target)
                    return true;
                j++;
            }
        }
        return false;
    }
};
```

<br>


## (LeetCode) Medium 230. Kth Smallest Element in a BST
-------

https://leetcode.com/problems/kth-smallest-element-in-a-bst/

与えられたバイナリーツリーの中で'k'番目に小さい値を求める問題です。

与えられたツリーを全探索し、ヒープに突っ込んで最後に`k`個popしたときの値が答えというシンプルなアプローチです。

```
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        priority_queue<int, vector<int>, greater<int>> pq;
        dfs(root, pq);
        int res = 0;
        for (int i = 0; i < k; i++) {
            res = pq.top();
            pq.pop();
        }
        return res;
    }

private:
    void dfs(TreeNode* node, priority_queue<int, vector<int>, greater<int>>& pq) {
        if (!node)
            return;
        pq.push(node->val);
        dfs(node->left, pq);
        dfs(node->right, pq);
    }
};
```

ヒープを使わない方法を考えると、in-oderで探索すれば良いですね。そう言えば、dfsを再起コール使わずに実装するの初めてかも。

```
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        if (!root)
            return -1;
        
        int res = 0, count = 0;
        stack<TreeNode*> stk;
        while (1) {
            while (root) {
                stk.push(root);
                root = root->left;
            }
            
            root = stk.top();
            stk.pop();
            if (++count == k)
                return root->val;
            root = root->right;
        }
        return -1;
    }
};
```

<br>


## (LeetCode) Medium 82. Remove Duplicates from Sorted List II
-------

https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/

重複がある値のノードを全て削除したリストを求める問題です。ただ解くだけの問題ですが、リストの問題は多少の慣れが必要な気がします。

```
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if (!head || !head->next)
            return head;
        
        if (head->val == head->next->val) {
            int val = head->val;
            while (head && head->val == val)
                head = head->next;
            return deleteDuplicates(head);
        }
        
        head->next = deleteDuplicates(head->next);
        return head;
    }
};
```

<br>


## (LeetCode) Easy 1539. Kth Missing Positive Number
-------

https://leetcode.com/problems/kth-missing-positive-number/

-1するしないが少し混乱するけど、単に解くだけの問題です。

```
class Solution {
public:
    int findKthPositive(vector<int>& arr, int k) {
        int res = 1;
        int i = 0;
        int count = 0;
        while (count != k) {
            if (i >= arr.size() || res < arr[i]) {
                res++;
                count++;
            } else if (res == arr[i++])
                res++;
        }
        return res - 1;
    }
};
```

<br>


## (LeetCode) Medium 526. Beautiful Arrangement
-------

https://leetcode.com/problems/beautiful-arrangement/

`beautiful arrangements`と定義されている1-nの数値からなる順列の組み合わせ数を求める問題です。

 - perm[i]がiで割り切れる
 - iがperm[i]で割り切れる

全ての要素に対して、のどちらか一方を満たす順列の組み合わせ数を求めるために、条件を満たす順列をDFSで探索（作成）していきます。スワップ＆DFSで条件を満たす順列を作っていくのが最初よく理解できなかったです…。

```
class Solution {
public:
    int countArrangement(int N) {
        vector<int> v;
        for (int i = 0; i < N; i++)
            v.push_back(i + 1);
        
        int res = 0;
        dfs(v, res, 0);
        return res;
    }

private:
    void dfs(vector<int>& v, int& res, int idx) {
        if (idx == v.size()) {
            res++;
            return;
        }

        for (int i = idx; i < v.size(); i++) {
            if (v[i] % (idx + 1) == 0 || (idx + 1) % v[i] == 0) {
                swap(v[i], v[idx]);
                dfs(v, res, idx + 1);
                swap(v[i], v[idx]);
            }
        }
    }
};
```

<br>
