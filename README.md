# sim-si-la-bim
A syllabifier for Brazilian Portuguese written in *foma*.

## intro

Syllabification is the process of breaking a given word into syllables according to the language's rules, and it helps NLP systems achieve goals such as to map a word into its phonetic transcription, or to generate prosody.Text-to-speech (TTS) systems, the language generation technology behind applications such as virtual assistants and screen readers, often perform syllabification as part of its text pre-processing pipeline, for example.

A few syllabifier algorithms have been proposed for Brazilian Portuguese, including that described in [(Silva et al.)](https://biblioteca.sbrt.org.br/articles/2721). Using that as reference, I have implemented the same rules in *foma*, a finite-state compiler and library.

## methodology

In their algorithm, the rules are based on the vowel(s) being the nucleus of each syllable, and the algorithm checks each vowel and determines which symbols around it will be considered to form a syllable, and from which those will be separated.

For example, this is the translated pseudocode of the first rule and the examples provided:

> ```
> if V = p⁰ and V ≠ <ã>,<õ> and ^(+1) = V and ^(+1) != G then
>    Case 1
> end if
> ```
> Examples: _a-eronave, a-inda_


What the conditional statement says is: if the vowel is in the initial position of a syllable, it is different from `ã` and `õ`, and the following symbol (`^(+1)`) is a vowel (and not a semi-vowel), then it is a Case 1.

The Cases define what will be done with the vowel and the symbols around it; Case 1, for example, means the vowel will be separated from the following symbol.

Thus, the same rule is written in *foma* as below:

```
def VowelsExceptÃÕ V .o. ~[ ã | õ ];
def Rule1Case1 [..] -> "-" || VowelsExceptÃÕ _ V;
```

It is a rewrite rule that will insert a single hyphen (`[..] -> "-"`) into the space between a symbol that matches `VowelsExceptÃÕ` and another symbol that matches `V`.

`V` is the regex that matches vowels, defined based on Table 1 in the paper (translated below). It is defined in the beginning of the `syllabifier.foma` file. Assuming it has been loaded into *foma*, this single rule can be compiled as regex on its own and tested:

```
foma[0]: regex Rule1Case1;
979 bytes. 3 states, 33 arcs, Cyclic.
foma[1]: down aeronave
a-eronave
```

In this example we see that the rule only matches with one part of the given word, the two first letters, and inserts a hyphen between them.

All the rules have been defined in `syllabifier.foma` this way, as well as a couple of pre- and post-processing rules, and compiled into the final transducer that takes a word and returns the same word with hyphens where the syllable boundaries are ([demonstration in "how to use" section](#how-to-use)).

### symbols

The symbols considered as graphemes can be single letters or clusters of letters that should be handled together. For example, `ca` and `ce` are in different groups because the consonant `c` varies in sound depending on the vowel, such as in the words "casa" [kasa] and "cedo" [sɛdu].


| name | definition           | symbols                                                                      |
|------|----------------------|------------------------------------------------------------------------------|
| V    | Vowels               | `a`,`e`,`o`,`á`,`é`,`ó`,`í`,`ú`,`ã`,`õ`,`â`,`ê`,`ô`,`à`,`ü`                                  |
| G    | Semi-vowels          | `i`,`u`                                                                         |
| C    | All consonants       | `lh`,`nh`, CO, CF, CL, CN                                                   |
| CO   | Occlusive consonants | `p`,`t`,`ca`,`co`,`cu`,`que`,`qui`,`b`,`d`,`ga`,`go`,`gu`,`gue`,`gui` |
| CF   | Fricative consonants | `f`,`v`,`s`,`ce`,`ci`,`ç`,`z`,`ss`,`ch`,`j`,`ge`,`gi`,`x`                    |
| CL   | Liquid consonants    | `l` except `lh`, `r`, `rr`                                                         |
| CN   | Nasal consonants     | `m`,`n`                                                                         |

\* Some other symbols used in the paper such as `SP` (whitespace) and `F` (line break) were instead handled as *foma*-style rules as those will work with only one word at a time.

\** Auxiliary regexes have been defined before the relevant rules that need them, for readability purposes.

### cases

As organized in the paper, each Case describes which graphemes will be joined with the vowel to form a syllable, or separated.

| case   | action                                                                                           |
|--------|--------------------------------------------------------------------------------------------------|
| Case 1 | Vowel is separated from the next grapheme                                                        |
| Case 2 | Vowel is joined with the next grapheme and separated from the subsequent   ones                  |
| Case 3 | Vowel is joined with the previous grapheme and separated from the   following                    |
| Case 4 | Vowel is joined with the previous and the next grapheme and separated   from the subsequent ones |
| Case 5 | Vowel is joined with the following 2 graphemes and separated from the 3rd   onwards              |
| Case 6 | Vowel is joined with the previous grapheme and all the following until   the end of the word     |


## how to use

The information on how to install *foma* is available in [its homepage](https://fomafst.github.io/). The syllabifier source can be loaded with the `foma` command line interface:

```
foma[0]: source syllabifier.foma
```

Once loaded, all of the rules and the final Syllabifier transducer will be ready to use. In order to syllabify a word, the `apply down` shell can be used.

```
foma[1]: apply down
apply down> aeronave
a-e-ro-na-ve
```

The word exemplified above, "aeronave" (aircraft), uses the first rule demonstrated in the previous section to define its first syllable, separating the vowel `a` from `e`. With the other rules already implemented, the transducer knows how to break apart the other syllables as well.

The same word exemplified before, "aeronave" (aircraft), now matches other rules besides the first one and has all of its syllables correctly



```
apply down> morfologia
mor-fo-lo-gi-a
apply down> computacional
com-pu-ta-cio-nal
```

## conclusion
This is a preliminary version of a syllabifier for Brazilian Portuguese that I wrote as an assignment for my Computational Morphology course. It implements all of the rules described in the paper, though some of them can still be improved upon and any issues encountered will be tracked in the [Issues](https://github.com/juliafalcao/sim-si-la-bim/issues) tab and hopefully fixed soon. I am also working on implementing validation with a proper word list and ground-truth syllables from dictionaries to evaluate accuracy.

### disclaimer
The syllabifier is explicitly proposed for Brazilian Portuguese (BP) and not more generally for the Portuguese language because the rules for dividing syllables vary between BP and European Portuguese (EP) due to phonological variation. For example, some rising diphthongs in BP are realized as vowel hiatus in EP and thus EP separates the hiatus but not BP; the word "série", for example, would be split as "sé-rie" in BP but "sé-ri-e" in EP. For more information, Wikibooks has a [list of examples](https://pt.wikibooks.org/wiki/Portugu%C3%AAs/S%C3%ADlaba/Divis%C3%A3o).

## references

D. Silva, D. Braga e F. Resende Jr. 2008. _Separação das sílabas e determinação da tonicidade no Português Brasileiro._ XXVI Simpósio Brasileiro de Telecomunicações, páginas 1–5.

Mans Hulden. 2009. _Foma: a finite-state compiler and library._ In Proceedings of the Demonstrations Session at EACL 2009, pages 29–32, Athens, Greece. Association for Computational Linguistics
