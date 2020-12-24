---
title: "編集距離 / 2つの文字列の類似度に関する問題"
date: 2020-12-25T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["競技プログラミング"]
---

DP (Dynamic Programming) を利用した典型的な問題である、2つの文字列の編集距離（類似度）に関連するお勉強ログです。

<!--more-->

## 編集距離とは
-------

2つの文字列の[編集距離](https://ja.wikipedia.org/wiki/%E3%83%AC%E3%83%BC%E3%83%99%E3%83%B3%E3%82%B7%E3%83%A5%E3%82%BF%E3%82%A4%E3%83%B3%E8%B7%9D%E9%9B%A2)とは、ある文字列Aに対して、1文字の挿入・削除・変換によって、もう一方の文字列Bに変換するのに必要な手順の最小回数です。

では、今回お勉強するLeedCodeの問題は以下です。

 - [718. Maximum Length of Repeated Subarray](https://leetcode.com/problems/maximum-length-of-repeated-subarray/)
 - [1143. Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)

<br>

## 718. Maximum Length of Repeated Subarray
-------

#### 問題文

これは編集距離とは直接は関係ありませんが、2つの文字列の共通の部分文字列の最大の長さを計算する問題です。

{{< figure src="./leetcode-question-719.png" title="" >}}

<br>

#### 考え方

まず文字列Aと文字列Bの2つを以下のようにDPのマトリックスを取ります。
{{< figure src="./leetcode-question-719-description.png" title="" >}}

文字列A（もしくはB）の各文字に対して、もう一方の文字列の各文字が一致しているかどうかを見ていき、
もし文字が一致していればその時点で部分文字列の最低文字数が1が確定します。
この時に、一つ前の文字がさらに一致していれば、部分文字列の長さは一つ前のdpの値+1となります。
つまり式にすると以下のようになります。

```
if A[i] == B[j] then
    dp[i][j] = dp[i - 1][j - 1] + 1
else
    dp[i][j] = 0
endif
```

あとはdpの中の最大値が答え（部分文字列の最大長）となります。文字列Aの長さをN, 文字列Bの長さをMとすると、Time complexity: O(NM), Space complexity: O(NM)

<br>

#### 解答

```
class Solution {
public:
    int findLength(vector<int>& A, vector<int>& B) {
        int M = A.size();
        int N = B.size();
        vector<vector<int>> dp(M + 1, vector<int>(N + 1, 0));
        int res = 0;

        for (int i = 1; i <= M; i++) {
            for (int j = 1; j <= N; j++) {
                if (A[i - 1] == B[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                    res = max(res, dp[i][j]);
                }
            }
        }
        
        return res;
    }
};
```

<br>


## 1143. Longest Common Subsequence
-------

#### 問題文

先程の問題の応用でこれが編集距離の問題です。先程の問題との違いですが、必ずしも各文字同士が隣り合っている必要がないという点です。
つまり、各文字の間に適当な文字を挿入したり削除したりしても良いわけです。

{{< figure src="./leetcode-question-1143.png" title="" >}}

<br>

#### 考え方

[アルゴリズム1000本ノック #3. Longest common subsequence](https://qiita.com/_rdtr/items/c49aa20f8d48fbea8bd2)にとても詳しく解説があるのでこれを参考にしてください。

式にすると以下です。

```
if A[i] == B[j] then
    dp[i][j] = dp[i - 1][j - 1] + 1
else
    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
endif
```

<br>

#### 解答

```
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int M = text1.size();
        int N = text2.size();
        vector<vector<int>> dp(M + 1, vector<int>(N + 1, 0));

        for (int i = 1; i <= M; i++) {
            for (int j = 1; j <= N; j++) {
                if (text1[i - 1] == text2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        return dp[M][N];
    }
};
```

<br>

## 参考書
-------

競技プログラミングやデータ構造アルゴリズムの学習本としてかなりお勧めな以下の本の5.5章でも解説があります。

{{< amazon category="amazon-books" key="問題解決力を鍛える!アルゴリズムとデータ構造" >}}

