<!--
© 2018, Devin Smith <dsmith@polysync.io>, Nathan Äschbacher <naschbacher@polysync.io>

This file is part of Rx

Rx is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

Rx is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with Rx.  If not, see <http://www.gnu.org/licenses/>.
-->

# Rx Specification

- [Overview](#overview)
- [Usage](#usage)
  * [Inline Elements](#inline-elements)
  * [Inline Examples](#inline-examples)
    + [Matching Literals](#matching-literals)
    + [Matching Mandatory tokens](#matching-mandatory-tokens)
    + [Matching Optional Tokens](#matching-optional-tokens)
    + [Matching Links](#matching-links)
  * [Block Elements](#block-elements)
  * [Block Examples](#block-examples)
    + [Matching Mandatory Block Level tokens](#matching-mandatory-block-level-tokens)
  * [Special Cases](#special-cases)
    + [Inline vs Block Level tokens](#inline-vs-block-level-tokens)
    + [Fenced Code Blocks](#fenced-code-blocks)
    + [HTML Comments](#html-comments)
  * [Special Case Examples](#special-case-examples)
    + [Matching inline tokens as the first element of a block element](#matching-inline-tokens-as-the-first-element-of-a-block-element)
    + [Matching Block Level Tokens for Fenced Code blocks](#matching-block-level-tokens-for-fenced-code-blocks)
    + [Matching Block Elements with HTML comments.](#matching-block-elements-with-html-comments)

## Overview

Rx is a DSL that extends Markdown for the purpose of defining a document in
terms of constraints on its structure and content. An Rx file can then be used
to validate the conformance of other plain markdown documents to the generic
document definition.

[CommonMark](https://spec.commonmark.org/0.28/) is a formal specification of
Markdown and is the specific standard that Rx extends. A valid CommonMark file
is also a valid Rx file and in the context of Rx, describes a document that must
match identically. In addition to this **Literal** type of specification, Rx
provides the three tokens `-!!-`, `-??-`, and `-""-` which indicate **Mandatory**,
**Optional**, and **Repeatable** respectively. CommonMark conceptualizes a markdown
document as consisting of block and inline elements and the exact meaning of the Rx
tokens is dependent on the type of Markdown element it is used in conjunction
with.

## Usage

*
    ### Inline Elements

    Inline elements consist of
    * Plain, textual content such as that within paragraphs and headings
    * Inline code blocks
    * Emphasis and Strong Emphasis
    * Links
    * Images
    * Inline HTML

    Rx tokens that appear in the context of inline elements represent prompts for
    arbitrary textual content to appear according to token type.

    * **Literal**
        Any textual content that is not an Rx token or a syntactic element of
        Markdown must be matched with an identical section of text.

    * `-!!-`
        **Mandatory**

        Some content must be inserted here. Each mandatory token must match with at
        least one non-whitespace characters, so adjacent mandatory tokens must be
        matched with content whose length is greater than or equal to the number of
        tokens.

    * `-??-`
        **Optional**

        Some content may be inserted here, but is not necessary for the document to
        be considered valid.

    * `-""-`
        **Repeatable**

        Has no signifigance in this context. Is not considered a valid usage of this
        token.

    ### Inline Examples

    #### Matching Literals

    ##### Rx

    ``` markdown
    # A Heading

    Some content.
    ```

    ##### Matches

    ``` markdown
    # A Heading

    Some content.
    ```

    ##### Rejects

    ``` markdown
    Literally anything else.
    ```

    #### Matching Mandatory tokens

    ##### Rx

    ``` markdown
    # We're off to see -!!-

    The -!!- of -!!-.
    ```

    ##### Matches

    ``` markdown
    # We're off to see the wizard

    The wonderful wizard of Oz.
    ```

    ``` markdown
    # We're off to see the veterinarian

    The priciest doctor of dogs.
    ```

    ``` markdown
    # We're off to see the optometrist

    The optometrist will tell us if we are in need of glasses.
    ```

    ##### Rejects

    ``` markdown
    # We're off to see

    The dentist because our teeth are full of cavities.
    ```

    > In this case, the Mandatory token in the heading has not been satisfied.

    #### Matching Optional Tokens

    ##### Rx

    ``` markdown
    Seeking a -??-qualified applicant for our remote -??- offices.
    ```

    ##### Matches

    ``` markdown
    Seeking a woefully unqualified applicant for our remote customer support offices.
    ```

    ``` markdown
    Seeking a qualified applicant for our remote offices.
    ```

    ##### Rejects

    ``` markdown
    Seeking a marginally qualified applicant for our remote call center.
    ```
    > An inline Optional token cannot by itself cause a mismatch since it can match
    > any textual content or nothing. In this case the mismatch is triggered by the
    > Literal content `offices` being omitted.

    #### Matching Links

    ##### Rx

    ``` markdown
    Please [visit](https:://-!!-.polysync.io) us on the web!
    ```

    ##### Matches

    ``` markdown
    Please [visit](https:://www.polysync.io) us on the web!
    ```

    ``` markdown
    Please [visit](https:://www.rx.polysync.io) us on the web!
    ```

    ##### Rejects

    ``` markdown
    Please [visit](https:://.polysync.io) us on the web!
    ```

    > Rejected because there was no prefix specified in the uri.

    ``` markdown
    Please [visit](http:://scam.polysync.io) us on the web!
    ```

    > Rejected because the `s` was omitted from the protocol portion of the uri.


*
    ### Block Elements

    Block elements contain inline elements and in some cases other block elements.
    They consist of
    * List Items
    * Block Quotes
    * Paragraphs
    * Headings
    * Code Blocks
    * HTML Blocks

    If the first textual content of a block element begins with an Rx token, then
    that token is considered a 'Block Level Token' and its meaning applies to the
    entire scope of that block.

    * **Literal**
        Strictly speaking, block elements do not directly contain Literals since all
        textual content is conceptualized as being within inline elements. That
        being said, the Literal content appearing within block elements must be
        matched identically just like in the rule for inline Literal elements, but
        the requirement of its presence or absence can be specified the presence of
        the following Block Level Tokens.

    * `-!!-`
        **Mandatory**

        - Element contains additional inline content or block elements.
            A block element of the same type **must** appear in this position in the
            matching document. In order to be considered a match, all inline and and
            block elements contained within the scope of this element must also be
            matched according to their relevant matching rules.
        - Element does not contain any additional content.
            A block element of the same type **must** appear in this position in the
            document however, the matching element may contain any arbitrary inline
            and/or block elements.

    * `-??-`
        **Optional**

        - Element contains additional inline content or block elements.
            A block element of the same type **may** appear in this position in the
            matching document but is not necessary. If an element does appear, in
            order to be considerer a match, all inline and and block elements
            contained within the scope of this element must also be matched
            according to their relevant matching rules.
        - Element does not contain any additional content.
            A block element of the same type **may** appear in this position in the
            document, but is not necessary. If an element does appear, it may
            contain any arbitrary inline and/or block elements and still be
            considered a match.

    * `-""-`
        **Repeatable**

        This token must appear in a block element that does not contain any other
        block or inline elements and whose type matches the previous
        block element in the document. It matches any number of block elements that
        match the previous block element.

    ### Block Examples

    #### Matching Mandatory Block Level tokens

    ##### Rx

    ``` markdown
    # -!!- The wonderful thing about tiggers

    -!!- Is tiggers are wonderful things!
    ```

    ##### Matches

    ``` markdown
    # The wonderful thing about tiggers

    Is tiggers are wonderful things!
    ```

    ##### Rejects

    ``` markdown
    The wonderful thing about tiggers

    Is tiggers are wonderful things!
    ```
    > Rejected because although the text of the first block matches, it is not
    > part of the same type of block element.

    ``` markdown
    # The wonderful thing about tiggers

    Is tiggers are wonderful things!

    Their tops are made out of rubber

    Their bottoms are made out of springs!
    ```
    > Rejected because of superflous paragraphs included at the end.


    #### Matching Optional Block Level tokens

    ##### Rx

    ``` markdown
    Four score and seven years ago our fathers brought forth on this continent a new
     nation, conceived in Liberty, and dedicated to the proposition that all men are
     created equal.

    -??- Now we are engaged in a great civil war, testing whether that nation or any
    nation so conceived and so dedicated, can long endure....

    -??- But, in a larger sense, we can not dedicate—we can not consecrate—we can
    not hallow—this ground....

    -??- TLDR; -!!-
    ```

    ##### Matches

    ``` markdown
    Four score and seven years ago our fathers brought forth on this continent a new
     nation, conceived in Liberty, and dedicated to the proposition that all men are
     created equal.

    Now we are engaged in a great civil war, testing whether that nation or any
    nation so conceived and so dedicated, can long endure....

    But, in a larger sense, we can not dedicate—we can not consecrate—we can
    not hallow—this ground....
    ```

    ``` markdown
    Four score and seven years ago our fathers brought forth on this continent a new
     nation, conceived in Liberty, and dedicated to the proposition that all men are
     created equal.

    TLDR; We must finish what we have started my dudes.
    ```

    ##### Rejects

    ``` markdown
    Four score and seven years ago our fathers brought forth on this continent a new
     nation, conceived in Liberty, and dedicated to the proposition that all men are
     created equal.

    TLDR;
    ```
    > Optional Block Level Tokens can only cause a mismatch if they are present, but
    > contain some other inline or block elements that exhibit a mismatch. In this
    > case, the Mandatory inline token was not satisfied.

    #### Combining Block Level and Inline tokens

    ##### Rx

    ``` markdown
    A literal intro paragraph.

    -!!- A mandatory paragraph about -!!-.

    -""-

    -??- An optional closing paragraph that must say this if present.
    ```

    ##### Matches

    ``` markdown
    A literal mandatory intro paragraph.

    A mandatory paragraph about turnips.

    A mandatory paragraph about bacon.

    An optional closing paragraph that must say this if present.
    ```

    ``` markdown
    A literal intro paragraph.

    A mandatory paragraph about superheroes.
    ```

    ##### Rejects

    ``` markdown
    A literal intro paragraph.
    ```

    ``` markdown
    A verbatim intro paragraph.

    An optional closing paragraph that must say this if present.
    ```

    #### Matching Repeatable tokens

    ##### Rx

    ``` markdown
    * First Item
    * -""-
    * Last Item
    ```

    ##### Matches

    ``` markdown
    * First Item
    * Second Item
    * Third Item
    * Last Item
    ```

    ``` markdown
    * First Item
    * Last Item
    ```

*
    ### Special Cases

    #### Inline vs Block Level tokens

    In order to specify a prompt for some inline content at the
    beginning of a paragraph, in order for the token to not be interpreted as a
    block element token, a block element token must precede it.

    #### Fenced Code Blocks

    Due to the special formatting of Fenced Code Blocks, their block element tokens
    must be placed at the end of their info strings.

    #### HTML Comments

    HTML comments that appear as the last inline content in a block element will be
    ignored with respect to matching rules.

    ### Special Case Examples

    #### Matching inline tokens as the first element of a block element

    ##### Rx

    ``` markdown
    -!!- is my favorite color.
    ```
    > **Wrong**. The token is interpreted as a block element token.

    ##### Matches

    ``` markdown
     is my favorite color.
    ```

    ##### Rejects

    ``` markdown
    Red is my favorite color.
    ```

    ##### Rx

    ``` markdown
    -!!--!!- is my favorite color.
    ```
    > **Correct**. The first token is interpreted as a block element token, while the second
    > is interpreted as a prompt for content.

    ##### Matches

    ``` markdown
    Red is my favorite color.
    ```

    ##### Rejects

    ``` markdown
    is my favorite color.
    ```

    #### Matching Block Level Tokens for Fenced Code blocks

    ##### Rx

    ``` markdown
    ~~~ -!!- rust
    -!!-
    ~~~
    ```
    > **Wrong**. The language info for the code block will not be interpreted
    > correctly.

    ##### Rx

    ``` markdown
    ~~~ rust -!!-
    -!!-
    ~~~
    ```
    > **Correct**. Indicates a mandatory code block with rust syntax.

    ##### Accepts

    ``` markdown
    ~~~ rust
    struct Foo {
        bar: u32,
        baz: u32
    };
    ~~~
    ```

    #### Matching Block Elements with HTML comments.

    ##### Rx

    ``` markdown
    # The Fresh -!!- of -!!- <!--Your title and locale-->

    Now this is a story all about how -!!- <!--Tell what happened to your life-->
    ```

    ##### Matches

    ``` markdown
    # The Fresh Prince of Bel-Air

    Now this is a story all about how my life got flipped-turned upside down
    ```

    ##### Rejects

    ``` markdown
    # The Fresh Prince of Bel-Air <!--Your title and locale-->

    Now this is a story all about how my life got flipped-turned upside down <!--Tell what happened to your life-->
    ```
    > Rejected because the comment in the Rx file is ignored, so the comment in the
    > document will be treated as superfluous content.

# License

© 2018, Devin Smith <dsmith@polysync.io>, Nathan Äschbacher <naschbacher@polysync.io>

[GPL version 3](https://github.com/PolySync/rx/LICENSE)
