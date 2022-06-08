## Overview

The current language :lang pseudo-class implementation is based on the [Selectors Level 3](https://www.w3.org/TR/2018/REC-selectors-3-20181106/#lang-pseudo) 
and accepts only one identifier as an argument for matching the element's language value.

[Level 4](https://www.w3.org/TR/selectors-4/#the-lang-pseudo) specs proposes a change in this behavior to accepting a comma-separated argument list of
[Language-Ranges](https://www.rfc-editor.org/rfc/rfc4647.html#section-2) and performs the [extended filtering operation](https://www.rfc-editor.org/rfc/rfc4647.html#section-3.3.2) 
to match language values.

## Details

From the level 3 specs:

>The pseudo-class :lang(C) represents an element that is in language C.
>Whether an element is represented by a :lang() selector is based solely on the element's language value
>(normalized to BCP 47 syntax if necessary)
>being equal to the identifier C, or beginning with the identifier C immediately followed by "-" (U+002D).

This means that that only elements where its lang value is equal to C or C is a *prefix* of its lang value are matched.

For example:

```
p:lang(de-DE) {
  background-color: yellow;
}
```

would match

```
<p lang="de-DE">lang=de-DE</p>
<p lang="de-DE-x-goethe">lang=de-DE-x-goethe</p>
```

but not 

```
<p lang="de-Latn-DE">lang=de-Latn-DE</p>
<p lang="de-Latn-DE-1996">lang=de-Latn-DE-1996</p>
<p lang="de-Deva-DE">lang=de-Deva-DE</p>
```

The main change on matching logic in Level 4 specs is that now those would also be matched, as the matching logic now
changes from a simple prefix matching to an [extended filtering operation](https://www.rfc-editor.org/rfc/rfc4647.html#section-3.3.2) where
an implicicit wildcard matching occurs (meaning that de-DE is the same as de-*-DE).

Also, according to the new specs, instead of an identifier, :lang now takes an argument list. From the [new specs](https://www.w3.org/TR/selectors-4/#the-lang-pseudo):

>The :lang() pseudo-class represents an element that is in one of the languages listed in its argument.
>It accepts a comma-separated list of one or more language ranges as its argument.
>Each language range in :lang() must be a valid CSS \<ident\> or \<string\>.

So, now the following is going to be possible:

```
p:lang(en, fr, de) {
  background-color: yellow;
}
```

instead of forcing the developer to write:

```
p:lang(en, fr, de) {
  background-color: yellow;
}

p:lang(fr) {
  background-color: yellow;
}

p:lang(de) {
  background-color: yellow;
}

```

simplifying writing rules for matching multiple languages.

## Issues with current tests

After the [proposed CL](https://chromium-review.googlesource.com/c/chromium/src/+/3515958), the following test fails: [third_party/blink/web_tests/external/wpt/css/selectors/i18n/css3-selectors-lang-014.html](https://source.chromium.org/chromium/chromium/src/+/084f128497aa57134c6fbc40d7715cfaf3298251:third_party/blink/web_tests/external/wpt/css/selectors/i18n/css3-selectors-lang-014.html)

This happens because of the previous expectations of prefix matching and it's already [solved on the above CL](https://chromium-review.googlesource.com/c/chromium/src/+/3515958/9/third_party/blink/web_tests/external/wpt/css/selectors/i18n/css4-selectors-lang-001.html), and will work on [Safari](https://webkit.org/status/#feature-css-selector-:lang()-level-4) but will fail on current Firefox. I plan to prepare a change for Firefox browser as well.
