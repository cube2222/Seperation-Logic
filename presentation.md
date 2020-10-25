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
I tak samo jak w prawdziwych komputerach, double free prowadzi do segfaulta.

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

$$
\begin{aligned}
& \{ \mathbf{true} \} \\
& \text{left} = \operatorname{cons}(42, -1, -1) \\
& \{ \operatorname{tree}(42, \bot, \bot)(\text{left}) \} \\
& \text{right} = \operatorname{cons}(47, -1, -1) \\
& \{ \operatorname{tree}(47, \bot, \bot)(\text{right}) \} \\
& \text{root} = \operatorname{cons}(32, \text{left}, \text{right}) \\
& \{ \operatorname{tree}(32, (42, \bot, \bot), (47, \bot, \bot))(\text{root}) \} \\
& \operatorname{deletetree}(\text{root}) \\
& \{ ??? \} \\
\end{aligned}
$$

^ To zobaczmy sobie teraz przykład konstrukcji i użycia.
Tu mamy problem.
Nie mamy jak dać warunku na skuteczne usunięcie drzewa...

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

# Back to the Tree Example

$$
\begin{aligned}
& \{ \mathbf{true} \} \\
& \text{left} = \operatorname{cons}(42, -1, -1) \\
& \{ \text{left} \mapsto 42, -1, -1 \wedge \operatorname{tree}(42, \bot, \bot)(\text{left}) \} \\
& \text{right} = \operatorname{cons}(47, -1, -1) \\
& \{ \text{right} \mapsto 47, -1, -1 \wedge \operatorname{tree}(47, \bot, \bot)(\text{right}) \wedge \text{left} \mapsto 42, -1, -1 \wedge \operatorname{tree}(42, \bot, \bot)(\text{left}) \} \} \\
& \text{root} = \operatorname{cons}(32, \text{left}, \text{right}) \\
& \{ \text{right} \mapsto 32, \text{left}, \text{right} \wedge \operatorname{tree}(32, (42, \bot, \bot), (47, \bot, \bot))(\text{root}) \wedge \text{right} \mapsto 47, -1, -1 \wedge \operatorname{tree}(47, \bot, \bot)(\text{right}) \wedge \text{left} \mapsto 42, -1, -1 \wedge \operatorname{tree}(42, \bot, \bot)(\text{left}) \} \\
& \{ \text{right} \mapsto 32, \text{left}, \text{right} \wedge \operatorname{tree}(32, (42, \bot, \bot), (47, \bot, \bot))(\text{root}) \wedge \text{right} \mapsto 47, -1, -1 \wedge \text{left} \mapsto 42, -1, -1 \} \\
& \operatorname{deletetree}(\text{root}) \\
& \{ \mathbf{emp} \} \\
\end{aligned}
$$

^ Dobra, teraz poszło w miarę gładko, chociaż trochę się tych asercji narobiło.
Przejdźmy teraz głębiej, do implementacji deletetree.

---

# Tree Example - Better Description

$$
\begin{aligned}
& \operatorname{tree}(v, \tau_{1}, \tau_{2})(a) \operatorname{iff} \\
& \text{ }\text{ } \exists_{v, t_1, t_2} a \mapsto v, t_1, t_2 \wedge \operatorname{tree}\tau_{1}t_1 \wedge \operatorname{tree}\tau_{2}t_2 \\
& \operatorname{tree}(\bot)(a) \operatorname{iff} \\
& \text{ }\text{ } a = -1 \\
\end{aligned}
$$

^ Musieliśmy obok asercji o byciu drzewem pisać też asercje o pamięci, to nas od tego uwolni.

---

# Tree Example - Implementation

$$
\begin{aligned}
& \{ \operatorname{tree}(\tau)(a)\} \\
& \operatorname{procedure} \mathrm{deletetree}(a) \\ 
& \operatorname{if} a \neq -1 \operatorname{then} \\
& \text{ }\text{ } \operatorname{deletetree}([a+1]) \\
& \text{ }\text{ } \operatorname{deletetree}([a+2]) \\
& \text{ }\text{ } \operatorname{dispose} a \\
& \{ \mathbf{emp} \} \\
\end{aligned}
$$


^ Wygląda sensownie. Przechodząc do głębszego dowodu.

---

# Tree Example - Implementation

$$
\begin{aligned}
& \{ \operatorname{\operatorname{tree}}(\tau)(a) \} \\
& \operatorname{procedure} \mathrm{deletetree}(a) \\ 
& \operatorname{if} a \neq -1 \operatorname{then} \\
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \operatorname{tree}(x, \tau_1, \tau_2)(a) \} \\
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \exists_{t_1, t_2} a \mapsto x, t_1, t_2 \wedge \operatorname{tree}(\tau_1)(t_1) \wedge \operatorname{tree}(\tau_2)(t_2) \} \\
& \text{ }\text{ } \operatorname{deletetree}([a+1]) \\
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \exists_{t_1, t_2} a \mapsto x, t_1, t_2 \wedge t_1 = -1 \wedge \operatorname{tree}(\tau_2)(t_2) \} \\
& \text{ }\text{ } \operatorname{deletetree}([a+2]) \\
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \exists_{t_1, t_2} a \mapsto x, t_1, t_2 \wedge t_1 = -1 \wedge t_2 = -1 \} \\
& \text{ }\text{ } \operatorname{dispose} a \\
& \text{ }\text{ } \{ \mathbf{emp} \} \\
& \{ \mathbf{emp} \} \\
\end{aligned}
$$


^ Pysznie. Jedyny problem - jest źle.

---

# Tree Example

<Counterexample tree, we'd like to assert that the tree isn't malformed like that.>

^ Otóż nigdzie nie sprawdzamy ani nie obsługujemy sytuacji, w której różne gałęzie drzewa wskazują na te same miejsca w Heapie.

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
