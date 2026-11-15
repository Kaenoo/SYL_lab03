# Laboratoire de systèmes logiques : ADD/SUB, ALU et comparateur
**Auteurs :** Kaeno Eyer & Omar Egal Ahmed 
**Date de rendu :** 25 novembre 2026 

---

## 2. Implémentation de la partie Arithmétique

### 2.3 Implémentation du bloc Calculation

#### QUESTION 1 : Analysez et décrivez le comportement du bit de carry en sortie lors d'une opération de soustraction, en vous basant uniquement sur vos observations, sans apporter de modifications au circuit.

Lors d'une opération de soustraction ($op\_i = 1$), l'opération $A - B$ est réalisée par l'additionneur 4 bits en additionnant $A$ avec le complément à 2 de $B$, soit $A + \overline{B} + 1$. 

On observe que le bit de retenue en sortie (`carry_o`) se comporte comme un **indicateur d'emprunt inversé** ($\overline{\text{borrow}}$) en arithmétique non signée :
* **Lorsque $A \ge B$ (non signé) :** la soustraction ne nécessite aucun emprunt et le bit **`carry_o` vaut `1`**.
* **Lorsque $A < B$ (non signé) :** le résultat déborde en dessous de zéro (un emprunt est nécessaire) et le bit **`carry_o` vaut `0`**.

---

#### QUESTION 2 : Déduisez quelle est la taille minimale de l'entrée `op_i` du circuit Calculation pour réaliser toutes les opérations définies dans le tableau 5.

La taille minimale de l'entrée `op_i` du circuit `Calculation` est de **1 bit**.

**Justification :**  
En représentation binaire (complément à 2), le calcul matériel de l'addition ($A + B$) et de la soustraction ($A - B$) est strictement identique pour les nombres signés et non signés. En observant le Tableau 5 des opcodes (`op_i[5:0]`) :
* $A + B \text{ (signé)} = \text{00\textbf{0}0}\phi\phi$ et $A + B \text{ (non signé)} = \text{00\textbf{0}1}\phi\phi$
* $A - B \text{ (signé)} = \text{00\textbf{1}0}\phi\phi$ et $A - B \text{ (non signé)} = \text{00\textbf{1}1}\phi\phi$

Seul le bit **`op_i[3]`** est nécessaire à l'intérieur de `Calculation` pour choisir entre l'addition (`op_i[3] = 0`) et la soustraction (`op_i[3] = 1`). La distinction entre signé et non signé (`op_i[2]`) ne modifie pas le calcul lui-même mais servira uniquement dans le bloc de gestion des sorties (`OutputHandling`) pour déterminer quel indicateur (`overflow_o` ou `carry_o`) génère une erreur.

---

#### QUESTION 3 : En vous appuyant sur la table de vérité ci-dessous, obtenez l'équation simplifiée de la sortie `overflow`, puis intégrez la gestion de l'indicateur dans le composant Calculation.

En arithmétique signée (complément à 2 sur 4 bits), le signe d'un nombre est défini par son bit de poids fort (bit 3) :
* Nombre positif ou nul ($\ge 0$) $\iff \text{bit } 3 = 0$
* Nombre strictement négatif ($< 0$) $\iff \text{bit } 3 = 1$

Un dépassement de capacité signé (`overflow_o = 1`) se produit uniquement dans deux situations :
1. **En addition ($A + B$, soit $op = 0$) :** lorsqu'on additionne deux opérandes de **même signe** et que le résultat $R\_o$ possède le **signe opposé**.
2. **En soustraction ($A - B$, soit $op = 1$) :** lorsqu'on soustrait deux opérandes de **signes opposés** et que le résultat $R\_o$ est du **signe opposé à $A$**.

##### Table de vérité complète de la sortie `overflow_o` (Table 6) :

| Opération ($op$) | $A$ ($a_3$) | $B$ ($b_3$) | $R\_o$ ($r_3$) | `overflow_o` |
| :---: | :---: | :---: | :---: | :---: |
| $A + B$ ($0$) | $\ge 0$ ($0$) | $\ge 0$ ($0$) | $\ge 0$ ($0$) | **0** |
| $A + B$ ($0$) | $\ge 0$ ($0$) | $\ge 0$ ($0$) | $< 0$ ($1$) | **1** |
| $A + B$ ($0$) | $\ge 0$ ($0$) | $< 0$ ($1$) | $\ge 0$ ($0$) | **0** |
| $A + B$ ($0$) | $\ge 0$ ($0$) | $< 0$ ($1$) | $< 0$ ($1$) | **0** |
| $A + B$ ($0$) | $< 0$ ($1$) | $\ge 0$ ($0$) | $\ge 0$ ($0$) | **0** |
| $A + B$ ($0$) | $< 0$ ($1$) | $\ge 0$ ($0$) | $< 0$ ($1$) | **0** |
| $A + B$ ($0$) | $< 0$ ($1$) | $< 0$ ($1$) | $\ge 0$ ($0$) | **1** |
| $A + B$ ($0$) | $< 0$ ($1$) | $< 0$ ($1$) | $< 0$ ($1$) | **0** |
| $A - B$ ($1$) | $\ge 0$ ($0$) | $\ge 0$ ($0$) | $\ge 0$ ($0$) | **0** |
| $A - B$ ($1$) | $\ge 0$ ($0$) | $\ge 0$ ($0$) | $< 0$ ($1$) | **0** |
| $A - B$ ($1$) | $\ge 0$ ($0$) | $< 0$ ($1$) | $\ge 0$ ($0$) | **0** |
| $A - B$ ($1$) | $\ge 0$ ($0$) | $< 0$ ($1$) | $< 0$ ($1$) | **1** |
| $A - B$ ($1$) | $< 0$ ($1$) | $\ge 0$ ($0$) | $\ge 0$ ($0$) | **1** |
| $A - B$ ($1$) | $< 0$ ($1$) | $\ge 0$ ($0$) | $< 0$ ($1$) | **0** |
| $A - B$ ($1$) | $< 0$ ($1$) | $< 0$ ($1$) | $\ge 0$ ($0$) | **0** |
| $A - B$ ($1$) | $< 0$ ($1$) | $< 0$ ($1$) | $< 0$ ($1$) | **0** |

##### Obtention de l'équation logique simplifiée :

En extrayant les 4 mintermes où `overflow_o = 1` (avec $op = op\_i[3]$, $a_3 = a\_i[3]$, $b_3 = b\_i[3]$ et $r_3 = r\_o[3]$) :

$$overflow\_o = (\overline{op} \cdot \overline{a_3} \cdot \overline{b_3} \cdot r_3) + (\overline{op} \cdot a_3 \cdot b_3 \cdot \overline{r_3}) + (op \cdot \overline{a_3} \cdot b_3 \cdot r_3) + (op \cdot a_3 \cdot \overline{b_3} \cdot \overline{r_3})$$

En regroupant les termes selon $\overline{a_3} \cdot r_3$ d'une part, et $a_3 \cdot \overline{r_3}$ d'autre part :

$$overflow\_o = \overline{a_3} \cdot r_3 \cdot (\overline{op} \cdot \overline{b_3} + op \cdot b_3) + a_3 \cdot \overline{r_3} \cdot (\overline{op} \cdot b_3 + op \cdot \overline{b_3})$$

En reconnaissant la fonction OU Exclusif ($b_3 \oplus op = \overline{op} \cdot b_3 + op \cdot \overline{b_3}$) et son inverse ($\overline{b_3 \oplus op} = \overline{op} \cdot \overline{b_3} + op \cdot b_3$) :

$$overflow\_o = \overline{a_3} \cdot \overline{(b_3 \oplus op)} \cdot r_3 + a_3 \cdot (b_3 \oplus op) \cdot \overline{r_3}$$

---

### 2.4 Implémentation du comparateur

#### QUESTION 4 : De combien de bits l'opcode du comparateur a-t-il besoin au minimum pour pouvoir effectuer toutes ses opérations ?

Le comparateur doit pouvoir réaliser **6 opérations** distinctes (Tableau 8).  
Comme $2^2 = 4 < 6 \le 2^3 = 8$, l'opcode du comparateur (`op_i`) a besoin au minimum de **3 bits** (correspondant aux bits de poids faible `op_i[2:0]` de l'opcode global de l'ALU).

---

#### QUESTION 5 : Le comparateur tel qu'implémenté récupère plus d'informations que nécessaire pour certaines comparaisons. Identifiez quelles entrées peuvent être ignorées et expliquez pourquoi.

Les entrées **`a_i[3:0]`** et **`b_i[3:0]`** peuvent être totalement ignorées (et connectées au composant *Do not connect* dans Logisim).

**Explication :**  
Lors d'une opération de comparaison (opcodes `011xxx`), le bit `op_i[3]` vaut toujours `1`, ce qui commande au bloc `Calculation` d'effectuer la soustraction **$A - B$**. Les quatre drapeaux d'état **`zero_i` ($Z$)**, **`carry_i` ($C$)**, **`negative_i` ($N$)** et **`overflow_i` ($V$)** générés par cette soustraction contiennent toute l'information nécessaire et suffisante pour déterminer la relation d'ordre (signée ou non signée) et d'égalité entre $A$ et $B$, sans jamais avoir besoin de relire les bits des opérandes `a_i` et `b_i`.

---

#### QUESTION 6 : Donnez les six équations de la sortie `cmp_o` pour chaque opération.

À partir des indicateurs issus de la soustraction $A - B$ ($Z = zero\_i$, $C = carry\_i$, $N = negative\_i$, $V = overflow\_i$), voici les 6 équations logiques de `cmp_o` selon la valeur de `op_i[2:0]` :

1. **$A > B$ (signé) — Opcode `011001` (`op_i[2:0] = 001`) :**  
   $A$ est strictement supérieur à $B$ en signé si le résultat n'est pas nul ($Z = 0$) et que le signe du résultat est cohérent avec l'absence de dépassement ($N = V$, soit $\overline{N \oplus V}$) :
   $$cmp\_o = \overline{zero\_i} \cdot \overline{(negative\_i \oplus overflow\_i)}$$
   *(Note : dans la Table 2 p. 4, cette ligne est notée $A \ge B \text{ (signé)}$, dont l'équation serait $\overline{negative\_i \oplus overflow\_i}$, tandis que la Table 8 p. 8 demande $A > B \text{ (signé)}$).*

2. **$A < B$ (signé) — Opcode `011010` (`op_i[2:0] = 010`) :**  
   $A$ est strictement inférieur à $B$ en signé si le résultat est négatif sans overflow ($N=1, V=0$), ou positif à cause d'un overflow ($N=0, V=1$) :
   $$cmp\_o = negative\_i \oplus overflow\_i$$

3. **$A \neq B$ — Opcode `011011` (`op_i[2:0] = 011`) :**  
   $A$ et $B$ sont différents si le résultat de $A - B$ n'est pas égal à zéro :
   $$cmp\_o = \overline{zero\_i}$$

4. **$A = B$ — Opcode `011100` (`op_i[2:0] = 100`) :**  
   $A$ et $B$ sont égaux si le résultat de $A - B$ vaut zéro :
   $$cmp\_o = zero\_i$$

5. **$A \ge B$ (non signé) — Opcode `011101` (`op_i[2:0] = 101`) :**  
   Comme observé à la Question 1, lors de la soustraction $A - B$, il n'y a pas d'emprunt lorsque $A \ge B$, ce qui se traduit par une retenue de sortie à `1` :
   $$cmp\_o = carry\_i$$

6. **$A < B$ (non signé) — Opcode `011110` (`op_i[2:0] = 110`) :**  
   Il y a un emprunt lors de la soustraction $A - B$, donc la retenue de sortie vaut `0` :
   $$cmp\_o = \overline{carry\_i}$$

---

### 2.5 Arithmetic Selection

#### QUESTION 7 : De combien de bits l'opcode de sélection a-t-il besoin au minimum pour pouvoir sélectionner correctement les opérations arithmétiques ?

L'opcode du bloc `Arithmetic_selection` a besoin au minimum de **1 bit**.

**Justification :**  
Le rôle du bloc `Arithmetic_selection` est uniquement de multiplexer entre deux sources :
* La sortie `calculation_i[3:0]` issue du bloc `Calculation` (opcodes de la forme `00xxxx`).
* La sortie `comp_i` (étendue sur 4 bits avec `000` sur les bits de poids fort) issue du bloc `Comp` (opcodes de la forme `011xxx`).

Un seul bit de sélection suffit donc pour commander un multiplexeur à 2 entrées : il s'agit du bit **`op_i[4]`** de l'opcode global (`0` sélectionne `Calculation`, `1` sélectionne `Comp`).