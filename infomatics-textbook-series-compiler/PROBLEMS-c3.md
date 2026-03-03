# 3章 文法 演習問題

## 3.1 論理積、論理和、論理否定の演算子を含文法GL = <PL,L>について、非終端記号の集合と終端記号の集合、語彙を求めよ。

$$
PL = \{ L \rightarrow A || L \vert A \\
 A \rightarrow N \&\& A \vert N \\
 N \rightarrow ! N \vert (L) \vert p\}
$$

非終端記号 $V_N = \{L, A, N\}$

終端記号 $V_T = \{\&\& , ||, !, p\}$

語彙 $V = V_N \cup V_T$

## 3.2 前問の文法GLの生成規則をBNFに置き換えよ。

- $\langle L \rangle ::= \langle A \rangle || \langle L \rangle | \langle A \rangle$
- $\langle A \rangle ::= \langle N \rangle \&\& \langle A \rangle | \langle N \rangle$
- $\langle N \rangle ::= !\langle N \rangle | (\langle L \rangle) | p$

## 3.3 文法GLの文 p && !p || p を を生成する最左導出と最右導出を求めよ。

最左導出

$$
L \\
\rightarrow A || L \\
\rightarrow N \&\& A || L \\
\rightarrow p \&\& A || L \\
\rightarrow p \&\& N || L \\
\rightarrow p \&\& ! N || L \\
\rightarrow p \&\& ! p || L \\
\rightarrow p \&\& ! p || A \\
\rightarrow p \&\& ! p || N \\
\rightarrow p \&\& ! p || p
$$


最右導出

$$
L \\
\rightarrow A || L \\
\rightarrow A || N \\
\rightarrow A || p \\
\rightarrow N \&\& A || p \\
\rightarrow N \&\&  N || p \\
\rightarrow N \&\& ! N || p \\
\rightarrow N \&\& ! p || p \\
\rightarrow p \&\& ! p || p 
$$

## 3.4 文法GLにおいて、論理積が論理和より優先順位が高いことを示せ。

$L \rightarrow A || L$ が存在し、 左の項について $A \rightarrow N \&\& A$ なので $\&\& > ||$

$L \rightarrow A || L$ が存在し、右の項について $L \rightarrow A \rightarrow N \&\& A$ なので $|| < \&\&$

以上より論理積の方が優先順位が高い。
