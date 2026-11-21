# Laboratoire de systèmes logiques : ADD/SUB, ALU et comparateur
**Auteurs :** Kaeno Eyer & Omar Egal Ahmed 
**Date de rendu :** 25 novembre 2026 

---

## 2. Implémentation de la partie Arithmétique

### 2.3 Implémentation du bloc Calculation

#### QUESTION 1 : Analysez et décrivez le comportement du bit de carry en sortie lors d'une opération de soustraction, en vous basant uniquement sur vos observations, sans apporter de modifications au circuit.

Lors d'une opération de soustraction ($op\_i = 1$), l'opération $A - B$ est réalisée par l'additionneur 4 bits en additionnant $A$ avec le complément à 2 de $B$, soit $A + (\overline{B} + 1)$. 

Lors d'une soustraction non signée ($A − B$), le bit carry_o fonctionne à l'inverse d'une retenue classique :
* **Lorsque $A \ge B$ (non signé) :** le résultat est positif ou nul (il n'y a pas d'emprunt), donc le bit **`carry_o` vaut `1`**.
* **Lorsque $A < B$ (non signé) :** le résultat déborde en dessous de zéro (un emprunt est nécessaire) et le bit **`carry_o` vaut `0`**.

---

#### QUESTION 2 : Déduisez quelle est la taille minimale de l'entrée `op_i` du circuit Calculation pour réaliser toutes les opérations définies dans le tableau 5.

La taille minimale de l'entrée `op_i` du circuit `Calculation` est de **1 bit**.

**Justification :**  
En représentation binaire (complément à 2), le calcul de l'addition ($A + B$) et de la soustraction ($A - B$) est strictement identique pour les nombres signés et non signés. En observant le Tableau 5 des opcodes (`op_i[5:0]`) :
* $A + B \text{ (signé)} = \text{00\textbf{0}0}\phi\phi$ et $A + B \text{ (non signé)} = \text{00\textbf{0}1}\phi\phi$
* $A - B \text{ (signé)} = \text{00\textbf{1}0}\phi\phi$ et $A - B \text{ (non signé)} = \text{00\textbf{1}1}\phi\phi$

Seul le bit **`op_i[3]`** est nécessaire à l'intérieur de `Calculation` pour choisir entre l'addition (`op_i[3] = 0`) et la soustraction (`op_i[3] = 1`). La distinction entre signé et non signé (`op_i[2]`) ne modifie pas le calcul lui-même mais servira uniquement dans le bloc des sorties (`OutputHandling`) pour déterminer quel indicateur (`overflow_o` ou `carry_o`) génère une erreur.

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

Avec les 4 mintermes où `overflow_o = 1` (avec $op = op\_i[3]$, $a_3 = a\_i[3]$, $b_3 = b\_i[3]$ et $r_3 = r\_o[3]$) :

$$overflow\_o = (\overline{op} \cdot \overline{a_3} \cdot \overline{b_3} \cdot r_3) + (\overline{op} \cdot a_3 \cdot b_3 \cdot \overline{r_3}) + (op \cdot \overline{a_3} \cdot b_3 \cdot r_3) + (op \cdot a_3 \cdot \overline{b_3} \cdot \overline{r_3})$$

En regroupant les termes selon $\overline{a_3} \cdot r_3$ d'une part, et $a_3 \cdot \overline{r_3}$ d'autre part :

$$overflow\_o = \overline{a_3} \cdot r_3 \cdot (\overline{op} \cdot \overline{b_3} + op \cdot b_3) + a_3 \cdot \overline{r_3} \cdot (\overline{op} \cdot b_3 + op \cdot \overline{b_3})$$

En reconnaissant la fonction OU Exclusif ($b_3 \oplus op = \overline{op} \cdot b_3 + op \cdot \overline{b_3}$) et son inverse ($\overline{b_3 \oplus op} = \overline{op} \cdot \overline{b_3} + op \cdot b_3$) :

$$overflow\_o = \overline{a_3} \cdot \overline{(b_3 \oplus op)} \cdot r_3 + a_3 \cdot (b_3 \oplus op) \cdot \overline{r_3}$$

---

### 2.4 Implémentation du comparateur

#### QUESTION 4 : De combien de bits l'opcode du comparateur a-t-il besoin au minimum pour pouvoir effectuer toutes ses opérations ?

Le comparateur doit pouvoir réaliser **6 opérations** distinctes (Tableau 8).  
Comme $2^2 = 4 < 6 \le 2^3 = 8$, l'opcode du comparateur (`op_i`) a besoin au minimum de **3 bits**.

---

#### QUESTION 5 : Le comparateur tel qu'implémenté récupère plus d'informations que nécessaire pour certaines comparaisons. Identifiez quelles entrées peuvent être ignorées et expliquez pourquoi.

Les entrées **`a_i[3:0]`** et **`b_i[3:0]`** peuvent être totalement ignorées (et connectées au composant *Do not connect* dans Logisim).

**Explication :**  
Lors d'une opération de comparaison (opcodes `011xxx`), le bit `op_i[3]` vaut toujours `1`, ce qui commande au bloc `Calculation` d'effectuer la soustraction **$A - B$**. Les quatre drapeaux d'état **`zero_i` ($Z$)**, **`carry_i` ($C$)**, **`negative_i` ($N$)** et **`overflow_i` ($V$)** générés par cette soustraction contiennent toute l'information nécessaire pour déterminer la relation d'ordre et d'égalité (en signée ou en non signée) entre $A$ et $B$.

---

#### QUESTION 6 : Donnez les six équations de la sortie `cmp_o` pour chaque opération.

À partir des indicateurs issus de la soustraction $A - B$ ($Z = zero\_i$, $C = carry\_i$, $N = negative\_i$, $V = overflow\_i$), voici les 6 équations logiques de `cmp_o` selon la valeur de `op_i[2:0]` :

1. **$A ≥ B$ (signé) — Opcode `011001` (`op_i[2:0] = 001`) :**  
   $A$ est strictement supérieur à $B$ en signé si le résultat n'est pas nul ($Z = 0$) et que le signe du résultat est cohérent avec l'absence de dépassement ($N = V$, soit $\overline{N \oplus V}$) :
   $$cmp\_o = \overline{(negative\_i \oplus overflow\_i)}$$

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
Le rôle du bloc `Arithmetic_selection` est uniquement de selectionner (via un multiplexer) entre deux sources :
* La sortie `calculation_i[3:0]` qui provient du bloc `Calculation` (opcodes de la forme `00xxxx`).
* La sortie `comp_i` (étendue sur 4 bits avec `000` sur les bits de poids fort) issue du bloc `Comp` (opcodes de la forme `011xxx`).

Un seul bit de sélection suffit alors pour le multiplexeur à 2 entrées : il s'agit du bit **`op_i[4]`** de l'opcode global (`0` sélectionne `Calculation`, `1` sélectionne `Comp`).

## 3.  Implémentation de la partie Logique

### 3.1 Implémentation de l’unité logique

#### QUESTION 8 : De combien de bits l’opcode de l’unité logique a-t-il besoin au minimum pour pouvoir effectuer toutes ses opérations ?

L'unité logique réalise 4 opérations distinctes (AND, OR, NAND, XOR). Comme $2^2 = 4$, l'entrée op_i a besoin au minimum de 2 bits, qui correspondent aux bits de poids faible  **`op_i[1:0]`** de l'opcode global (`00` = AND, `01` = OR, `10` = NAND, `11` = XOR).

### 3.3 Gestion des sorties

#### QUESTION 9 : Définissez le fonctionnement de la sortie `error_o` en fonction de chaque type d’opération (calculation, logique, comparaison).

La sortie `error_o` passe à **`1`** lorsqu'il y a un dépassement de capacité (résultat faux) ou lorsque l'opcode n'existe pas. Voici son comportement selon le type d'opération :

1. **Opérations de calcul (`Calculation` — opcodes `00xxxx`) :**
   * **En signé ($op\_i[2] = 0$) :** Une erreur apparaît s'il y a un dépassement signé, donc **`error_o = overflow_o`**.
   * **En non signé ($op\_i[2] = 1$) :**
     * **Addition ($op\_i[3] = 0$) :** Le résultat dépasse $15$ si la retenue vaut $1$ (**`carry_o = 1`**).
     * **Soustraction ($op\_i[3] = 1$) :** Le résultat est négatif (donc invalide en non signé) s'il y a un emprunt, c'est-à-dire quand **`carry_o = 0`**.
     * **En résumé :** En non signé, **`error_o = op_i[3] XOR carry_o`**.

2. **Opérations de comparaison (`Comparateur` — opcodes `01xxxx`) :**
   * Pour les 6 comparaisons valides (`011001` à `011110`), il n'y a jamais d'erreur (**`error_o = 0`**).
   * Pour les codes inutilisés de cette plage (`010xxx`, `011000` et `011111`), l'opcode n'est pas défini donc **`error_o = 1`**.

3. **Opérations logiques (`Logique` — opcodes `10xxxx`) :**
   * Les opérations bit-à-bit (`AND`, `OR`, `NAND`, `XOR`) ne peuvent pas déborder et tous les opcodes sont valides : **`error_o = 0`**.

4. **Opérations non définies (opcodes `11xxxx`) :**
   * L'opcode est inconnu : **`error_o = 1`**.