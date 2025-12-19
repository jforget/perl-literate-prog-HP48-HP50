-*- encoding: utf-8; indent-tabs-mode: nil -*-

# Programmation littéraire pour HP-48 et HP-50

## Introduction

Ceci est un texte auto-descriptif
présentant le format que j'ai choisi pour faire
de la programmation littéraire pour
les calculatrices HP fonctionnant en RPL.


En fait, j'en suis venu à utiliser ce mécanisme pour autre chose que des
HP-48 et des HP-50. Je l'utilise pour d'autres programmes pour lesquels
l'ajout de commentaires est malcommode et pourtant nécessaire. Pour l'instant,
cela ajoute les HP-41 et les programmes APL ou shell.


Il est préférable de lire en parallèle le texte d'origine (français + anglais
avec les symboles spéciaux) et le texte HTML généré
pour mieux se rendre compte du fonctionnement du système.


La programmation littéraire a été popularisée, peut-être même
inventée, par Donald Knuth. À partir d'un fichier source,
<var>toto</var><tt>.web</tt> contenant du code, des commentaires
et quelques directives d'édition, il génère un fichier
<var>toto</var><tt>.tex</tt> pour la documentation et un fichier
source Pascal <var>toto</var><tt>.p</tt> pour le programme à proprement
parler. Une variante appelée CWEB part d'un fichier
<var>toto</var><tt>.cweb</tt> pour obtenir un fichier de documentation
<var>toto</var><tt>.tex</tt>  et un fichier programme en C
<var>toto</var><tt>.c</tt>

Et comme le fichier à l'usage des humains est distinct
du fichier à l'usage du compilateur, rien n'oblige à ce qu'ils
présentent le code dans le même ordre. Donc, par exemple, le fichier
<var>toto</var><tt>.tex</tt> commence par donner une idée générale
de l'algorithme, puis il définit la structure de données principale,
puis le détail de l'algorithme, puis les initialisations et enfin
le traitement des erreurs. À l'inverse, pour les besoins du compilateur,
le fichier <var>toto</var><tt>.p</tt> doit commencer par la description
des structures de données, puis la déclaration des variables globales
et le corps de l'algorithme est mélangé avec le traitement des cas limites
et la gestion des erreurs.
Comme le fichier source <var>toto</var><tt>.web</tt>
et le fichier <var>toto</var><tt>.tex</tt>
ont la même structure, Knuth a appelé <tt>weave</tt>
(«&nbsp;tisser&nbsp;») le programme permettant
de passer de <var>toto</var><tt>.web</tt> à <var>toto</var><tt>.tex</tt>.
À l'inverse, <var>toto</var><tt>.p</tt> et <var>toto</var><tt>.web</tt>
ont des structures nettement différentes, donc le programme permettant de
passer de l'un à l'autre s'appelle <tt>tangle</tt> («&nbsp;emmêler&nbsp;»).


Ma version diffère sur plusieurs points.
<ul>
<li>Le fichier documentation généré n'est pas du T<sub>E</sub>X,
mais du HTML et du Markdown.</li>
<li>WEB et CWEB génèrent un seul fichier de documentation, mon
système permet de générer plusieurs fichiers dans des langues
différentes à partir d'un fichier <tt>.hpweb</tt> multilingue.</li>
<li>WEB génère du Pascal, CWEB génère du C, mon système génère du
RPL pour les calculatrices HP-48-50 ou des sources HP-41, APL ou shell.</li>
<li>WEB et CWEB fournissent chacun deux programmes différents pour générer
d'un côté le fichier de documentation et de l'autre côté le fichier source.
Dans mon système, c'est le même programme qui tisse les fichiers de documentation
et qui emmêle les fichiers de code.</li>
<li>CWEB a un mécanisme assez malcommode pour générer plusieurs fichiers
de code&nbsp;: les <i lang='en'>change files</i> (comme le résultat de <tt>diff</tt>
et son traitement ultérieur par <tt>patch</tt>, mais en moins bien). Et cela permet de générer seulement
des fichiers de code similaires, se distinguant par quelques variantes éparses,
par exemple un fichier à compiler avec <tt>gcc</tt> et un fichier faisant la même
chose, mais compilé par <tt>mingw</tt>.
Mon système permet de générer des fichiers de code qui peuvent être des variantes
d'un fichier de base, ou bien des fichiers complémentaires, partageant très peu de code.</li>
</ul>


## Principe

Le texte source est écrit dans un fichier
<a href='http://www.yaml.org/'>YAML</a> contenant deux
variables. La première, très brève, se contente d'énumérer
les langues utilisées en donnant à cette occasion le titre
du fichier HTML généré. Elle liste également les noms des fichiers
qui contiendront le code.


La seconde variable est une liste de fragments. Chaque fragment
peut être&nbsp;:

<ul>
  <li>un fragment section</li>
  <li>du texte</li>
  <li>du code</li>
</ul>

Attention, il faut bien distinguer les «&nbsp;fragments sections&nbsp;»
des «&nbsp;sections&nbsp;». Un fragment section se termine lorsque
commence un autre fragment, que ce soit du texte, du code, voire un
autre fragment section, tandis qu'une section regroupe tous les fragments
à partir d'un fragment section (inclus) jusqu'au fragment section suivant
(exclus, car faisant partie de la section suivante).


Le tissage consiste à construire un fichier HTML et un fichier Markdown pour chaque
code langue. Le programme passe en revue tous les fragments
et tient compte des fragments sections, des fragments codes
indépendamment de la langue et des fragments textes associés
à la langue en cours de traitement.

Pour chaque fragment, le programme effectue un léger traitement
de formattage. Ainsi, les fragments sections donnent lieu
à une balise <tt>&lt;hn&gt;</tt> et à
une balise <tt>&lt;a name='xxx'&gt;</tt>.
Les fragments de code sont encadrés par des balises <tt>&lt;pre&gt;</tt>.

En outre, dans les fragments de texte et de code, on repère les références
(balisées par «&nbsp;@&nbsp;» ou par «&nbsp;|&nbsp;») pour établir
un lien <tt>&lt;a href='#xxx'&gt;</tt> vers
d'autres sections.


L'emmêlement se passe ainsi pour un fichier de code donné&nbsp;:
le programme commence par répertorier toutes les sections
appelées dans ce fichier. Il passe également en revue tous les
fragments de code de toutes
les sections pour en extraire les références consistant à inclure
une section dans une autre. Après s'être assuré qu'il n'y a pas
de dépendance circulaire et après avoir déterminé dans quel ordre
il faut générer les sections, le programme effectue ces insertions
et écrit le fichier de code final.


## Détails

### Jeu de caractères

Le fichier est écrit en UTF-8 pour être lu par YAML. Mais la machine
cible utilise un codage spécifique assez proche de l'ISO-8859.
Pour les caractères utilisés par les HP-48 et HP-50 qui ne figurent
pas dans le jeu de caractère ISO-8859-1, il faudra utiliser une
séquence de caractères introduite par un <i lang='en'>backslash</i>.
Par exemple, les lignes&nbsp;:


```
\<) \x- \.V \v/ \.S \GS \|> \pi \.d \<= \>= \=/ \Ga \-> \<- \|v \|^
\Gg \Gd \Ge \Gn \Gh \Gl \Gr \Gs \Gt \Gw
\GD \PI \GW \[] \oo
```

donneront lieu à&nbsp;:


```
∡ x̄ ∇ √ ∫ ∑ ▶ π ∂ ≤ ≥ ≠ α → ← ↓↑
γ δ ε η θ λ ρ σ τ ω
∆ ∏ Ω ∎ ∞
```

Notons que certains caractères qui figurent dans le jeu de caractères
ISO-8859-1 ont eux aussi une séquence introduite par <i lang='en'>backslash</i>
alors qu'il n'en ont pas réellement besoin.


```
\<< \^o \Gm \>> \.x \O/ \Gb \:-
«   °   µ   »   ×   Ø   ß   ÷
```

Et si vous avez besoin de faire apparaître une telle séquence
sans la remplacer par le caractère spécial, il suffit d'insérer
un <i lang='en'>backslash</i> avant cette séquence, pour désactiver
l'interprétation du <i lang='en'>backslash</i>. C'est vrai également
pour désactiver l'interprétation du «&nbsp;@&nbsp;» et du «&nbsp;|&nbsp;»
dans les cas où ils pourraient être compris à tort comme des liens vers d'autres
sections.


Remarque&nbsp;: les germanophones remarqueront que le jeu de caractères
des calculatrices HP confond allègrement le béta grec et le s-tsett allemand.
Désolé, ce n'est pas moi qui ai pris cette option.


### Syntaxe YAML

Pour les besoins de la syntaxe YAML, pour chaque fragment code ou texte,
la première ligne du contenu doit commencer par 4&nbsp;espaces
et les lignes suivantes par 4&nbsp;espaces ou plus.


Lorsqu'il y a besoin d'un code langue, il est possible de spécifier
plusieurs codes langues séparés par des virgules. Cela permet de mettre
en commun le texte associé, s'il ne contient aucun élément traduisible
ou si la traduction est identique à l'original. Par exemple.


```
'fr,en': Introduction
```

est remplacé par&nbsp;:


```
fr: Introduction
en: Introduction
```

Dans un fragment de texte ou un fragment de code, la première
ligne fixe l'indentation minimale du fragment et toutes les lignes
qui suivent doivent avoir une indentation supérieure ou égale.
Pour passer outre cette limitation, vous pouvez spécifier une
première ligne composée en tout et pour tout d'un point précédé
de quatre espaces et vous avez toute latitude pour l'indentation
de la deuxième ligne. La première ligne sera supprimée lors de
l'emmêlement et du tissage.

Pour les fragments de code, cela vous permet d'avoir une indentation
cohérente d'un fragment à l'autre. Pour les fragments de texte, je
ne vois pas dans quelles circonstances cela pourrait servir, mais
on ne sait jamais.

Et si vous voulez quand même une ligne constituée d'un point et
rien d'autre, mettez cette ligne en double. Le premier exemplaire
de la ligne sera supprimé, le second sera conservé.

En fait, il existe une syntaxe particulière en YAML pour avoir un groupe
de lignes dans lequel la première ligne est plus indentée que les
suivantes, mais je ne l'ai pas retenue. Mon astuce avec un point isolé
est simple et facile à mémoriser.


Rappel&nbsp;: YAML n'aime pas les tabulations. Pensez donc à
utiliser si nécessaire <tt>untabify</tt> sous Emacs ou <tt>expand</tt>
sous shell. Ou bien, pour les utilisateurs d'Emacs, vous pouvez
insérer au début du fichier un commentaire tel que&nbsp;:

<pre>
# -*- indent-tabs-mode: nil -*-
</pre>


## Préparation

La préparation consiste à répertorier quelles langues utiliser
pour la documentation et quels fichiers générer pour le code.
Cela se fait en examinant la première variable saisie dans le fichier
YAML. Cette variable est une table de hachage, interprétée ainsi&nbsp;:

Dans le couple clé-valeur, si la valeur est <tt>dir</tt>, <tt>dir.</tt>, <tt>code</tt> ou <tt>code.</tt>,
la clé est le nom d'un fichier qui contiendra du code. Sinon, la clé
est un code langue et la valeur est le titre du fichier HTML généré.
La différence entre <tt>dir</tt> et <tt>dir.</tt> est qu'un fichier
<tt>dir</tt> ou <tt>code</tt> contient des nombres réels avec une virgule pour séparer
la partie entière de la partie décimale, tandis qu'un fichier <tt>dir.</tt>
ou <tt>code.</tt> contient des nombres réels où la séparation se fait
avec un point.


Si vous remontez au tout début du présent fichier <tt>.hpweb</tt>,
vous pouvez constater qu'il y aura quatre fichiers de code&nbsp;:
<tt>ex1</tt>, <tt>ex2</tt>, <tt>ex3</tt> et <tt>ex-cycle</tt>. De plus, la documentation
est écrite en français (code <tt>fr</tt>, titre «&nbsp;Programmation littéraire
pour HP-48 et HP-50&nbsp;») et en anglais (code langue <tt>en</tt>, titre
<i lang='en'>Literate programming for HP-48 and HP-50</i>).


D'autres valeurs ont été ajoutées ultérieurement pour les fichiers codes&nbsp;:
<tt>code41</tt>, <tt>apl</tt> et <tt>shell</tt>.


Pour les codes langues, il n'y a pas de contrôle vis-à-vis de la table ISO-639.
Vous pouvez saisir n'importe quel code constitué de caractères
alphanumériques, ils seront interprétés comme indiqué ci-dessus. Ainsi, si
le fichier <tt>ams-soviet.hpweb</tt> commence par&nbsp;:


```
'Side_Globe,Don_Kay': Small_Fred,Mare_Tail
Skip_Spin: code
Flap_Lid: dir
Light_Bulb: Top_Trough,Rum_Tub,Straight_Key
```

il y aura deux fichiers de code&nbsp;:
<tt>Skip_Spin</tt> et <tt>Flap_Lid</tt>,
trois fichiers de documentation HTML
<tt>ams-soviet.Side_Globe.html</tt> intitulé «&nbsp;Small_Fred,Mare_Tail&nbsp;»
<tt>ams-soviet.Don_Kay.html</tt> intitulé «&nbsp;Small_Fred,Mare_Tail&nbsp;»
et <tt>ams-soviet.Light_Bulb.html</tt> intitulé «&nbsp;Top_Trough,Rum_Tub,Straight_Key&nbsp;»
et trois fichiers de documentation Markdown avec des noms équivalents.
Les codes langues «&nbsp;Side_Globe&nbsp;», «&nbsp;Don_Kay&nbsp;» et
«&nbsp;Light_Bulb&nbsp;» ne figurent pas dans la table ISO-639.


Ces exemples sont caricaturaux. Un cas d'emploi plus sérieux est d'utiliser
un pseudo-code langue <tt>docu</tt>  pour la documentation utilisateur,
un pseudo-code langue <tt>docp</tt> pour la documentation destinée aux programmeurs
et un pseudo-code langue <tt>docm</tt> pour la documentation de maintenance.

Quoi qu'il en soit, nous continuons à utiliser la désignation «&nbsp;code langue&nbsp;».


## Tissage

Le tissage est présenté ci-dessous pour un code langue donné,
alors qu'il traite tous les codes langues en parallèle.

Le nom du fichier en sortie est le nom du fichier en entrée,
moins l'extension <tt>.hpweb</tt>, plus un point, le code
langue et l'extension <tt>.html</tt> ou <tt>.md</tt>. Par exemple, pour le
français, le fichier <tt>description.hpweb</tt> donnera
les fichiers en sortie <tt>description.fr.html</tt>
et <tt>description.fr.md</tt>


Le titre spécifié dans la première variable du fichier YAML
permet d'avoir le contenu de l'élément <tt>&lt;title&gt;</tt>
et de l'élément <tt>&lt;h1&gt;</tt> du fichier HTML, ainsi que
de l'élément <tt>#</tt> du fichier Markdown.


Pour le tissage, le programme extrait dans l'ordre tous les fragments sections,
tous les textes dépendant du code langue utilisé et tous les
fragments de code.


Chaque fragment section génère une balise <tt>&lt;h2&gt;</tt>
et une balise <tt>&lt;a name='...'&gt;</tt>, le nom étant
donné par l'attribut <tt>section</tt> de la section. Le contenu
est donné par le nom de la section et le texte associé à la langue.
Pour le fichier Markdown, il y aura une balise équivalente <tt>##</tt>, mais pas
de balise Markdown équivalente à la balise <tt>a name</tt> du HTML.


Si un fragment section contient une clé <tt>level</tt>, alors le
programme utilise la valeur <var>n</var> correspondante pour générer
une balise <tt>&lt;h</tt><var>n</var><tt>&gt;</tt> à la place
de <tt>&lt;h2&gt;</tt>.


Si la valeur correspondant à la clé <tt>section</tt> est 0,
il s'agit d'une section anonyme. Dans ce cas, la balise
<tt>&lt;a name='...'&gt;</tt> sera générée avec le titre de la
section.


Pour les textes, le balisage HTML par défaut est les balises <tt>&lt;p&gt;</tt>.
Chaque paragraphe (fragment de texte ou portion de fragment de texte
délimitée par des lignes vides) sera encadré par
<tt>&lt;p&gt;</tt> et <tt>&lt;/p&gt;</tt>, sauf si une balise existe
déjà au début et à la fin. Et si par hasard, vous
ne voulez pas de balise, il vous suffit d'ajouter
un commentaire HTML <tt>&lt;!-- blabla --&gt;</tt>.


Les fragments de type code sont tous repris
dans le fichier HTML généré. De plus, pour les
fragments de type <tt>GROB</tt>, le programme génère un
fichier <tt>.png</tt> représentant le bitmap associé
(développement ultérieur, ce n'est pas encore prêt).
Le fichier n'est pas appelé automatiquement dans le HTML
généré, il faut l'appeler «&nbsp;manuellement&nbsp;» avec
une balise <tt>&lt;img&gt;</tt> dans un fragment texte.


Dans tous les cas, fragment section, texte et code, le tissage recherche
les séquences introduites par un <i lang='en'>backslash</i> pour
les remplacer par le caractère associé (en fait, la séquence
<tt>&amp;#</tt><var>nnn</var><tt>;</tt> dans le fichier HTML). Par exemple,
la séquence <tt>\pi</tt> sera remplacée par π dans le fichier
Markdown et par <tt>&amp;#960;</tt> dans le fichier HTML.
En même temps, dans le fichier HTML, les noms de section encadrés par des <tt>@</tt> ou des
<tt>|</tt> sont transformés en liens hypertextes vers la section du
même nom (cela dit, il n'y a pas de contrôle, le lien peut pointer
vers le néant s'il n'existe aucune section de ce nom).
Et dans le cas des fragments codes, les <tt>&lt;&gt;&amp;</tt> sont
remplacés par la séquence HTML correspondante. Ce n'est pas fait
pour les fragments sections et les textes, car ceux-ci contiennent déjà du marquage
HTML et on considère que les  <tt>&lt;&gt;&amp;</tt> y sont déjà sous
la forme <tt>&amp;lt;&amp;gt;&amp;amp;</tt>.


## Emmêlement

L'emmêlement permet de générer un ou plusieurs fichiers
téléchargeables sur les machines HP-48 ou HP-50.
Il peut y avoir plusieurs raisons pour générer
plusieurs fichiers&nbsp;:

<ul>
  <li>un fichier contient les fonctions de base et les autres fichiers les fonctions étendues,</li>
  <li>les fichiers contiennent des chaînes de caractères dans des langues différentes
  pour les invites et les messages,</li>
  <li>les fichiers concernent des machines cibles différentes, par exemple une HP-48
  avec un affichage 64×131 et une HP-50 avec un affichage 80×131,</li>
  <li>un fichier contient les routines utilitaires sous formes de variables globales
  dans le répertoire de la HP, pour faciliter la mise au point, tandis qu'un autre
  fichier stocke toutes ces routines utilitaires dans des variables locales
  des programmes fournis.</li>
</ul>


### Extraction des liens

La première étape consiste à recenser tous les fragments de code
qui s'appliquent au fichier de code en cours de constitution et
à identifier les liens (appels et insertions) qui s'y trouvent.
À cette occasion, le programme bâtit un graphe des appels et un
graphe des insertions.


La sélection du code se fait à deux niveaux&nbsp;: section et fragment.
Si la section est une section anonyme (<tt>section:&nbsp;0</tt>),
elle ne sera sélectionnée dans aucun fichier de code.
Une section nommée sera sélectionnée pour un fichier si le fragment
section contient un attribut avec le nom du fichier comme clé
et avec un nombre supérieur ou égal à 1 comme valeur.
S'il n'y a pas un tel attribut, la section pourra être quand
même utilisée, si elle est incluse dans une section sélectionnée
(ou incluse dans une section elle-même incluse dans... et ainsi de
suite, récursivement).


La sélection sur les fragments est assez semblable. Si un fragment
comporte un attribut avec le nom du fichier de code en clé, alors
le fragment de code sera inclus dans le fichier de code.
La seule différence par rapport aux sections est que si un fragment
ne comporte aucun attribut dont la clé est un nom de fichier de code,
alors il sera appelé dans tous les fichiers.


Voici un exemple (à lire dans le fichier <tt>.hpweb</tt> pour mieux comprendre)


#### <a name='CVTCAR'>`CVTCAR`</a>

```
«
```

```
:::: ex-cycle, ex1 ::::
<<<<SUBSG>>>> → SUBSG
```

```
  «
    "<)" 128 SUBSG   "<-" 142 SUBSG   "PI" 156 SUBSG
    "x-" 129 SUBSG   "|v" 143 SUBSG   "GW" 157 SUBSG
    ".V" 130 SUBSG   "|^" 144 SUBSG   "[]" 158 SUBSG
    "v/" 131 SUBSG   "Gg" 145 SUBSG   "oo" 159 SUBSG
    ".S" 132 SUBSG   "Gd" 146 SUBSG   "<<" 171 SUBSG
    "GS" 133 SUBSG   "Ge" 147 SUBSG   "^o" 176 SUBSG
    "|>" 134 SUBSG   "Gn" 148 SUBSG   "Gm" 181 SUBSG
    "pi" 135 SUBSG   "Gh" 149 SUBSG   ">>" 187 SUBSG
    ".d" 136 SUBSG   "Gl" 150 SUBSG   ".x" 215 SUBSG
    "<=" 137 SUBSG   "Gr" 151 SUBSG   "O/" 216 SUBSG
    ">=" 138 SUBSG   "Gs" 152 SUBSG   "Gb" 223 SUBSG
    "=/" 139 SUBSG   "Gt" 153 SUBSG   ":-" 247 SUBSG
    "Ga" 140 SUBSG   "Gw" 154 SUBSG
    "->" 141 SUBSG   "GD" 155 SUBSG
    IF DUP "%%HP" POS
    THEN
      DUP 10 CHR POS 1 +
      OVER SIZE
      SUB
    END
  »
»
```

#### <a name='SUBSG'>`SUBSG`</a>

```
« 92 CHR ROT + SWAP CHR
```

```
:::: ex-cycle, ex1, ex3 ::::
  → ch av ap
```

```
:::: ex2 ::::
'ap' STO 'av' STO 'ch' STO
```

```
  « WHILE ch av POS
    REPEAT
      ch 1 OVER av POS 1 - SUB
      ap +
      ch DUP av POS av SIZE +
      OVER SIZE SUB +
      'ch' STO
    END ch
  »
»
```

```
:::: ex-cycle ::::
"cycle" <<<<CVTCAR>>>>
```

La section <a href='#CVTCAR' class='call'>CVTCAR</a> sera appelée dans les trois fichiers <tt>ex1</tt>, <tt>ex2</tt> et
<tt>ex3</tt>, tandis que la section <a href='#SUBSG' class='call'>SUBSG</a> ne sera, apparemment, sélectionnée
que dans <tt>ex2</tt> et <tt>ex3</tt>. Pour la section <a href='#CVTCAR' class='call'>CVTCAR</a>, le fichier <tt>ex2</tt>
utilisera un stockage dans des variables globales pour des besoins
de débugage (opération <tt>STO</tt>), tandis que les fichiers <tt>ex1</tt> et <tt>ex3</tt> stockeront tout dans des variables
locales (opération flèche horizontale <tt>→</tt>). De plus, le fichier <tt>ex1</tt> inclura le code de <a href='#SUBSG' class='call'>SUBSG</a> et le stockera dans
une autre variable locale.


Quant au fichier <tt>ex-cycle</tt>, il appelle au premier niveau la section <a href='#SUBSG' class='call'>SUBSG</a>,
qui inclut la section <a href='#CVTCAR' class='call'>CVTCAR</a>, qui inclut à son tour <a href='#SUBSG' class='call'>SUBSG</a>, ce qui constitue un problème certain.


### Tri des sections de code

La deuxième étape a deux buts&nbsp;:

<ol>
<li>
établir un tri topologique des sections en fonction des insertions,
pour savoir quelles sections il faut traiter en premier, lesquelles
il faut traiter en second et ainsi de suite.
</li>
<li>
détecter les cycles dans le graphe des insertions.
</li>
</ol>


Le tri topologique consiste à passer en revue tous les liens d'insertion
et à comparer le niveau hiérarchique de la section englobante et celui
de la section insérée. Le niveau de la section englobante doit être supérieur,
strictement, à celui de la section insérée. Si ce n'est pas le cas, le niveau
de la section englobante est mis à jour avec le niveau hiérarchique de la section
insérée plus 1.


Le mécanisme s'arrête dans deux cas. Tout d'abord, si on a effectué une passe
sur tous les liens d'insertion sans qu'il y ait eu besoin de mettre à jour
un niveau hiérarchique. Le tri est alors achevé. Ou alors, si on détecte qu'une
section a un niveau hiérarchique supérieur au nombre de sections existantes.
Dans ce cas, cela signifie qu'il existe un cycle dans les liens d'insertion.
Et le fichier n'est pas généré.


Remarque&nbsp;: dans la version actuelle, on tient compte de toutes les sections
et de tous les liens d'insertion déclarés pour le fichier, même si parmi
ces sections, il en existe qui constituent du «&nbsp;code mort&nbsp;» (sections
qui ne sont pas reliées au fichier à générer par un lien d'insertion). Cela peut
créer des rejets abusifs. Cela sera corrigé dans une version ultérieure.


Il y a d'autres cas d'erreur, comme une section que l'on a oublié d'inclure,
ou bien que l'on a inclue plusieurs fois alors qu'il ne fallait pas le faire,
ou encore une section globale inclue en tant que section locale.
Ces cas de figure ne sont pas traitée automatiquement par le programme,
c'est au programmeur de s'en rendre compte en jetant un coup d'&oelig;il
sur le graphe des appels et des insertions.


### Génération des sections de code

Les sections sont générées dans l'ordre de la hiérarchie croissante.
Dans chaque fragment de code, on extrait les liens, aussi bien les
liens d'insertion (balisés par <tt>|</tt>) que les liens d'appel
(balisés par <tt>@</tt>). Un lien d'insertion est remplacé par
le code de la section associée, tout simplement. Pour les liens
d'appel, c'est un peu plus compliqué. Si la section appelée est
une section globale, alors le code généré est réduit au nom de
la section. En revanche, s'il s'agit d'une section locale,
il faut la faire suivre de <tt>EVAL</tt>.

Une section peut être globale dans un premier fichier de code
et locale dans un autre. C'est pour cela que l'instruction <tt>EVAL</tt>
n'est pas codée dans le source, mais générée lors de la construction
du code.


### Écriture des fichiers de code

Pour un fichier de type <tt>code</tt>, toutes les sections globales
sont écrites, dans l'ordre de leur attribut <tt>rang</tt> (celui
qui a été spécifié par le nom du fichier de code). Pour un fichier
de type <tt>dir</tt>, on commence par le mot-clé <tt>DIR</tt>,
puis pour chaque section globale, on écrit le nom puis le code.
Et on termine par <tt>END</tt>. Cela sert à créer un répertoire
entier dans la machine cible.


## Utilisation pour les HP-41

Ce format et ce programme peuvent être utilisés pour générer des programmes
pour HP-41, avec quelques restrictions. Tout d'abord, les fichiers
générés sont de type <tt>code41</tt>, analogue à <tt>code</tt>, mais
les séquences d'appel marquées par <tt>@</tt> sont interdites. En revanche,
il est possible d'utiliser les balises d'insertion <tt>|</tt>.


Si l'on utilise le paramètre <tt>-number</tt> sur la ligne de commande,
les lignes du fichier obtenu sont numérotées, ce qui permet de mieux
contrôler la saisie du programme sur HP-41.
La sortie doit être compatible avec
<a href="https://www.hpmuseum.org/software/41uc.htm">HP41UC</a>.


## Utilisation pour APL et shell

Ce format et ce programme peuvent également être utilisés pour générer
des programmes APL et shell. Pour cela, il faut utiliser le type <tt>apl</tt>
ou <tt>shell</tt>. À l'inverse des programmes pour HP-41, on peut marquer
les appels par des balises <tt>@</tt>, mais on ne peut pas faire d'insertion
avec des balises <tt>|</tt>. De plus, pour les rares modulos d'APL et les fréquents
pipes de shell, il faut écrire <tt>\|</tt> avec un backslash (qui n'apparaît pas
dans la version Markdown, désolé).


## Licence

Ce code est diffusé sous les mêmes termes que Perl, la licence GPL
et la licence artistique.


