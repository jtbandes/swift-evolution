# Refining Identifier and Operator Symbology

* Proposal: [SE-NNNN](NNNN-refining-identifier-and-operator-symbology.md)
* Authors: [Jacob Bandes-Storch](https://github.com/jtbandes), [Erica Sadun](https://github.com/erica), [Xiaodi Wu](https://github.com/xwu), Jonathan Shapiro
* Review Manager: TBD
* Status: **Awaiting review**

<!--
* Decision Notes: [Rationale](https://lists.swift.org/pipermail/swift-evolution/), [Additional Commentary](https://lists.swift.org/pipermail/swift-evolution/)
* Bugs: [SR-NNNN](https://bugs.swift.org/browse/SR-NNNN), [SR-MMMM](https://bugs.swift.org/browse/SR-MMMM)
* Previous Revision: [1](https://github.com/apple/swift-evolution/blob/...commit-ID.../proposals/NNNN-filename.md)
* Previous Proposal: [SE-XXXX](XXXX-filename.md)
-->

## Introduction

This proposal seeks to refine and rationalize Swift's identifier and operator symbology. Specifically, this proposal:

- adopts the Unicode recommendation for identifier characters, with some minor exceptions;
- restricts the legal operator set to the current ASCII operator characters;
- changes where dots may appear in operators; and
- disallows Emoji from identifiers and operators.


### Prior discussion threads & proposals

- [Proposal: Normalize Unicode identifiers](https://github.com/apple/swift-evolution/pull/531)
- [Unicode identifiers & operators](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160912/027108.html), with [pre-proposal](https://gist.github.com/jtbandes/c0b0c072181dcd22c3147802025d0b59) (a precursor to this document)
- [Lexical matters: identifiers and operators](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160926/027479.html)
- [Proposal: Allow Single Dollar Sign as Valid Identifier](https://github.com/apple/swift-evolution/pull/354)
- [Free the '$' Symbol!](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151228/005133.html)
- [Request to add middle dot (U+00B7) as operator character?](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151214/003176.html)


### Guiding principles

Chris Lattner has written:

> …our current operator space (particularly the unicode segments covered) is not super well considered.  It would be great for someone to take a more systematic pass over them to rationalize things.

<!-- -->

> We need a token to be unambiguously an operator or identifier - we can have different rules for the leading and subsequent characters though.

<!-- -->

> …any proposal that breaks:

>     let 🐶🐮 = "moof"

> will not be tolerated.  :-) :-)

## Motivation

By supporting custom Unicode operators and identifiers, Swift attempts to accomodate programmers and programming styles from many languages and cultures. It deserves a well-thought-out specification of which characters are valid. However, Swift's current identifier and operator character sets do not conform to any Unicode standards, nor have they been rationalized in the language or compiler documentation.

Identifiers, which serve as *names* for various entities, are linguistic in nature and **must** permit a variety of characters to properly serve non–English-speaking coders. This issue has been considered by the communities of many programming languages already, and the Unicode Consortium has published recommendations on how to choose identifier character sets — Swift should make an effort to conform to these recommendations.

Operators, on the other hand, should be rare and carefully chosen, because they suffer from low discoverability and difficult readability. They are by nature *symbols*, not names. This places a cognitive cost on users with respect to both recall ("What is the operator that applies the behavior I need?") and recognition ("What does the operator in this code do?"). While *almost every* nontrivial program defines many new identifiers, most programs do not define new operators.

As operators become more esoteric or customized, the cognitive cost rises. Recognizing a function name like `formUnion(with:)` is simpler for many programmers than recalling what the `∪` operator does. Swift's current operator character set includes many characters that aren't traditional and recognizable operators — this encourages problematic and frivolous uses in an otherwise safe language.

Today, there are many discrepancies and edge cases motivating these changes:

- · is an identifier, while • is an operator.
- The Greek question mark ; is a valid identifier.
- Braille patterns ⠟ seem letter-like, but are operator characters.
- 🙂🤘▶️🛩🂡 are identifiers, while ☹️✌️🔼✈️♠️ are operators.
- Some *non-combining* diacritics ´ ¨ ꓻ are valid in identifiers.
- Some completely non-linguistic characters, such as ۞ and ༒, are valid in identifiers.
- Some symbols such as ⚄ and ♄ are operators, despite not really being "operator-like".
- A small handful of characters 〡〢〣〤〥〦〧〨〩   〪  〫  〬  〭  〮  〯 are valid in **both** identifiers and operators.
- Some non-printing characters such as 2064 INVISIBLE PLUS and 200B ZERO WIDTH SPACE are valid identifiers.
- Currency symbols are split across operators (¢ £ ¤ ¥) and identifiers ($ ₪ € ₱ ₹ ฿ ...).

This matter should be considered in a near timeframe (Swift 3.1 or 4) as it is both fundamental to Swift and will produce source-breaking changes.

## Precedent in other languages

**Haskell** distinguishes identifiers/operators by their [general category](http://www.fileformat.info/info/unicode/category/index.htm) such as "any Unicode lowercase letter", "any Unicode symbol or punctuation", and so forth. Identifiers can start with any lowercase letter or `_`, and may contain any letter/digit/`'`/`_`. This includes letters like δ and Я, and digits like ٢.

- [Haskell Syntax Reference](https://www.haskell.org/onlinereport/syntax-iso.html)
- [Haskell Lexer](https://github.com/ghc/ghc/blob/714bebff44076061d0a719c4eda2cfd213b7ac3d/compiler/parser/Lexer.x#L1949-L1973)

**Scala** similarly allows letters, numbers, `$`, and `_` in identifiers, distinguishing by general categories `Ll`, `Lu`, `Lt`, `Lo`, and `Nl`. Operator characters include mathematical and other symbols (`Sm` and `So`) in addition to other ASCII symbol characters.

- [Scala Lexical Syntax](http://www.scala-lang.org/files/archive/spec/2.11/01-lexical-syntax.html#lexical-syntax)

**ECMAScript 2015 ("ES6")** uses `ID_Start` and `ID_Continue`, as well as `Other_ID_Start` / `Other_ID_Continue`, for identifiers.

- [ECMAScript Specification: Names and Keywords](http://www.ecma-international.org/ecma-262/6.0/#sec-names-and-keywords)

**Python 3** uses `XID_Start` and `XID_Continue`.

- [The Python Language Reference: Identifiers and Keywords](https://docs.python.org/3/reference/lexical_analysis.html#grammar-token-identifier)
- [PEP 3131: Supporting Non-ASCII Identifiers](https://www.python.org/dev/peps/pep-3131/)

## Proposed solution

For identifiers, adopt the recommendations made in [UAX #31 Identifier and Pattern Syntax](http://unicode.org/reports/tr31/), deriving the sets of valid characters from `ID_Start` and `ID_Continue`. Normalize identifiers using Normalization Form C (NFC).

(For operators, no such recommendation currently exists, although active work is in progress to update UAX #31 to address "operator identifiers".)

Restrict operators to those ASCII characters which are currently operators. All other operator characters are removed from the language.

Allow dots in operators in any location, but only in runs of two or more.

(Overall, this proposal is aggressive in its removal of problematic characters. We are not attempting to prevent the addition or re-addition of characters in the future, but by paring the set down now, we require any future changes to pass the high bar of the Swift Evolution process.)

## Detailed design

### Identifiers

Swift identifier characters will [conform to UAX #31](http://unicode.org/reports/tr31/#Conformance) as follows:

- [**UAX31-C1.**](http://unicode.org/reports/tr31/#C1) The conformance described herein refers to the Unicode 9.0.0 version of UAX #31 (dated 2016-05-31 and retrieved 2016-10-09).

- [**UAX31-C2.**](http://unicode.org/reports/tr31/#C2) Swift shall observe the following requirements:

  - [**UAX31-R1.**](http://unicode.org/reports/tr31/#R1) Swift shall augment the definition of "Default Identifiers" with the following **profiles**:

    1. `ID_Start` and `ID_Continue` shall be used for `Start` and `Continue` (replacing `XID_Start` and `XID_Continue`). This **excludes** characters in `Other_ID_Start` and `Other_ID_Continue`.

    2. _ 005F LOW LINE shall additionally be allowed as a `Start` character.
    
    3. The emoji characters 🐶 1F436 DOG FACE and 🐮 1F42E COW FACE shall be allowed as `Start` and `Continue` characters.

    4. ([**UAX31-R1a.**](http://unicode.org/reports/tr31/#R1a)) The join-control characters ZWJ and ZWNJ are strictly limited to the special cases A1, A2, and B described in UAX #31. (This requirement is covered in the [Normalize Unicode Identifiers proposal](https://github.com/apple/swift-evolution/pull/531).)

  - [**UAX31-R4.**](http://unicode.org/reports/tr31/#R4) Swift shall consider two identifiers equivalent when they have the same normalized form under [**NFC**](http://unicode.org/reports/tr15/). (This requirement is covered in the [Normalize Unicode Identifiers proposal](https://github.com/apple/swift-evolution/pull/531).)
  
[These changes](http://unicode.org/cldr/utility/unicodeset.jsp?a=%5B%5Ba-zA-Z_%5Cu00A8%5Cu00AA%5Cu00AD%5Cu00AF%5Cu00B2-%5Cu00B5%5Cu00B7-%5Cu00BA%5Cu00BC-%5Cu00BE%5Cu00C0-%5Cu00D6%5Cu00D8-%5Cu00F6%5Cu00F8-%5Cu00FF%5Cu0100-%5Cu02FF%5Cu0370-%5Cu167F%5Cu1681-%5Cu180D%5Cu180F-%5Cu1DBF%5Cu1E00-%5Cu1FFF%5Cu200B-%5Cu200D%5Cu202A-%5Cu202E%5Cu203F-%5Cu2040%5Cu2054%5Cu2060-%5Cu206F%5Cu2070-%5Cu20CF%5Cu2100-%5Cu218F%5Cu2460-%5Cu24FF%5Cu2776-%5Cu2793%5Cu2C00-%5Cu2DFF%5Cu2E80-%5Cu2FFF%5Cu3004-%5Cu3007%5Cu3021-%5Cu302F%5Cu3031-%5Cu303F%5Cu3040-%5CuD7FF%5CuF900-%5CuFD3D%5CuFD40-%5CuFDCF%5CuFDF0-%5CuFE1F%5CuFE30-%5CuFE44%5CuFE47-%5CuFFFD%5CU00010000-%5CU0001FFFD%5CU00020000-%5CU0002FFFD%5CU00030000-%5CU0003FFFD%5CU000E0000-%5CU000EFFFD%5D%5B0-9%5Cu0300-%5Cu036F%5Cu1DC0-%5Cu1DFF%5Cu20D0-%5Cu20FF%5CuFE20-%5CuFE2F%5D%5D&b=%5B%5B:ID_Continue:%5D%5CU0001F436%5CU0001F42E%5D) result in the removal of some 5,500 valid code points from the identifier characters, as well as hundreds of thousands of unassigned code points. (Though it does not appear on this unicode.org utility, which currently supports only Unicode 8 data, the · 00B7 MIDDLE DOT is no longer an identifier character.) Adopting `ID_Start` and `ID_Continue` does not add any new identifier characters.

#### Grammar changes

    identifier-head → [:ID_Start:]
    identifier-head → _ 🐶 🐮
    identifier-character → identifier-head
    identifier-character → [:ID_Continue:]

### Operators

Swift operator characters will be limited to **only** the following ASCII characters:

! % & * + - . / < = > ? ^ | ~

The current restrictions on reserved tokens and operators **will remain**: `=`, `->`, `//`, `/*`, `*/`, `.`, `?`, prefix `<`, prefix `&`,  postfix `>`, and postfix `!` are reserved.


#### Dots in operators

The current requirements for dots in operator names are:

> If an operator doesn’t begin with a dot, it can’t contain a dot elsewhere.

This proposal **changes the rule** to:

> Dots may only appear in operators in runs of two or more.

Under the revised rule, `..<` and `...` are allowed, but `<.<` is not. We also **reserve the `..` operator**, permitting the compiler to use `..` for a "method cascade" syntax in the future, as [supported by Dart](http://news.dartlang.org/2012/02/method-cascades-in-dart-posted-by-gilad.html).

Motivations for incorporating the two-dot rule are:

* It helps avoid future lexical complications arising from lone `.`s.

* It's a conservative approach, erring towards overly restrictive. Dropping the rule in future (thereby allowing single dots) may be possible.

* It doesn't require special cases for existing infix dot operators in the standard library, `...` (closed range) and `..<` (half-open range). It also leaves the door open for the standard library to add analogous half-open and fully-open range operators `<..` and `<..<`. 

* If we fail to adopt this rule now, then future backward-compatibility requirements will preclude the introduction of some potentially useful language enhancements.

#### Grammar changes
    
    operator → operator-head operator-characters[opt]
    
    operator-head → ! % & * + - / < = > ? ^ | ~
    operator-head → operator-dot operator-dots
    operator-character → operator-head
    operator-characters → operator-character operator-character[opt]

    operator-dot → .
    operator-dots → operator-dot operator-dots[opt]

### Emoji

If adopted, this proposal eliminates emoji from Swift identifiers and operators. Despite their novelty and utility, emoji characters introduce significant challenges to the language:

- Their categorization into identifiers and operators is not semantically motivated, and is fraught with discrepancies.

- Emoji characters are not displayed consistently and uniformly across different systems and fonts. Including [all Unicode emoji](http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%3AEmoji%3A%5D) introduces characters that don't render as emoji on Apple platforms without a variant selector, but which also wouldn't normally be used as identifier characters (e.g. ⏏ ▪ ▫).

- Some emoji nearly overlap with existing operator syntax: ❗️❓➕➖➗✖️

- Full emoji support necessitates handling a variety of use cases for joining characters and variant selectors, which would not otherwise be useful in most cases. It would be hard to avoid permitting sequences of characters which aren't valid emoji, or being overly restrictive and not properly supporting emoji introduced in future versions of Unicode.

As an exception, in homage to Swift's origins, we permit 🐶 and 🐮 in identifiers.


## Source compatibility

This change is source-breaking in cases where developers have incorporated emoji or custom non-ASCII operators, or identifiers with characters which have been disallowed. This is unlikely to be a significant breakage for the majority of serious Swift code.

Code using the middle dot `·` in identifiers may be slightly more common. `·` is now disallowed entirely.

Diagnostics for invalid characters are already produced today. We can improve them easily if needed.

Maintaining source compatibility for Swift 3 should be easy: just keep the old parsing & identifier lookup code.

## Effect on ABI stability

This proposal does not affect the ABI format itself, although the [Normalize Unicode Identifiers proposal](https://github.com/apple/swift-evolution/pull/531) affects the ABI of compiled modules.

The standard library will not be affected; it uses ASCII symbols with no combining characters.

## Effect on API resilience

This proposal doesn't affect API resilience.

## Alternatives considered

- Define operator characters using Unicode categories such as [`Sm` and `So`](http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%5B%3ASm%3A%5D%5B%3ASo%3A%5D%5D). This approach would include many "non-operator-like" characters and doesn't seem to provide a significant benefit aside from a simpler definition.

- Hand-pick a set of "operator-like" characters to include. The proposal authors tried this painstaking approach, and came up with a relatively agreeable set of about [650 code points](http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%21%5C%24%25%5C%26*%2B%5C-%2F%3C%3D%3E%3F%5C%5E%7C~%0D%0A%0D%0A%5Cu00AC%0D%0A%5Cu00B1%0D%0A%5Cu00B7%0D%0A%5Cu00D7%0D%0A%5Cu00F7%0D%0A%0D%0A%5Cu2208-%5Cu220D%0D%0A%5Cu220F-%5Cu2211%0D%0A%5Cu22C0-%5Cu22C3%0D%0A%5Cu2212-%5Cu221D%0D%0A%5Cu2238%0D%0A%5Cu223A%0D%0A%5Cu2240%0D%0A%5Cu228C-%5Cu228E%0D%0A%5Cu2293-%5Cu22A3%0D%0A%5Cu22BA-%5Cu22BD%0D%0A%5Cu22C4-%5Cu22C7%0D%0A%5Cu22C9-%5Cu22CC%0D%0A%5Cu22D2-%5Cu22D3%0D%0A%5Cu2223-%5Cu222A%0D%0A%5Cu2236-%5Cu2237%0D%0A%5Cu2239%0D%0A%5Cu223B-%5Cu223E%0D%0A%5Cu2241-%5Cu228B%0D%0A%5Cu228F-%5Cu2292%0D%0A%5Cu22A6-%5Cu22B9%0D%0A%5Cu22C8%0D%0A%5Cu22CD%0D%0A%5Cu22D0-%5Cu22D1%0D%0A%5Cu22D4-%5Cu22FF%0D%0A%5Cu22CE-%5Cu22CF%0D%0A%0D%0A%5Cu2A00-%5Cu2AFF%0D%0A%0D%0A%5Cu27C2%0D%0A%5Cu27C3%0D%0A%5Cu27C4%0D%0A%5Cu27C7%0D%0A%5Cu27C8%0D%0A%5Cu27C9%0D%0A%5Cu27CA%0D%0A%5Cu27CE-%5Cu27D7%0D%0A%5Cu27DA-%5Cu27DF%0D%0A%5Cu27E0-%5Cu27E5%0D%0A%0D%0A%5Cu29B5-%5Cu29C3%0D%0A%5Cu29C4-%5Cu29C9%0D%0A%5Cu29CA-%5Cu29D0%0D%0A%5Cu29D1-%5Cu29D7%0D%0A%5Cu29DF%0D%0A%5Cu29E1%0D%0A%5Cu29E2%0D%0A%5Cu29E3-%5Cu29E6%0D%0A%5Cu29FA%0D%0A%5Cu29FB%0D%0A%0D%0A%5Cu2308-%5Cu230B%0D%0A%5Cu2336-%5Cu237A%0D%0A%5Cu2395%5D) (although this set would require further refinement), but ultimately felt the motivation for including non-ASCII operators is much lower than for identifiers, and the harm to readers/writers of programs outweighs their potential utility.

- Use Normalization Form KC (NFKC) instead of NFC. The decision to use NFC comes from [Normalize Unicode Identifiers proposal](https://github.com/apple/swift-evolution/pull/531). Also, UAX #31 states:

  > Generally if the programming language has case-sensitive identifiers, then Normalization Form C is appropriate; whereas, if the programming language has case-insensitive identifiers, then Normalization Form KC is more appropriate.

  NFKC may also produce surprising results; for example, "ſ" and "s" are equivalent under NFKC.

- Continue to allow single `.`s in operators, and perhaps even expand the original rule to allow them anywhere (even if the operator does not begin with `.`).

  This would allow a wider variety of custom operators (for some interesting possibilities, see the operators in Haskell's [Lens](https://github.com/ekmett/lens/wiki/Operators) package). However, there are a handful of potential complications:

  - Combining prefix or postfix operators with member access: `foo*.bar` would need to be parsed as `foo *. bar` rather than `(foo*).bar`. Parentheses could be required to disambiguate.

  - Combining infix operators with contextual members: `foo*.bar` would need to be parsed as `foo *. bar` rather than `foo * (.bar)`. Whitespace or parentheses could be required to disambiguate.

  - Hypothetically, if operators were accessible as members such as `MyNumber.+`, allowing operators with single `.`s would require escaping operator names (perhaps with backticks, such as `` MyNumber.`+` ``).

  This would also require operators of the form `[!?]*\.` (for example `.` `?.` `!.`  `!!.`) to be reserved, to prevent users from defining custom operators that conflict with member access and optional chaining.

  We believe that requiring dots to appear in groups of at least two, while in some ways more restrictive, will prevent a significant amount of future pain, and does not require special-case considerations such as the above.


## Future directions

While not within the scope of this proposal, the following considerations may provide useful context for the proposed changes. We encourage the community to pick up these topics when the time is right.

- **Re-expand operators to allow some non-ASCII characters.** There is work in progress to update UAX #31 with definitions for "operator identifiers" — when this work is completed, it would be worth considering for Swift.

- **Introduce a syntax for method cascades.** The Dart language supports [method cascades](http://news.dartlang.org/2012/02/method-cascades-in-dart-posted-by-gilad.html), whereby multiple methods can be called on an object within one expression: `foo..bar()..baz()` effectively performs `foo.bar(); foo.baz()`. This syntax can also be used with assignments and subscripts. Such a feature might be very useful in Swift; this proposal reserves the `..` operator so that it may be added in the future.

- **Introduce "mixfix" operator declarations.** Mixfix operators are based on pattern matching, and would allow more than two operands. For example, the ternary operator `? :` can be defined as a mixfix operator with three "holes": `_ ? _ : _`. Subscripts might be subsumed by mixfix declarations such as `_ [ _ ]`. Some holes could be made `@autoclosure`, and there might even be holes whose argument is represented as an AST, rather than a value or thunk, supporting advanced metaprogramming (for instance, F#'s [code quotations](https://docs.microsoft.com/en-us/dotnet/articles/fsharp/language-reference/code-quotations)).

- **Diminish or remove the lexical distinction between operators and identifiers.** If precedence and fixity applied to traditional identifiers as well as operators, it would be possible to incorporate ASCII equivalents for standard operators (e.g. `and` for `&&`, to allow `A and B`). If additionally combined with mixfix operator support, this might enable powerful DSLs (for instance, C#'s [LINQ](https://en.wikipedia.org/wiki/Language_Integrated_Query)).
