---

# Separation Logic

---

# Short Recap of Hoare Logic

^ Ale najpierw przypomnijmy sobie trochę logikę Hoara.

---

$$
\begin{aligned}
& \{\operatorname{\textit{array}}(a, i, j)\} \\
& \operatorname{procedure} \mathrm{ms}(a, i, j) \\ 
& \operatorname{newvar} m:=(i+j) / 2 \\
& \operatorname{if} i<j \operatorname{then} \\
& \text{ }\text{ } \operatorname{ms}(a, i, m) \\
& \text{ }\text{ } \operatorname{ms}(a, m+1, j) \\
& \text{ }\text{ } \operatorname{merge}(a, i, m+1, j) \\
& \{\operatorname{\textit{sorted}}(a, i, j)\} \\
\end{aligned}
$$

---

$$
\begin{aligned}
& \{\operatorname{\textit{array}}(a, i, j)\} \\
& \operatorname{procedure} \mathrm{ms}(a, i, j) \\ 
& \operatorname{newvar} m:=(i+j) / 2 \\
& \operatorname{if} i<j \operatorname{then} \\
& \text{ }\text{ } \{\operatorname{\textit{array}}(a, i, j)\} \\
& \text{ }\text{ } \{\operatorname{\textit{array}}(a, i, m) \wedge \operatorname{\textit{array}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{ms}(a, i, m) \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, m) \wedge \operatorname{\textit{array}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{ms}(a, m+1, j) \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, m) \wedge \operatorname{\textit{sorted}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{merge}(a, i, m+1, j) \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, j)\} \\
& \{\operatorname{\textit{sorted}}(a, i, j)\} \\
\end{aligned}
$$

^ Ze spełniającą reasonable wymagania mergem, mamy poprawny program.

---

# But why?

^ Ale czemu? Po co?
Otóż czasami - raczej rzadko w praktyce, fakt, ale jednak - chcielibyśmy wiedzieć na 100% że nasz program lub algorytm działa.
Ktoś może powiedzieć testy, otóż pisanie testów to świetna praktyka, jednak testy nie dają nam gwarancji, większość testów mógłbym obejść porządnym switchem.
Dowód, właśnie tego rodzaju, pozwala mi potwierdzić na 100%, że dany program działa.

---

# Adding the Heap

---

# Adding the Heap

![inline 100%](linked_list.png)

---

# Adding the Heap

$$
\begin{aligned}
& \text{Store}_V = V \rightarrow \text{Values} \\
& \text{Heap} = \text{Addresses} \rightarrow \text{Values} \\
\\
& \text{Values} = \text{Addresses} \cup \text{Atoms} \\
\end{aligned}
$$

^ Idea była na Semantyce i Weryfikacji programów.
Store (nasze zmienne) zaczynają wskazywać na int
Heap (pamięć alokowalna) wskazuje z adresu na int
Tak ogółem.

---

# Adding the Heap

$$
\begin{aligned}
& \text{Store}_V = V \rightarrow \text{Integers} \\
& \text{Heap} = \text{Integers} \rightarrow \text{Integers} \\
\end{aligned}
$$

^ W praktyce nie będziemy tego zgłębiać, więc po prostu wszędzie są liczby całkowite.

---

# Adding the Heap

$$
\begin{aligned}
&\langle\mathrm{comm}\rangle: =\cdots \\
&\mid\langle\mathrm{var}\rangle :=\operatorname{cons}(\langle\exp \rangle, \ldots,\langle\exp \rangle) & \text { allocation } \\
&\mid\langle\mathrm{var}\rangle :=[\langle\exp \rangle] & \text { lookup } \\
&\mid[\langle\exp\rangle]:=\langle\exp\rangle & \text { mutation } \\
&\mid dispose \langle\exp \rangle & \text { deallocation }
\end{aligned}
$$

^ Rozszczerzymy też nasz język o potrzebne konstrukty. tzw. Komendy.
Tu warto zwrócić uwagę, że albo odczytujemy, albo zapisujemy do pamięci, nie możemy zrobić przepisania z pamięci do pamięci.
Trochę jak w assemblerze.

---

# Tree Example

![inline 125%](tree_example.png)

^ Spójrzmy teraz na drzewo, troszkę bardziej złożoną od listy strukturę wskaźnikową.

---

# Tree Example

$$
\begin{aligned}
& \operatorname{tree}(v, \tau_{1}, \tau_{2})(a) \operatorname{iff} \\
& \text{ }\text{ } [a] = v \wedge \operatorname{tree}\tau_{1}(a+1) \wedge \operatorname{tree}\tau_{2}(a+2) \\
& \operatorname{tree}(\bot)(a) \operatorname{iff} \\
& \text{ }\text{ } a = -1 \\
\end{aligned}
$$

^ Ale co to znaczy "być drzewem"? Pomóc może tutaj oczywiście lektura "Sekretne życie drzew", ale w skrócie:

---

# Tree Example

<Bad proof sketch of deletetree using pure Hoare logic.>

$$
\begin{aligned}
& \{ \mathbf{emp} \} \\
& x = \operatorname{cons}(1) \\
& \{ x \mapsto 1 \} \\
& y = \operatorname{cons}(2, 3) \\
& \{ x \mapsto 1 \wedge y \mapsto 2 \wedge y + 1 \mapsto 3 \} \\
& \{ x \mapsto 1 \wedge y \mapsto 2, 3 \} \\
& \{ x \mapsto - \wedge y \mapsto 2, 3 \} \\
& \operatorname{dispose} x \\
& \{ y \mapsto 2, 3 \} \\
& \operatorname{dispose} y \\
& \{ \mathbf{emp} \} \\
\end{aligned}
$$

---

# Adding Heap-related Predicates - Assertions

$$
\begin{aligned}
&\langle\text { assert }\rangle::=\cdots\\
&\mid \mathbf{emp} & \text { empty heap } \\
&\mid\langle\exp \rangle \mapsto\langle\exp \rangle & \text { singleton heap } \\
\end{aligned}
$$

^ Wprowadźmy też predykaty operujące na stanie Heapu, tzw. asercje.

---

# Adding Heap-related Predicates - Assertions

$$
\begin{aligned}
& \{ \mathbf{emp} \} \\
& x = \operatorname{cons}(1) \\
& \{ x \mapsto 1 \} \\
& y = \operatorname{cons}(2, 3) \\
& \{ x \mapsto 1 \wedge y \mapsto 2 \wedge y + 1 \mapsto 3 \} \\
& \{ x \mapsto 1 \wedge y \mapsto 2, 3 \} \\
& \{ x \mapsto - \wedge y \mapsto 2, 3 \} \\
& \operatorname{dispose} x \\
& \{ y \mapsto 2, 3 \} \\
& \operatorname{dispose} y \\
& \{ \mathbf{emp} \} \\
\end{aligned}
$$

---

# Tree Example

<Bad proof of deletetree using pure Hoare logic.>

---

# Tree Example

<Counterexample tree, we'd like to assert that the tree isn't malformed like that.>

---

# Tree Example

<Intro with simple examples of logic extensions. Ownership and seperating conjunction.>

---

# Tree Example

<Back to the tree example. How to fix this? By asserting which parts of the predicate operate on which parts of memory. That's what separation logic is all about>

---

$$
\frac{\{P\} C\{Q\} \quad\left\{P^{\prime}\right\} C^{\prime}\left\{Q^{\prime}\right\}}{\left\{P * P^{\prime}\right\} C \| C^{\prime}\left\{Q * Q^{\prime}\right\}}
$$

---

$$
\frac{\{x \mapsto-\}[x]:=4\{x \mapsto 4\}}{\{x \mapsto-\wedge y \mapsto 3\}[x]:=4\{x \mapsto 4 \wedge y \mapsto 3\}}
$$
