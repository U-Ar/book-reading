# 3章 文法 演習問題

## 3.1 論理積、論理和、論理否定の演算子を含文法GL = <PL,L>について、非終端記号の集合と終端記号の集合、語彙を求めよ。

$$
PL = \{ L \rightarrow A || L \vert A \\ A \rightarrow N \&\& A \vert N \\ N \rightarrow ! N \vert (L) \vert p\}
$$

非終端記号 $V_N = \{L, A, N\}$

終端記号 $V_T = \{\&\& , ||, !, p\}$

語彙 $V = V_N \cup V_T$

## 3.2 前問の文法GLの生成規則をBNFに置き換えよ。

- $\langle L \rangle ::= \langle A \rangle || \langle L \rangle | \langle A \rangle$
- $\langle A \rangle ::= \langle N \rangle \&\& \langle A \rangle | \langle N \rangle$
- $\langle N \rangle ::= !\langle N \rangle | (\langle L \rangle) | p$

## 3.3 文法GLの文 p && !p || p を を生成する最左導出と最右導出を求めよ。


