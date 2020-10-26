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
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \exists_{t_1, t_2} a \mapsto x, t_1, t_2 \wedge \operatorname{tree}(\tau_2)(t_2) \} \\
& \text{ }\text{ } \operatorname{deletetree}([a+2]) \\
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \exists_{t_1, t_2} a \mapsto x, t_1, t_2 \} \\
& \text{ }\text{ } \operatorname{dispose} a \\
& \text{ }\text{ } \{ \mathbf{emp} \} \\
& \{ \mathbf{emp} \} \\
\end{aligned}
$$


^ Pysznie. Jedyny problem - jest źle.

---

# Tree Counterexample

![inline 125%](tree_counterexample.png)

^ Otóż nigdzie nie sprawdzamy ani nie obsługujemy sytuacji, w której różne gałęzie drzewa wskazują na te same miejsca w Heapie.
Śmiało zrobimy double free.
Nigdzie nie rozgraniczamy pamięci, więc różne części naszych predukatów mogą operować na tej samej pamięci.

---

# Assertions - continued

$$
\begin{aligned}
&\langle\text { assert }\rangle::=\cdots\\
&\mid \mathbf{emp} & \text { empty heap } \\
&\mid\langle\exp \rangle \mapsto\langle\exp \rangle & \text { singleton heap } \\
&\mid\langle\text{assert}\rangle * \langle\text{assert}\rangle & \text { separating conjunction } \\
\end{aligned}
$$

^ Wobec tego musimy dodać kolejne asercje, które są główną ideą i głębszym sensem Separation Logic.
Idea jest taka, że asercje oddzielona separating conjunction opisują różne elementy pamięci.
Idei ciąg dalszy jest taki, że każdy fragment oddzielony przez seperating conjunction w pewnym sensie "posiada" assertowane adresy.

---

# Seperating Conjunction - Example

![inline 125%](separating_conjunction_example_1.png)

$$
x \mapsto 3, y
$$

^ x wskazuje na rekord 3, y. Wyrażenie to posiada x'a.

---

# Seperating Conjunction - Example

![inline 125%](separating_conjunction_example_2.png)

$$
x \mapsto 3, y * y \mapsto 3, x
$$

^ x wskazuje na rekord 3, y. y na rekord 3, x. To działa dlatego bo posiadany jest adres po lewej, a całość się nie rozchodzi dalej.

---

# Seperating Conjunction - WRONG Example

![inline 125%](separating_conjunction_example_3.png)

$$
x \mapsto 3, z * y \mapsto 3, z
$$

^ Nie działa, mamy dwie asercje rozdzielone separating conjunction operujące na tym samym adresie.

---

# Back to the Tree Example

^ Dobra, to wracając do drzewa. Jak to naprawić?
Otóż możemy to naprawić zmieniając definicję drzewa, na taką, które rozdziela pamięć.

---

# Back to the Tree Example

$$
\begin{aligned}
& \operatorname{tree}(v, \tau_{1}, \tau_{2})(a) \operatorname{iff} \\
& \text{ }\text{ } \exists_{v, t_1, t_2} a \mapsto v, t_1, t_2 * \operatorname{tree}\tau_{1}t_1 * \operatorname{tree}\tau_{2}t_2 \\
& \operatorname{tree}(\bot)(a) \operatorname{iff} \\
& \text{ }\text{ } \mathbf{emp} \wedge a = -1 \\
\end{aligned}
$$

^ Zastępujemy koniunkcję przez Seperating Conjunction. I to pokazuje całą ideę Separation Logic.
Rozdzielamy asercje tak żeby operowały na różnych elementach pamięci.
Tu warto sobie uświadomić, że (tree tau t) rozwija się do asercji, która ma po lewych stronach strzałek wszystkie adresy wskazujące na węzły-dzieci, więc asertuje ich obecność.
W przypadku braku czegokolwiek, zaznaczamy, że nie operujemy na żadnej pamięci, a nasze a jest null pointerem.

---

# Back to the Tree Example - Implementation

$$
\begin{aligned}
& \{ \operatorname{\operatorname{tree}}(\tau)(a) \} \\
& \operatorname{procedure} \mathrm{deletetree}(a) \\ 
& \operatorname{if} a \neq -1 \operatorname{then} \\
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \operatorname{tree}(x, \tau_1, \tau_2)(a) \} \\
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \exists_{t_1, t_2} a \mapsto x, t_1, t_2 * \operatorname{tree}(\tau_1)(t_1) * \operatorname{tree}(\tau_2)(t_2) \} \\
& \text{ }\text{ } \operatorname{deletetree}([a+1]) \\
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \exists_{t_1, t_2} a \mapsto x, t_1, t_2 * \mathbf{emp} * \operatorname{tree}(\tau_2)(t_2) \} \\
& \text{ }\text{ } \operatorname{deletetree}([a+2]) \\
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \exists_{t_1, t_2} a \mapsto x, t_1, t_2 * \mathbf{emp} * \mathbf{emp} \} \\
& \text{ }\text{ } \operatorname{dispose} a \\
& \text{ }\text{ } \{ \mathbf{emp} * \mathbf{emp} * \mathbf{emp} \} \\
& \{ \mathbf{emp} \} \\
\end{aligned}
$$

^ W końcu możemy poprawnie udowodnić deletetree. Nasz wcześniejszy kontrprzykład nie może się wydarzyć, bo adres nie może być użyty w dwóch poddrzewach, jako że są one rozdzielone separating conjunction.

---

# Inference Rules

^ W praktyce dostajemy też zestaw reguł wnioskowania których możemy używać podobnie do reguł wnioskowania w logice Hoara.
Przejdźmy przez nie pokrótce, chociaż po szczegóły proszę zajrzeć sobie do papieru.

---

# Consequence

$$
\frac{\begin{aligned} & p^{\prime} \Rightarrow p & \{p\}c\{q\} & & q \Rightarrow q^{\prime} \end{aligned}}{\{p^{\prime}\} c \{q^{\prime}\}}
$$

^ Gdzie p, q to asercje, a c to komenda.

---

# Auxiliary Variable Elimination

$$
\frac{\{p\}c\{q\}}{\{\exists_{v} p\} c \{\exists_{v} q\}} v \notin \operatorname{FV}(c)
$$

---

# Substitution

$$
\frac{\{p\} c\{q\}}{(\{p\} c\{q\}) / v_{1} \rightarrow e_{1}, \ldots, v_{n} \rightarrow e_{n}}
$$
when
$$
v_{1}, \ldots, v_{n} \in FV(p, c, q) \wedge \forall_i \operatorname{modified}_{c}(v_{i}) \implies \operatorname{variable}(e_{i}) \wedge \forall_{j \neq i} e_{i} \notin \operatorname{FV}(e_{j})
$$

---

# End of Part 1

---

# Separation Logic in Concurrency

---

$$
\frac{\{P\} C\{Q\} \quad\left\{P^{\prime}\right\} C^{\prime}\left\{Q^{\prime}\right\}}{\left\{P * P^{\prime}\right\} C \| C^{\prime}\left\{Q * Q^{\prime}\right\}}
$$

---

$$
\frac{\{x \mapsto-\}[x]:=4\{x \mapsto 4\}}{\{x \mapsto-\wedge y \mapsto 3\}[x]:=4\{x \mapsto 4 \wedge y \mapsto 3\}}
$$
