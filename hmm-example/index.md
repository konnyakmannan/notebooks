---
title: 隠れマルコフモデルを hmmlearn で解く
description: 自明な隠れマルコフモデル(HMM)を解く例
date: 2023-03-09
tags: ['Python', 'Modeling']
draft: true
---

このノートは

- [ネットワーク行動学 -都市と移動-](http://bin.t.u-tokyo.ac.jp/kaken/)
  - 特に図3.15の数値例
- [隠れマルコフモデル 入門 | Kabuku Developers Blog](https://www.kabuku.co.jp/developers/hmm)

を参考に作成しました。
上記参考資料の執筆者の方に多謝です。

## 隠れマルコフモデル

隠れマルコフモデル (HMM) は観測値と直接観測できない (隠れた) 値で構成される。
前者を確率変数 $o_t$、後者を潜在確率変数 $q_t$ と書く。
それぞれの取り得る状態を $o_t\in \{v_0, v_1,\,...v_{N-1}\}$
および $q_t\in \{s_0, s_1,\,...s_{M-1}\}$ と定義する。

HMMには３つのパラメータがある。

ひとつめは状態遷移確率 $A=\{a_{ij}\}_{i,j=0}^{M-1}$

$$
a_{ij} = \mathrm{Prob}(q_t=s_j | q_{t-1}=s_i)
$$

ふたつめは出現確率 $B=\{\{b_{ij}\}_{i=0}^{M-1}\}_{j=0}^{N-1}$

$$
b_{ij} = \mathrm{Prob}(o_t=v_j | q_t=s_i)
$$

最後は初期状態確率 $\pi=\{ \pi_{i} \}_{i=0}^{M-1}$

$$
\pi_{i} = \mathrm{Prob}(q_0=s_i)
$$

以上はまとめて $\lambda = (A,\,B,\,\pi)$と略記される。

HMM に関して3つの問題設定が考えうる。

1. **評価問題**
  - $\{o_t\}_{t=0}^{M-1}$ と $\lambda$ が与えられたとき $\mathrm{Prob}(\{o_t\}_{t=0}^{N-1}|\lambda)$ を求める
2. **復号化問題**
  - $\{o_t\}_{t=0}^{M-1}$ と $\lambda$ が与えられたとき最適な $\{q_t\}_{t=0}^{M-1}|\lambda$ を求める
3. **推定問題**
  - $\{o_t\}_{t=0}^{M-1}$ が与えられたとき $\lambda$ を求める

以下の具体例では「複合化問題」を考える。

### Remarks

- ここでは $q_t$ が離散値をとる離散型HMMをあつかう
- $q_t$ から観測値から出力されるMooreマシンをあつかう

## 具体例

以下の数値でモデルを定義する。

- 観測値変数 : $o_t\in \{a, b\}$
- 潜在変数 : $q_t\in \{\alpha,\, \beta\}$

- 遷移確率 $A$ :

$$
\begin{array}{c|c|c}
q_t\backslash q_{t+1}& \alpha & \beta \\ \hline
\alpha & 0.7 & 0.3 \\ \hline
\beta & 0.0 & 1.0
\end{array}
$$

- 出現確率 $B$ :

$$
\begin{array}{c|c|c}
q_t\backslash o_t& a & b \\ \hline
\alpha & 0.6 & 0.4 \\ \hline
\beta & 0.2 & 0.8
\end{array}
$$

- 初期状態確率 $\pi$ :

$$
\begin{array}{c|c}
q_0 & & \\ \hline
\alpha & 1.0 & \\ \hline
\beta & 0.0 &
\end{array}
$$

このモデルについて
「観測系列 : "$abb$" と上記 $\lambda=(A,\, B,\, \pi)$ が与えられたとき、最適な潜在変数の系列はなにか？」
という自明な問題を考える。

### 手計算

設定上、取りうる潜在変数の系列は3通りあり、
それぞれ潜在系列が観測系列を出力する同時確率をすべて計算できる。

- 1 : "$\alpha \alpha \alpha$"
  - $1.0 \times 0.6\times 0.7\times 0.4\times 0.7\times 0.4 = 0.04704$
- 2 : "$\alpha \alpha \beta$"
  - $1.0 \times 0.6\times 0.7\times 0.4\times 0.3\times 0.8 = 0.04032$
- 3 : "$\alpha \beta \beta$"
  - $1.0 \times 0.6\times 0.3\times 0.8\times 1.0\times 0.8 = 0.1152$

結果、"$\alpha \beta \beta$" が最尤系列であると言える。

### hmmlearn

[hmmlearn](https://hmmlearn.readthedocs.io/en/latest/) を使って同じ問題を解く。


`numpy` と `hmmlearn` は下記 version を使う。

```python
import hmmlearn
from hmmlearn import hmm
import numpy as np

print(f'{hmmlearn.__version__=}')
print(f'{np.__version__=}')

# Output:
# hmmlearn.__version__='0.2.6'
# np.__version__='1.21.3'
```

数値を変数に定義する。
観測値と状態はそれぞれ 0 と 1 にマップした。

```python
# 観測値
observe_states = {"a":0, "b":1}
observes = ["a", "b", "b"]
n_samples = 1
# str を 0 / 1 に置きかえ
observe_codes = np.array(
  [observe_states[o] for o in observes]
  ).reshape((len(observes), n_samples))

# 潜在状態
states = {"alpha":0, "beta":1}
inv_dict_states = {str(v): k for k, v in states.items()}

# 初期状態確率
startprob = np.array([1.0, 0.0])
# 遷移確率
transmat = np.array([[0.7, 0.3], [0.0, 1.0]])
# 出現確率
emmisionprob = np.array([[0.6, 0.4], [0.2, 0.8]])
```

`MultinomialHMM` インスタンスを作成し、
パラメータをセットする。

```python
# hmmlearn のインスタンス作成
model = hmm.MultinomialHMM(
    n_components=len(states), init_params='', params=''
)
model.n_features = len(observe_states)
model.startprob_= startprob
model.transmat_= transmat
model.emissionprob_= emmisionprob
# 推定
logprob, decoded_codes = model.decode(observe_codes)

decoded = [inv_dict_states[str(d)] for d in decoded_codes]
print(f'{decoded=}')
print(f'{np.exp(logprob)=}')

# Output:
# decoded=['alpha', 'beta', 'beta']
# np.exp(logprob)=0.1152
```

推定の結果、同時確率が手計算の結果と一致し、潜在系列が期待どおりとなった。

