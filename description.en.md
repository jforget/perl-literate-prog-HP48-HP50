-*- encoding: utf-8; indent-tabs-mode: nil -*-

# Literate programming for HP-48 and HP-50

## Introduction

This is a self-describing text showing the
format I chose to use literate programming for
RPL-using HP calculators.


Actually, I have reused this process for programs beyond HP-48 and HP-50.
I use it for systems where writing comments is cumbersome and yet
very necessary. For the moment, this includes HP-41, APL and shell.


It is better to read in parallel the source file (French + English
with some special symbols) and the generated HTML file, so the
operation of the system is easier to understand.


Literay programming was popularised, maybe even created,
by Donald Knuth. Starting with a source file
<var>toto</var><tt>.web</tt> which contains code, comments and
a few editing commands, he generates a
<var>toto</var><tt>.tex</tt> documentation file and a
<var>toto</var><tt>.p</tt> Pascal source file for the program.
A variant named CWEB starts with a
<var>toto</var><tt>.cweb</tt> file to produce a
<var>toto</var><tt>.tex</tt>  documentation file and a
<var>toto</var><tt>.c</tt> C programme file.

Since the human-readable file is different from the compiler-readable
file, they can present the code in a different order. So, for
example, the <var>foo</var><tt>.tex</tt> file can first give a
general overview of the algorithm, then define the main data
structure, show the details of the algorithm and end with
intialisation and error processing. On the other hand, the
<var>toto</var><tt>.p</tt> file must begin with the definition of
data structures and the declaration of global variables and the
algorithm's main functions include the processing of border cases
and of errors. The source file
<var>toto</var><tt>.web</tt> has the same structure as
<var>toto</var><tt>.tex</tt>, so the programme generating
the T<sub>E</sub>X file is called <tt>weave</tt>. On the other
hand, the structure of <var>toto</var><tt>.p</tt> is very different,
so the programme generating the Pascal file is called <tt>tangle</tt>.


My version differs on several points.
<ul>
<li>The human-readable file is not T<sub>E</sub>X  but HTML and Markdown</li>
<li>WEB and CWEB generate only one documentation file, my
programme uses a multilingual <tt>.hpweb</tt> file to generate
several documentation files.</li>
<li>WEB generates Pascal, CWEB generates C or C++ and my programme
generates User-RPL for HP-48-50 calculators or other sources files in APL or shell or for HP-41.</li>
<li>Both WEB and CWEB provide two different programs to weave and tangle.
In my project, the same program weaves the documentation files and tangles
the code files.</li>
<li>CWEB has a cumbersome mechanism to produce several code files: change files
(a bit like <tt>diff</tt>'s output, processed by <tt>patch</tt>, but less efficient).
And the generated code files are much similar, such as a C program for <tt>gcc</tt>
and a C program for <tt>mingw</tt> and doing the same things.
My system allows you to produce easily several code files, either variants of a basic
file, or independant dissimilar files.</li>
</ul>


## Principle

The source text comes from a <a href='http://www.yaml.org/'>YAML</a>
file. This YAML file contains two variables. The first one,
much shorter than the second one, lists all the languages
used and for each of them gives the title of the HTML file.
It lists also the filenames where code will be stored.


The second variable is a list of chunks. Each chunk can be:

<ul>
  <li>a section fragment</li>
  <li>some text</li>
  <li>some code</li>
</ul>

Note that a "section fragment" is not the same as a section.
A section fragment ends when the next fragment, text or code, begins.
A section is the collection of all fragments from a first section
fragment (included) to the next section fragment (excluded, because
it belongs to the next section).


Weaving consists in building an HTML file and a Markdown file for each language code.
The program scans all fragments, keeping the sections fragments
and code fragment irrespective of the language code and the
text fragments tagged with the current language code.

Each fragment is slightly reformatted. A section fragment
generates a <tt>&lt;hn&gt;</tt> tag and a
<tt>&lt;a name='xxx'&gt;</tt> tag.
The code fragments are sandwiched between <tt>&lt;pre&gt;</tt> tags.

In addition, in text fragments and code fragments, the program
looks for references tagged by "@" or "|" and generates
links <tt>&lt;a href='#xxx'&gt;</tt> to other
sections.


For a given code file, tangling consists in the following actions.
The program starts by listing all the sections assigned to the
file. It also scans all code fragments from all sections to extract the insertion-type
references. It checks there is no cycle in these references
and determines in which order the sections will be generated
by inserting other sections. Then, the program writes the
code sections into the final code file.


## Details

### Characters set

The file is written in UTF-8 to be readable by YAML. But the
target machine uses a specific encoding similar to ISO-8859.
For the characters belonging to the HP-48/HP-50 character set,
but not the ISO-8859-1 character set, you will use backslash sequences.
So, the following lines:


```
\<) \x- \.V \v/ \.S \GS \|> \pi \.d \<= \>= \=/ \Ga \-> \<- \|v \|^
\Gg \Gd \Ge \Gn \Gh \Gl \Gr \Gs \Gt \Gw
\GD \PI \GW \[] \oo
```

will result in:


```
∡ x̄ ∇ √ ∫ ∑ ▶ π ∂ ≤ ≥ ≠ α → ← ↓↑
γ δ ε η θ λ ρ σ τ ω
∆ ∏ Ω ∎ ∞
```

Note that some characters which do belong to the ISO-8859-1 character set
have a backslash sequence, although they would not need it.


```
\<< \^o \Gm \>> \.x \O/ \Gb \:-
«   °   µ   »   ×   Ø   ß   ÷
```

And if you need to display such a sequence without replacing it
by the special character, you just have to insert a backslash before
the sequence, to deactivate the interpretation of the backslash.
The same procedure applies to the cases when "@" or "|" would be
wrongly understood as links to other sections.


The German-speaking people will note that the character set used by HP
calculators does not make any difference between the Greek beta and the
German s-tsett. I am sorry, but it is not my fault.


### YAML Syntax

For the YAML syntax, the first line of each code or text chunk
must begin with 4 spaces and the next lines must begin
with 4 or more spaces.


When a language code is required, it is allowed to enter several
codes joined with a comma. This allows to use the same text for all
given languages, if this text contains no translatable element,
or if the translation is identical to the original sentence.For example


```
'fr,en': Introduction
```

is replaced by:


```
fr: Introduction
en: Introduction
```

In a text fragment or a code fragment, the first line specify the
minimum indentation of the lines of the fragment and all subsequent
lines must have the same indentation or a greater one. To circumvent
this limitation, you can write a first line consisting of a sigle
period, following four spaces. And then, you can choose any indentation
for the second line and the other. During weaving and tangling, the
first line with a single period will be removed.

For code fragments, this feature allows you to keep a consistent
indentation from a code fragment to the next. For text fragments,
I do not know in which case it could be useful, but you
never know.

And if you really want a line consisting of a single period,
just write two such lines. The first will be removed, but the
second one will be kept.

Actually, there is a syntax in YAML dealing with a group of lines
in which the first line is more indented than the next. But I do
not remember it. My trick with an isolated period is simple and
easy to remember.


Remember that YAML does not like tabulations. So, use <tt>untabify</tt>
under Emacs or <tt>expand</tt> under shell when necessary. Emacs users
can also insert a comment such as:

<pre>
# -*- indent-tabs-mode: nil -*-
</pre>


## Preparation

The preparation consists in determining which languages will be used
for the documentation and which files will contain code. This is specified
in the first variable typed in the YAML file. This variable is a hashtable,
which is interpreted in the following way.

If the value of a key-value pair is <tt>dir</tt>, <tt>dir.</tt>, <tt>code</tt> or <tt>code.</tt>, then
the key is the filename for a file which will contain code. Else, the
key is a language code and the value is the title of the generated HTML
file.
The difference between <tt>dir</tt> and <tt>dir.</tt> is that in a
<tt>dir</tt> or <tt>code</tt> file, the real numbers are typed
with a comma separating the integer part from the fractional part,
while in a <tt>dir.</tt> or <tt>code.</tt> file, the separation is
acheived with a period.


If you go back to the beginning of the present <tt>desctiption.hpweb</tt>
file, you can see that three code files will be generated: <tt>ex1</tt>,
<tt>ex2</tt>, <tt>ex3</tt> and <tt>ex-cycle</tt>. The documentation is written in
English (language code <tt>en</tt>, title "Literate programming for HP-48 and HP-50")
and in French (language code <tt>fr</tt>, title <i lang='fr'>Programmation littéraire
pour HP-48 et HP-50</i>).


Other values have been added after the initial version:
<tt>code41</tt>, <tt>apl</tt> et <tt>shell</tt>.


For language codes, there is no check with the ISO-639 table.
You can enter any alphanumeric string for the various codes. Thus, if
the <tt>ams-soviet.hpweb</tt> file starts with:


```
'Side_Globe,Don_Kay': Small_Fred,Mare_Tail
Skip_Spin: code
Flap_Lid: dir
Light_Bulb: Top_Trough,Rum_Tub,Straight_Key
```

there will be two code files:
<tt>Skip_Spin</tt> et <tt>Flap_Lid</tt>,
and three HTML documentation files:
<tt>ams-soviet.Side_Globe.html</tt> titled "Small_Fred,Mare_Tail"
<tt>ams-soviet.Don_Kay.html</tt> titled "Small_Fred,Mare_Tail"
et <tt>ams-soviet.Light_Bulb.html</tt> titled "Top_Trough,Rum_Tub,Straight_Key"
and three Markdown documentation files with similar names.
The language codes "Side_Globe", "Don_Kay" and "Light_Bulb"
are not mentionned in the ISO-639 table.


These examples are far-fetched. A more concrete example is to use a
<tt>docu</tt> "language code" for the user documentation, a <tt>docp</tt>
"language code" for the documentation for programmers and a <tt>docm</tt>
"language code" for the maintenance documentation.

Anyhow, we will continue to pretend these are language codes.


## Weaving

Weaving is explained hereafter for one language code, although
the real program processes all language codes in parallel.

The name of the output file is the name of the input file,
minus the <tt>.hpweb</tt> extension, plus a dot, the language
code and the extension <tt>.html</tt> or <tt>.md</tt>. For example, for
the English language, the
input file <tt>description.hpweb</tt> will produce the output
files <tt>description.en.html</tt> and <tt>description.en.md</tt>.


The title specified in the first variable of the YAML
file gives the contents of the <tt>&lt;title&gt;</tt>
and the <tt>&lt;h1&gt;</tt> elements in the HTML file and
the <tt>#</tt> element of the Markdown file.


When weaving, the program extracts all section fragments, all texts
which belongs to the current language code, all code fragments.


Each section fragment generates a <tt>&lt;h2&gt;</tt> tag and a
<tt>&lt;a name='...'&gt;</tt> tag, the name of which is
given by the <tt>section</tt> attribute. The contents is the
section name and the text associated to the language.
In the Markdown file, there will be a <tt>##</tt> tag instead of the <tt>h2</tt>
tag, but nothing to replace the <tt>a name</tt> HTML tag.


If a section fragment includes a <tt>level</tt> key, the program
uses the corresponding value <var>n</var> to generate a
<tt>&lt;h</tt><var>n</var><tt>&gt;</tt> tag instead of
the <tt>&lt;h2&gt;</tt> tag.


If the value corresponding to the <tt>section</tt> key is 0,
this is an anonymous section. The
<tt>&lt;a name='...'&gt;</tt> tag will be generated using the
section's title.


Default tagging for HTML texts is <tt>&lt;p&gt;</tt>, so
each paragraph (text fragment or text fragment portion
delimited by empty lines) will be surrounded by
<tt>&lt;p&gt;</tt> and <tt>&lt;/p&gt;</tt>, except if another tag exists
both at the beginning and at the end of the paragraph.
And if you do not want any tag, just add
an HTML comment <tt>&lt;!-- blahblah --&gt;</tt>.


All code fragments are used when generating
the HTML file.
In addition, for <tt>GROB</tt> fragments, the program generates
a <tt>.png</tt> file containing the bitmap (to be developped
later, it is not ready yet).
This file is not automatically linked to in the generated
HTML file, you must include a <tt>&lt;img&gt;</tt> tag in
a text fragment to show it.


In all cases, section fragments, text and code, weaving looks for substrings
starting with a backslash to substitute them with the associated
character (or actually, the corresponding
<tt>&amp;#</tt><var>nnn</var><tt>;</tt> sequence in the HTML file). For example,
<tt>\pi</tt> will be replaced by π in the Markdown file
and by <tt>&amp;#960;</tt> in the HTML file.
At the same time, in the HTML file, the section names sandwiched between two <tt>@</tt>'s
or two <tt>|</tt>'s are changed into an hyperlink to the section
with the corresponding name (yet, there is no check; if the name
defines no section, the link will be dangling).
And for code fragments, the <tt>&lt;&gt;&amp;</tt> are escaped.
This is not done for section fragments and texts, because they already include
HTML tags, and we consider that all <tt>&lt;&gt;&amp;</tt> are already coded
as <tt>&amp;lt;&amp;gt;&amp;amp;</tt>.


## Tangling

Tangling results in the generating of one or several
files loadable on HP-48's and HP-50's. There can
be several reasons to generate several files&nbsp;:

<ul>
<li>one file contains the basic functions and the others contain the extensions,</li>
<li>each file contains prompt strings and message strings in a different language,</li>
<li>the files apply to different target machines, for example one file for HP-48's
with a 64×131 screen and another file for HP-50's with a 80×131 screen,</li>
<li>one file stores all routines into global variables, to facilitate
development and debuging, while the other stores these routines as local variables
inside the main programs.</li>
</ul>


### Extracting the links

The first step consists in collating all code fragments which are
involved in building the current code file, and extracting all the
links (calls and inserts) which exist in these fragments.
During this step, the program builds a graph for the insert links
and another graph for the call links.


Selecting code is done on two levels: section and fragment.
An anonymous section  (<tt>section:&nbsp;0</tt>) is never selected
in any code file. A named section will be selected if the
section fragment includes a attribute whose key is the
code file name and whose value is a number, 1 or greater.
If no such attribute exists, the section might still be selected
if it is included in an already selected section (or included
in a section included in... and so on, recursively).


The selection on fragments is rather similar to the selection on sections.
If a fragment contains an attribute pair whose key is a code file name,
the fragment will be selected in this file. The main difference with section
selection is that if the code fragment does not contain any attribute
whose key is a code file name, then this fragment will be used by all
code files.


Here is an example (best understood if you read the <tt>.hpweb</tt> file)


#### `CVTCAR`

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

#### `SUBSG`

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

The <a href='#cvtcar' class='call'>CVTCAR</a> section will be selected in three files, <tt>ex1</tt>, <tt>ex2</tt> and <tt>ex3</tt>.
On the other hand, it seems that the <a href='#subsg' class='call'>SUBSG</a> section will be selected
only in <tt>ex2</tt> and <tt>ex3</tt>. For the <a href='#cvtcar' class='call'>CVTCAR</a> section, the <tt>ex2</tt> file
will store values into global variables, for debugging purposes (using operation <tt>STO</tt>). The
<tt>ex1</tt> and <tt>ex3</tt> files will store them into local variables (using operation
horizontal arrow <tt>→</tt>). In addition,
the <tt>ex1</tt> file will include the <a href='#subsg' class='call'>SUBSG</a> code and store it into another local variable.


As for the <tt>ex-cycle</tt> file, il selects the <a href='#subsg' class='call'>SUBSG</a> section, which includes the
<a href='#cvtcar' class='call'>CVTCAR</a> section, which in turn includes <a href='#subsg' class='call'>SUBSG</a>, which causes some kind of problem.


### Sorting the code sections

The second step achieves two purposes:

<li>
topologically sort the sections according to the insertion
links, to determine in which order the sections have to be generated.
</li>
<li>
detect any cycles in the insertions graph.
</li>
</ol>


The topological sort consists in scanning all insertion links and comparing
the hierachical level of the calling section with the level of the inserted
section. The level of the calling section must be greater than (and not equal to)
the level of the inserted section. If this is not the case, the calling section's
level is updated, using the inserted section's level plus one.


The algorithm stops in two ways. First, it stops if during a scan of all insertion
links, no hierarchical level has been updated. The sort is finished. Or second,
if we notice that a section has a hierarchical level greater than the total
number of sections. That means that there is a cycle in the insertion links and
that the sort must be interrupted. And we do not generate the code file.


NB: in the current version, all sections and all insertion links are checked,
even if some sections are "dead code", that is, they are not linked to the
generated file with insertion links. That can produce a rejection of the generated
file. This will be corrected in a next release.


There are other error conditions, such as a section which has not been included
or a section which has been included several times instead of just once,
or a global section which has been included into another section, just as
a local one. These error cases are not detected by the program. The programmer
has to detect them, mainly by examining the insertion and call graph.


### Generating the code sections

Sections are generated in hierarchical order. In each code fragment,
the program extracts both types of links, insertions links tagged
by <tt>|</tt> and call links tagged by <tt>@</tt>. The processing
of insertion links is simple: they are replaced by the code
of the inserted section. The processing of call links is a bit more
complicated: if the section is a global one (file-level section),
the link is replaced by the section name and that's all. If the
section is a local one, it has to be followed by <tt>EVAL</tt>.

A section can be global in a first code file and local in a second
one. This is the reason why <tt>EVAL</tt> is not written in the
source file, but generated on demand when building the code files.


### Writing the code files

For a <tt>code</tt> type file, all global sections are written
ordered by their <tt>rang</tt> attribute (which was specified
by the filename). For a <tt>dir</tt> type file, the program
writes first the keyword <tt>DIR</tt>. Then, for all global
sections, it writes the section name and the section code.
At last, it ends the code file with the keyword <tt>END</tt>.
Such a file allows the creation of a whole directory in
the target calculator.


## Use with HP-41

This format and this program can be used to generate programs for the HP-41,
with some constraints. First, the generated files can only be <tt>code41</tt>
files, there is no equivalent of <tt>dir</tt> files.
In addition, <tt>@</tt>-marked calls are prohibited. But you can use
<tt>|</tt>-marked insertions.


When using the command-line parameter <tt>-number</tt>,
the lines in the code files are numbered, so it is easier to check
for missing lines when entering the program into a HP-41.
The output must be compatible with
<a href="https://www.hpmuseum.org/software/41uc.htm">HP41UC</a>.


## Use with APL and shell

This format and this program can also be used to generate APL programs and shell
scripts. For this, use a filetype <tt>apl</tt> or <tt>shell</tt>. Contrary to
the HP-41 programs, you can mark calls with <tt>@</tt>, but you cannot insert
a piece of code elsewhere with <tt>|</tt>. Anyhow, if you want to code a
seldom used APL modulo or a frequently used shell pipe, you must type <tt>\|</tt>
with a backslash (which does not show in the Markdown variant, sorry).


## License

This code is published under the same terms as Perl: GPL and Artistic License.


