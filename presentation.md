---

[.footer: Gist: Hey, I'm a footer. If you didn't hear a part of what was said and don't understand a slide, I should be there to state the main point of it.]

---

# Separation Logic

^ Autorem tego papieru jest John C. Reynolds. Stawia podstawy do tego co później zostało nagrodzone nagrodą Goedla. The Gödel Prize is an annual prize for outstanding papers in the area of theoretical computer science.

---

# Short Recap of Hoare Logic

^ Ale najpierw przypomnijmy sobie trochę logikę Hoara, która służy do dowodzenia poprawności sekwencyjnych programów bez wskaźników, czyli z samymi rejestrami jako wartości.

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
& \text{ }\text{ } \{\operatorname{\textit{array}}(a, i, m) \land \operatorname{\textit{array}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{ms}(a, i, m) \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, m) \land \operatorname{\textit{array}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{ms}(a, m+1, j) \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, m) \land \operatorname{\textit{sorted}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{merge}(a, i, m+1, j) \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, j)\} \\
& \{\operatorname{\textit{sorted}}(a, i, j)\} \\
\end{aligned}
$$

^ Ze spełniającą reasonable wymagania mergem, mamy poprawny program.
Funkcja jest rekurencyjna, więc możemy o rekurencyjnych wywołaniach czynić te założenia, które próbujemy udowodnić dla funkcji.

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

![inline 80%](linked_list.png)

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

[.footer: Gist: We're adding a layer of indirection.]

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

[.footer: Gist: Commands to manipulate the heap.]

^ Rozszczerzymy też nasz język o potrzebne konstrukty. tzw. Komendy.
Tu warto zwrócić uwagę, że albo odczytujemy, albo zapisujemy do pamięci, nie możemy zrobić przepisania z pamięci do pamięci. Trochę jak w assemblerze.
Czyli kwadratowy nawias nie może być po obu stronach przypisania jednocześnie.
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
& \text{ }\text{ } [a] = v \land \operatorname{tree}\tau_{1}(a+1) \land \operatorname{tree}\tau_{2}(a+2) \\
& \operatorname{tree}(\bot)(a) \operatorname{iff} \\
& \text{ }\text{ } a = -1 \\
\end{aligned}
$$

[.footer: Gist: Describing an address which contains a tree.]

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

[.footer: Gist: Can't assert memory deallocation.]

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

[.footer: Gist: Assertions to describe the state of the heap.]

^ Wprowadźmy też predykaty operujące na stanie Heapu, tzw. asercje.

---

# Adding Heap-related Predicates - Assertions

$$
\begin{aligned}
& \{ \mathbf{emp} \} \\
& x = \operatorname{cons}(1) \\
& \{ x \mapsto 1 \} \\
& y = \operatorname{cons}(2, 3) \\
& \{ x \mapsto 1 \land y \mapsto 2 \land y + 1 \mapsto 3 \} \\
& \{ x \mapsto 1 \land y \mapsto 2, 3 \} \\
& \{ x \mapsto - \land y \mapsto 2, 3 \} \\
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
& \{ \text{left} \mapsto 42, -1, -1 \land \operatorname{tree}(42, \bot, \bot)(\text{left}) \} \\
& \text{right} = \operatorname{cons}(47, -1, -1) \\
& \{ \text{right} \mapsto 47, -1, -1 \land \operatorname{tree}(47, \bot, \bot)(\text{right}) \land \text{left} \mapsto 42, -1, -1 \land \operatorname{tree}(42, \bot, \bot)(\text{left}) \} \} \\
& \text{root} = \operatorname{cons}(32, \text{left}, \text{right}) \\
& \{ \text{root} \mapsto 32, \text{left}, \text{right} \land \operatorname{tree}(32, (42, \bot, \bot), (47, \bot, \bot))(\text{root}) \land \text{right} \mapsto 47, -1, -1 \land \operatorname{tree}(47, \bot, \bot)(\text{right}) \land \text{left} \mapsto 42, -1, -1 \land \operatorname{tree}(42, \bot, \bot)(\text{left}) \} \\
& \{ \text{root} \mapsto 32, \text{left}, \text{right} \land \operatorname{tree}(32, (42, \bot, \bot), (47, \bot, \bot))(\text{root}) \land \text{right} \mapsto 47, -1, -1 \land \text{left} \mapsto 42, -1, -1 \} \\
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
& \text{ }\text{ } \exists_{v, t_1, t_2} a \mapsto v, t_1, t_2 \land \operatorname{tree}\tau_{1}t_1 \land \operatorname{tree}\tau_{2}t_2 \\
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
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \exists_{t_1, t_2} a \mapsto x, t_1, t_2 \land \operatorname{tree}(\tau_1)(t_1) \land \operatorname{tree}(\tau_2)(t_2) \} \\
& \text{ }\text{ } \operatorname{deletetree}([a+1]) \\
& \text{ }\text{ } \{ \exists_{x, \tau_1, \tau_2} \exists_{t_1, t_2} a \mapsto x, t_1, t_2 \land \operatorname{tree}(\tau_2)(t_2) \} \\
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

[.footer: Gist: Same node freed in two branches.]

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

[.footer: Gist: Add assertions to describe non-overlapping parts of the heap.]

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

# Seperating Conjunction - Non-Example

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
& \text{ }\text{ } \mathbf{emp} \land a = -1 \\
\end{aligned}
$$

[.footer: Gist: Use separating conjunction to assert branches use disjoint parts of the heap.]

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

[.footer: Gist: Using **emp** to describe deletion correctness.]

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
v_{1}, \ldots, v_{n} \in FV(p, c, q) \land \forall_i \operatorname{modified}_{c}(v_{i}) \implies \operatorname{variable}(e_{i}) \land \forall_{j \neq i} e_{i} \notin \operatorname{FV}(e_{j})
$$

---

# End of Part 1

[.footer: Gist: Time to ask questions.]

^ Tutaj warto powiedzieć, że główne osiągnięcie tej pierwszej części to stworzenie logiki która jest sound. Czyli każdy poprawny dowód to faktycznie poprawny semantycznie program.

---

# Separation Logic in Concurrency

^ Teraz przejdźmy do części za którą dana została nagroda. Części dotyczącej programów współbieżnych.
A mianowicie problemem w programach współbieżnych jest at its core synchronizacja dostępów do zasobów, np. pamięci.
Separation Logic się tu nasuwa idealnie, jako mechanizm pozwalający nam udowadniać z wzglądem na podział pamięci (zasobów) między procesy.
Taki kierunek badań był wręcz przez Raynoldsa zasugerowany w oryginalnym papierze o Separation Logic.
Ale dopiero O'Hearn i Brookes kilka lat później przedstawili siostrzane papiery opisujące to bliżej.

---

# Basic Principles

^ Postawmy teraz na początek trochę fundamentów.

---

# Basic Principles - Raciness

A program is racy if two concurrent processes attempt to access the same portion of state at the same time.

Otherwise the program is race-free.

Example of a racy program:

$$
x:=y+x \| x:=x \times z
$$

^ Tu widzimy przykład użycia operatora podwójnej pałki. Rozdziela on dwa równocześnie działające procesy.

---

# Basic Principles - Raciness

Statements which differ in their meaning in a racy program:

$$
x:=x+1; x:=x+1
$$

-

$$
x:=x+2
$$

^ Jak program jest bez race'ów to nie musimy przejmować się interferencją innych procesów w nasze działanie.
Inaczej musimy się przejmować drobnymi detalami, jak np. wciśnięcie się kogoś między dwie plus jedynki.

---

# Basic Principles - Mutual Exclusion Groups

A “mutual exclusion group” is a class of commands whose elements (or their occurrences) are required not to overlap in their executions.

$$
P(s); x:=y+x; V(s) \| P(s); x:=x \times z; V(s)
$$

^ Możemy więc łączyć sekwencje komend z różnych procesów w grupy wzajemnego wykluczenia.
Tu przypomnienie, że P to złapanie semafory, a V zwolnienie go.
Technicznie rzecz biorąc, jeśli s > 0, to P zmniejsza s o 1, w przeciwnym wypadku czeka.
V zwiększa s o jeden.
Tu przykład zrealizowania mutual exlusion groups używając semaforów.

---

# Basic Principles - Mutual Exclusion Groups

$$
\begin{aligned}
& \operatorname{\mathbf{with}} r \operatorname{\mathbf{when}} B \operatorname{\mathbf{do}} C \operatorname{\mathbf{endwith}} \\
& r - Resource \\
& B - Boolean \\
& C - Command \\
\end{aligned}
$$

$$
\begin{aligned}
& P(s) = \operatorname{\mathbf{with}} s \operatorname{\mathbf{when}} s > 0 \operatorname{\mathbf{do}} s := s - 1 \operatorname{\mathbf{endwith}} \\
& V(s) = \operatorname{\mathbf{with}} s \operatorname{\mathbf{when}} \mathbf{true} \operatorname{\mathbf{do}} s := s + 1 \operatorname{\mathbf{endwith}} \\
\end{aligned}
$$

[.footer: Gist: Implementing semaphores in terms of **with when do endwith** blocks.]

^ My oprzemy się głównie na innym prymitywie synchronizacji, bloku with-when-do.
Z których tylko jeden jednocześnie może mieć with'a na danym resoursie, i tylko wtedy kiedy B jest spełnione.
Poniżej jak możemy zaimplementować podniesienie semaforu.

---

# Basic Principles - Daringness

A program is cautious if, whenever concurrent processes access the same piece of state, they do so only within commands from the same mutual exclusion group. Otherwise, the program is daring.

^ Tu warto zaznaczyć różnicę między racy a daring. 
Racy program faktycznie używa danych zmiennych z wielu procesów jednocześnie.
Daring program używa danych zmiennych z wielu procesów bez używania grup wzajemnego wykluczenia, ale robi to w sposób zdefiniowany, nie jednoczesny.

---

# Basic Principles - Daringness

$$
\begin{aligned}
& \operatorname{\mathbf{semaphore}} free := 1 ; busy := 0 \\
& \begin{aligned}
& \operatorname{P}(free); & & & & \operatorname{P}(busy); \\
& [10] := m; & \| & & & n := [10]; \\
& \operatorname{V}(busy); & & & & \operatorname{V}(free); \\
& \end{aligned}
\end{aligned}
$$

[.footer: Gist: Accessing *m* without a mutual exclusion group.]

^ Warto też zwrócić uwage, że nie jest łatwo wykryć np. kompilatorem co jest grupą wzajemnego wykluczenia,
bo np. P i V na różnych semaforach nie muszą tworzyć bloków, tak jak wcześniej.
Ten program jest daring, chociaż synchronizuje (wysyła sobie wiadomości) semaforami.
My będziemy chcieli mieć możliwość dowodzenia poprawności programów, które są darind.
Tj. takich jak ten.
Opisz co robi program. (może być w while'u)
Inny dobry przykład to memory manager, któremu pamięć jest zwracana, ale pamiętamy wskaźnik do tej pamięci.

---

# Basic Principles - Ownership Hypothesis

A code fragment can access only those portions of state that it owns.

^ Kontynuując suche definicje.
To jest dla jakiejś sensownej definicji ownershipu.
W praktyce będziemy za to postrzegać ten fragment kodu, który ma adres po lewej stronie strzałeczki w asercji.

---

# Basic Principles - Separation Property

At any time, the state can be partitioned into that owned by each process and each grouping of mutual exclusion.

^ I już ostatnia definicja.
Rozbijamy ten ownership między procesy i grupy wzajemnego wykluczenia. Czyli np. semafor może posiadać stan.

---

# A Simple Proof

$$
\begin{aligned}
& \operatorname{\mathbf{semaphore}} free := 1 ; busy := 0 \\
& \begin{aligned}
& \operatorname{P}(free); & & & & \operatorname{P}(busy); \\
& [10] := m; & \| & & & n := [10]; \\
& \operatorname{V}(busy); & & & & \operatorname{V}(free); \\
& \end{aligned}
\end{aligned}
$$

^ Wróćmy teraz do naszego programu przykładowego

---

# A Simple Proof - The Idea

$$
\begin{aligned}
& \operatorname{\mathbf{semaphore}} free := 1 ; busy := 0 \\
& \begin{aligned}
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \operatorname{P}(free); & & & & \operatorname{P}(busy); \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& [10] := m; & \| & & & n := [10]; \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& \operatorname{V}(busy); & & & & \operatorname{V}(free); \\
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \end{aligned}
\end{aligned}
$$

[.footer: Gist: Ownership flows in and out of the semaphores.]

^ Idea jest żeby ownership się układał w ten sposób.
Wyciągamy ownership z semafory, zmieniamy i zwracamy do drugiej semafory.
Z niej wyciąga to drugi proces i operuje na tym, ostatecznie zwracając do pierwszej.
Adres 10 zawsze należy do jednego z: proces z lewej, semafora, proces z prawej.
Tu warto zauważyć, że udowadniamy tylko bezpieczeństwo, nie pełną poprawność.

---

# A Little Reminder

$$
P * Q
$$

and

$$
\begin{aligned}
\{P\}[10] := 42\{Q\} \\
P \implies 10 \mapsto – * \operatorname{true} \\
\end{aligned}
$$

^ Małe przypomnienie.
Pierwsze znaczy, że możemy rozdzielić Heap na dwie części, z których jedna spełnia P, a druga Q.
W drugim ważne jest, że jeśli komenda modyfikuje adres 10, to P musi być implikować poprawność tego adresu, że jest on zaalokowany.
Tu ważne jest, że gwiazdka separuje 10 -> - i true.

---

# Disjoint Concurrency Rule

$$
\frac{\{P\} C\{Q\} \quad\left\{P^{\prime}\right\} C^{\prime}\left\{Q^{\prime}\right\}}{\left\{P * P^{\prime}\right\} C \| C^{\prime}\left\{Q * Q^{\prime}\right\}}
$$

[.footer: Gist: Two non-interfering processes should be able to run concurrently.]

^ To ma sens. Oba, C i C' posiadają swoje adresy, więc operują na róznych. Także można je opisać separującą koniunkcją. 

---

# Getting back to Mergesort

$$
\begin{aligned}
& \{\operatorname{\textit{array}}(a, i, j)\} \\
& \operatorname{procedure} \mathrm{ms}(a, i, j) \\ 
& \operatorname{newvar} m:=(i+j) / 2 \\
& \operatorname{if} i<j \operatorname{then} \\
& \text{ }\text{ } \{\operatorname{\textit{array}}(a, i, j)\} \\
& \text{ }\text{ } \{\operatorname{\textit{array}}(a, i, m) \land \operatorname{\textit{array}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{ms}(a, i, m) \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, m) \land \operatorname{\textit{array}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{ms}(a, m+1, j) \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, m) \land \operatorname{\textit{sorted}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{merge}(a, i, m+1, j) \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, j)\} \\
& \{\operatorname{\textit{sorted}}(a, i, j)\} \\
\end{aligned}
$$

^ Przypomnijmy sobie nasz mergesort.
Teraz przekształćmy go w wersję współbieżną.

---

# Concurrent Mergesort - Array Definition

$$
array(a,i,j) := \forall_{k} i \leq k \leq j \implies a + k \mapsto - * \mathbf{true}
$$

^ Czyli teraz asercja arraya posiada adresy od a+i do a+j.
Będziemy dzielić tablicę między wiele wątków, stąd musieliśmy zmienić tą definicję.

---

# Concurrent Mergesort

$$
\begin{aligned}
& \{\operatorname{\textit{array}}(a, i, j)\} \\
& \operatorname{procedure} \mathrm{ms}(a, i, j) \\ 
& \operatorname{newvar} m:=(i+j) / 2 \\
& \operatorname{if} i<j \operatorname{then} \\
& \text{ }\text{ } \{\operatorname{\textit{array}}(a, i, j)\} \\
& \text{ }\text{ } \{\operatorname{\textit{array}}(a, i, m) * \operatorname{\textit{array}}(a, m+1, j)\} \\
& \begin{aligned} \\
& \text{ }\text{ } \{\operatorname{\textit{array}}(a, i, m)\} & & & & \{\operatorname{\textit{array}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{ms}(a, i, m); & \| & & & \operatorname{ms}(a, m+1, j); \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, m)\} & & & & \{\operatorname{\textit{sorted}}(a, m+1, j)\} \\
& \end{aligned} \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, m) * \operatorname{\textit{sorted}}(a, m+1, j)\} \\
& \text{ }\text{ } \operatorname{merge}(a, i, m+1, j) \\
& \text{ }\text{ } \{\operatorname{\textit{sorted}}(a, i, j)\} \\
& \{\operatorname{\textit{sorted}}(a, i, j)\} \\
\end{aligned}
$$

^ Tu widzimy jak łądnie możemy rozdzielić te tablice, oddać je różnym funkcjom, i potem znów połączyć.

---

# Resources - Now Better

$$
\operatorname{\mathbf{with}} r \operatorname{\mathbf{when}} B \operatorname{\mathbf{do}} C \operatorname{\mathbf{endwith}}
$$

New Program Form:

$$
\begin{aligned}
& init; \\
& \operatorname{\mathbf{resource}} r_1(\text{variable list}), \dots, r_m(\text{variable list}) \\
& C_1 \| \dots \| C_n \\
\end{aligned}
$$

^ Mieliśmy wcześniej pewien konstrukt, resourcu, na którym oparliśmy semafory.
Teraz dopakujemy te resourcy zmiennymi składowymi, i będziemy rozważać bardzo ustrukturyzowaną postać programu...

---

# Concurrent Program - New Proof Form

$$RI$$ - Resource Invariant

$$
\begin{aligned}
& \{P\} \\
& init; \\
& \{P' * RI_{r_1} * \dots * RI_{r_m}\} \\
& \operatorname{\mathbf{resource}} r_1(\text{variable list}), \dots, r_m(\text{variable list}) \\
& C_1 \| \dots \| C_n \\
& \{Q * RI_{r_1} * \dots * RI_{r_m}\} \\
\end{aligned}
$$

^ Dorzucamy tzw. resource invarianty do każdego resourca. Co ważne, one normalnie mogą posiadać adres, wtedy możemy powiedzieć, że resource ownuje adres.

---

# Getting Back to Semaphore Message Sending

$$
\begin{aligned}
& \operatorname{\mathbf{semaphore}} free := 1 ; busy := 0 \\
& \begin{aligned}
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \operatorname{P}(free); & & & & \operatorname{P}(busy); \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& [10] := m; & \| & & & n := [10]; \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& \operatorname{V}(busy); & & & & \operatorname{V}(free); \\
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \end{aligned}
\end{aligned}
$$

^ Wróćmy do naszego daring przykładu z przesyłaniem wiadomości poprzez blokowanie i zwalnianie semafor.

---

# Message Sending - Rewrite Definitions

$$
\begin{aligned}
& free := 1, busy := 0; \\
& \operatorname{\mathbf{resource}} free(free), busy(busy); \\
& \begin{aligned}
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \operatorname{P}(free); & & & & \operatorname{P}(busy); \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& [10] := m; & \| & & & n := [10]; \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& \operatorname{V}(busy); & & & & \operatorname{V}(free); \\
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \end{aligned}
\end{aligned}
$$

[.footer: Gist: We changed semaphores to resources.]

^ Przemieńmy deklaracje resourców na nowe.

---

# Message Sending - Add Definitions

$$
\begin{aligned}
& \{ 10 \mapsto - \} \\
& free := 1, busy := 0; \\
& \{ (free = 1 \land 10 \mapsto -) * (busy = 0 \land \mathbf{emp}) \} \\
& \{ RI_{free} * RI_{busy} * \mathbf{emp} * \mathbf{emp} \} \\
& \operatorname{\mathbf{resource}} free(free), busy(busy); \\
& \{ \mathbf{emp} * \mathbf{emp} \} \\
& \begin{aligned}
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \operatorname{P}(free); & & & & \operatorname{P}(busy); \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& [10] := m; & \| & & & n := [10]; \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& \operatorname{V}(busy); & & & & \operatorname{V}(free); \\
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \end{aligned}
\end{aligned}
$$

^ Dodajmy warunki.
Tzn. dodajemy pewne (niezdefiniowane jeszcze) Resource Invariants, przed uruchomieniem procesów "przekazujemy" te invarianty resourcom i zostajemy z emp'ami dla procesów.

---

# Message Sending - Resource Invariants

$$
RI_s = (s = 0 \land emp) \lor (s = 1 \land 10 \mapsto –)
$$

$$
\begin{aligned}
& \{ 10 \mapsto - \} \\
& free := 1, busy := 0; \\
& \{ (free = 1 \land 10 \mapsto -) * (busy = 0 \land \mathbf{emp}) \} \\
& \{ RI_{free} * RI_{busy} * \mathbf{emp} * \mathbf{emp} \} \\
& \operatorname{\mathbf{resource}} free(free), busy(busy); \\
& \{ \mathbf{emp} * \mathbf{emp} \} \\
& \begin{aligned}
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \operatorname{P}(free); & & & & \operatorname{P}(busy); \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& [10] := m; & \| & & & n := [10]; \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& \operatorname{V}(busy); & & & & \operatorname{V}(free); \\
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \end{aligned}
\end{aligned}
$$

^ Definiujemy invarianty. Opisują one dobrze kiedy semafora coś posiada, a kiedy nie.

---

# Resource Invariant Deep Dive

The command C has access to $$RI_{r}$$

$$
\frac{\left\{\left(P * R I_{r}\right) \land B\right\} C\left\{Q * R I_{r}\right\}}{\{P\} \operatorname{\mathbf { with }} r \operatorname{\mathbf { when }} B \operatorname{\mathbf { do }} C \operatorname{\mathbf { endwith }\{}Q\}}
$$

^ Co tu jest ważne? Że w "with" bloku komenda "przejmuje" resource invariant

---

# Resource Invariant Deep Dive - P(free)

$$
RI_s = (s = 0 \land emp) \lor (s = 1 \land 10 \mapsto –)
$$

$$
P(free) = \operatorname{\mathbf{with}} free \operatorname{\mathbf{when}} free > 0 \operatorname{\mathbf{do}} free := free - 1 \operatorname{\mathbf{endwith}}
$$

$$
\begin{aligned}
& \{(P * RI_{free}) \land B\} \\
& \{(\mathbf{emp} *(( free=0 \land \mathbf{emp}) \lor( free=1 \land 10 \mapsto-))) \land  free>0\} \\
& \{ free=1 \land 10 \mapsto-\} \\
&  free:= free-1 \\
& \{ free=0 \land 10 \mapsto-\} \\
& \{10 \mapsto-*( free=0 \land \mathbf{emp})\} \\
& \{10 \mapsto-*(( free=0 \land \mathbf{emp}) \lor( free=1 \land 10 \mapsto-))\} \\
& \{Q * RI_{free}\} \\
\end{aligned}
$$

^ Tutaj krok po kroku widzimy jak wygląda body brania semafory i przekaz własności.
Zostawiamy semafore z wartością 0 i bez dziesiątki.

---

# Resource Invariant Deep Dive - V(free)

$$
RI_s = (s = 0 \land emp) \lor (s = 1 \land 10 \mapsto –)
$$

$$
V(free) = \operatorname{\mathbf{with}} free \operatorname{\mathbf{when}} \mathbf{true} \operatorname{\mathbf{do}} free := free + 1 \operatorname{\mathbf{endwith}}
$$

$$
\begin{aligned}
& \{(P * RI_{free}) \land B\} \\
& \{10 \mapsto-*((free=0 \land \text { emp }) \lor ( free=1 \land 10 \mapsto-))\} \\
& \{10 \mapsto-*( free=0 \land \text { emp })\} \\
&  free:= free+1 \\
& \{10 \mapsto-*( free=1 \land \text { emp })\} \\
& \{\text { emp } *( free=1 \land 10 \mapsto-)\} \\
& \{\text { emp } *(( free=0 \land \text { emp }) \lor ( free=1 \land 10 \mapsto-))\} \\
& \{Q * RI_{free}\} \\
\end{aligned}
$$

^ Tutaj krok po kroku widzimy jak wygląda body opuszczania semafory i przekaz własności.
Zostawiamy semafore z wartością 1 i z dziesiątką.

---

# Message Sending

$$
RI_s = (s = 0 \land emp) \lor (s = 1 \land 10 \mapsto –)
$$

$$
\begin{aligned}
& \{ 10 \mapsto - \} \\
& free := 1, busy := 0; \\
& \{ (free = 1 \land 10 \mapsto -) * (busy = 0 \land \mathbf{emp}) \} \\
& \{ RI_{free} * RI_{busy} * \mathbf{emp} * \mathbf{emp} \} \\
& \operatorname{\mathbf{resource}} free(free), busy(busy); \\
& \{ \mathbf{emp} * \mathbf{emp} \} \\
& \begin{aligned}
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \operatorname{P}(free); & & & & \operatorname{P}(busy); \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& [10] := m; & \| & & & n := [10]; \\
& \{ 10 \mapsto - \} & & & & \{ 10 \mapsto - \} \\
& \operatorname{V}(busy); & & & & \operatorname{V}(free); \\
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \end{aligned}
\end{aligned}
$$

^ Tu jeszcze raz możemy w praktyce zobaczyć, co się stało.
Lewy proces będąc w while'u mógłby znów wyjąć z free wskaźnik, znów do niego coś włożyć i znów przekazać prawemu.
Tu warto zauważyć, że nie musieliśmy mieć globalnego predykatu, który opisuje zależność między oboma semaforami, możemy o nich rozumować lokalnie.

---

# Memory Manager

$$
\begin{aligned}
&\operatorname { alloc }(x, a, b) \triangleq \operatorname{\mathbf{with}} mm \operatorname{\mathbf{when}} \operatorname{\mathbf{true}} \operatorname{\mathbf{do}} \\
& \text{ }\text{ }\text{ }\text{ } \operatorname{\mathbf{if}} f=\mathrm{nil} \operatorname{\mathbf{then}} x:=\operatorname{cons}(a, b) \\
& \text{ }\text{ }\text{ }\text{ } \operatorname{\mathbf{else}} x:=f ; f:=x .2 ; x .1:=a ; x .2:=b
\end{aligned}
$$

$$
\begin{aligned}
&\operatorname { dealloc }(y) \triangleq \operatorname{\mathbf{with}} mm \operatorname{\mathbf{when}} \operatorname{\mathbf{true}} \operatorname{\mathbf{do}} \\
& \text{ }\text{ }\text{ }\text{ } y.2:=f \\
& \text{ }\text{ }\text{ }\text{ } f:=y \\
\end{aligned}
$$

^ Spójrzmy na jeszcze jeden przykład użycia. Wspomniany wcześniej memory manager, czyli alloc i free.
Używają listy par a, b zdealokowanych, tak żeby znów ich użyć w przyszłej alokacji.

---

# Memory Manager - alloc

$$
\begin{aligned}
& \{\mathbf{emp}\}\operatorname{alloc}(x,a,b)\{x \mapsto a, b\}: \\
& \{\mathbf{emp} * \operatorname{\mathit{list}} f\} \\
& \{\operatorname{\mathit{list}} f\} \\
& \text{ }\text{ } \operatorname{\mathbf{if}} f=\mathrm{nil} \operatorname{\mathbf{then}} \\
& \text{ }\text{ }\text{ }\text{ } \{\operatorname{\mathit{list}} f \land f = nil\} \\
& \text{ }\text{ }\text{ }\text{ } \{f = nil \land \mathbf{emp}\} \\
& \text{ }\text{ }\text{ }\text{ } x:=\operatorname{cons}(a, b) \\
& \text{ }\text{ }\text{ }\text{ } \{(x \mapsto a, b) * (f = nil \land \mathbf{emp})\} \\
& \text{ }\text{ }\text{ }\text{ } \{(x \mapsto a, b) * \operatorname{\mathit{list}} f\} \\
& \text{ }\text{ } \operatorname{\mathbf{else}} \\
& \text{ }\text{ }\text{ }\text{ } \{\operatorname{\mathit{list}} f \land f \neq nil\} \\
& \text{ }\text{ }\text{ }\text{ } \{\exists_{y} f \mapsto -, y * \operatorname{\mathit{list}} y\} \\
& \text{ }\text{ }\text{ }\text{ } x:=f ; f:=x .2 ; x .1:=a ; x .2:=b \\
& \text{ }\text{ }\text{ }\text{ } \{(x \mapsto a, b) * \operatorname{\mathit{list}} f\} \\
& \{(x \mapsto a, b) * \operatorname{\mathit{list}} f\} \\
\end{aligned}
$$

[.footer: Gist: Correctness of alloc. It uses a list f to store deallocated memory nodes.]

^ Przypomnijmy, że list f wyciągamy z resourca mm blokiem with, dlatego mamy dostęp dotego predykatu..

---

# Memory Manager - dealloc

$$
\begin{aligned}
& \{y \mapsto -, -\}\operatorname{dealloc}(y)\{\mathbf{emp}\}: \\
& \{y \mapsto -, - * \operatorname{\mathit{list}} f\} \\
& y.2 := f; \\
& f := y; \\
& \{\operatorname{\mathit{list}} f\} \\
& \{\mathbf{emp} * \operatorname{\mathit{list}} f\} \\
\end{aligned}
$$

[.footer: Gist: Correctness of dealloc. It uses a list f to store deallocated memory nodes.]

---

# Memory Manager - usage

$$
\begin{aligned}
& \{ \mathbf{emp} * \mathbf{emp} \} \\
& \begin{aligned}
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \operatorname{alloc}(x,a,b); & & & & \operatorname{alloc}(y, a', b'); \\
& \{ x \mapsto a, b \} & & & & \{ y \mapsto a', b' \} \\
& x.1 := 4; & \| & & & y.1 := y; \\
& \{ x \mapsto 4, b \} & & & & \{ y \mapsto 7, b' \} \\
& \operatorname{dealloc}(x); & & & & \operatorname{dealloc}(y); \\
& \{\mathbf{emp}\} & & & & \{\mathbf{emp}\} \\
& \{ \mathbf{emp} * \mathbf{emp} \} \\
& \{ \mathbf{emp} \} \\
& \end{aligned}
\end{aligned}
$$

[.footer: Gist: Allocation can reuse memory deallocated by the other process and use it with no limits. A process could use memory after deallocation. This is daring, but provably correct.]

^ W zależności od przeplotu jeden proces może z alokacji otrzymać adres zwrócony przez drugi i używać go bez ograniczeń. To jest przykład daring programu, bo pierwszy mógłby zachować zmienną z tym adresem i odnieść się do oddanej pamięci.
Nasze zasady wnioskowania nie pozwalają na udowodnienie niepoprawnego w ten sposób kodu, poprzez asercję emp.

---

# End of Part 2

[.footer: Gist: Time to ask questions.]

^ Tutaj warto powiedzieć, że główne osiągnięcie tej pierwszej części to stworzenie systemu który jest modularny. Możemy udowadniać zachowania naszych programów koncentrując się na aktualnej funkcji i procesie.
Co więcej, rozwiązania bazujące na tym zostały z sukcesem zaimplementowane.
Autorami tych dwóch papierów ("Resources, Concurrency and Local Reasoning" i "A semantics for concurrent separation logic") są Peter W. O’Hearn i Stephen Brookes.

---

# Papers

- Separation Logic: A Logic for Shared Mutable Data Structures
    - John C. Reynolds
- Resources, Concurrency and Local Reasoning
    - Peter W. O’Hearn
- A semantics for concurrent separation logic
    - Stephen Brookes
