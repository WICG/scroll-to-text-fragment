# Alternative Content Types

## Introduction

The existing [scroll-to-text-fragment
spec](https://wicg.github.io/scroll-to-text-fragment/) enables links to
specific textual content within a page. However there are many kinds of
non-textual content which may also be of interest. This document explores
several use cases and proposes methods by which they may be addressed.

## Use cases

There are several content types users may be trying to view when following a
link to see some particular content. Primarily, these are:

* Text
* Images
* Videos

In addition to the [use cases presented for text
content](README.md#motivating-use-cases), there are many use cases where the
content of interest is images, video, or some element on the page.

### Image aggregation or attribution

Images are often collected from other sites with attribution (e.g.
[Wikipedia articles](https://en.wikipedia.org/),
[Pinterest](https://www.pinterest.com/),
[Microsoft Edge collections](https://support.microsoft.com/en-us/microsoft-edge/organize-your-ideas-with-collections-in-microsoft-edge-60fd7bba-6cfd-00b9-3787-b197231b507e))
and link back to the original content
page. Having the ability to scroll to the image would greatly decrease the
friction in finding that image in its original context.

### Image search engines

Image search engines often provide the ability to view the image in the context
of the original page. When the image is not at the top of the page, this results
in an inconvenient experience, where you do not even have the ability to use the
find-in-page feature since there is no way to search for an image.

Search engines could use this extension to provide a link for users to scroll to
the relevant image in the target page.

### Sharing a specific image or video

Just as with text, when referencing some rich content on a web page, it is
desirable to be able to link directly to it. It is often the case on sites with
many images or videos that it could be non-trivial to find the content of
interest after navigating.

## Principles

To enable links to non-textual content, we need to specify the content to scroll
to. Here, we follow the same principles as with textual content:

1.   Specify the content to scroll to, rather than where the content lies in the
     structure of the page.
1.   The simplest form of the specifier should work for most content and
     web pages.
1.   However, additional syntax may be necessary to work for other cases. This
     additional syntax should only be used when necessary and may not be able to
     specify contrived or manufactured examples, but should extend coverage
     considerably past the most simple syntax.
     
## Security

The issues here are analogous to those described in
[Restrictions for scroll-to-CSS-selectors](https://docs.google.com/document/d/15HVLD6nddA0OaI8Dd0ayBP2jlGw5JpRD-njAyY1oNZo/edit#heading=h.s4z585kmzt11)
and
[Text Fragment Security Issues](https://docs.google.com/document/d/1YHcl1-vE_ZnZ0kL2almeikAj2gkwCq8_5xwIae7PVik/edit#heading=h.uoiwg23pt0tx).
If an attacker can navigate or convince a user to navigate to a URL with a
"scroll-to-image" URL, and if they can determine that the page scrolled automatically
on load (or some other side effect like longer load time), they may be able to infer
the existence of the resource on the page (with enough CSS selector syntax they could
also infer arbitrary properties of the DOM, e.g., through
[CSS timing attacks](https://blog.sheddow.xyz/css-timing-attack/)).

Similar to the issues with text fragments, there may be cases where an attacker might
be able to determine the value of an attribute value. For this reason, we provide a
limited list of attributes which we'll allow matching; hence the
[Restrictions](#css-selector-restrictions) section below.

Note: We are still iterating on the potential consequences and mitigations here. The
below proposal is a vision of where we'd like to get to but the details are still
being decided.

## Proposed solution

We propose a restricted CSS selector syntax in the
[fragment directive](https://wicg.github.io/scroll-to-text-fragment/#the-fragment-directive)
of the URL. The selector syntax is severely restricted to avoid allowing selection
based on arbitrary attributes or page structure.

### Fragment Directive Syntax

Use a slightly adapted (to fragment directives) syntax from the W3C Selectors and
States Reference Note for the WebAnnotations
[CSS Selector](https://www.w3.org/TR/2017/NOTE-selectors-states-20170223/#FragmentSelector_frag).
Here is an example:

```
https://example.org#:~:selector(type=CssSelector,value=img[src$="example.org"])
```

![CSS Selector example showing two images in a mobile device frame with the second being selected with a CSS Selector.](https://user-images.githubusercontent.com/145676/109510528-69029680-7aa2-11eb-8dda-cad0648d867d.png)

The [Selectors and States as Fragment Identifiers](https://www.w3.org/TR/2017/NOTE-selectors-states-20170223/#h-frags)
section of the above Reference Note describes the functional `selector(...)` syntax
and [CSS Selector](https://www.w3.org/TR/2017/NOTE-selectors-states-20170223/#CssSelector_def)
defines specifically how CSS Selectors are defined. The same note also
[describes](https://www.w3.org/TR/2017/NOTE-selectors-states-20170223/#json-examples-converted-to-fragment-identifiers)
how to map the selectors into the fragment identifier syntax.

The proposal here is to levarage this work but implement only `type=CssSelector`
and start with interpreting only the `value` key.

The fragment directive allows these selectors to co-exist with pages that use the
fragment for routing or other reasons and is already shipped in Chrome as part of
text fragments.

Like text fragments, multiple such directives can be supplied, mixed with
text fragments or other potential future directives. E.g.

```
https://example.org#:~:text=foo&selector(type=CssSelector…)&newThing
```

The same handling as
[specified in text fragments](https://github.com/WICG/scroll-to-text-fragment#multiple-text-directives)
should be used in this case.

_Note_: Currently, the behavior is that only the first selector (from left to
right in the URL) is scrolled into view (the rest may or may not be indicated
by the UA). We may wish to amend this to scroll into view the first match in
_document order_ rather than the current _selector order_.

### CSS selector restrictions

The CSS selector specified in the `value=` key is restricted to a small subset of
the selector syntax. This prevents a potential attacker from being able to reason
about unrelated parts of a page or produce selectors with long runtimes.

Selectors that do not meet the below restrictions will be blocked and the directive
will not be invoked.

Restrictions:

* Must be a [simple](https://www.w3.org/TR/selectors/#simple) or
  [compound](https://www.w3.org/TR/selectors/#compound) selector
* Uses only the following selectors:
  * [Type](https://www.w3.org/TR/selectors/#type-selector) (i.e. element name like
    `img`, `video`, etc.)
  * [Class](https://www.w3.org/TR/selectors/#class-html)
  * [Id](https://www.w3.org/TR/selectors/#id-selectors)
  * [Attribute](https://www.w3.org/TR/selectors/#attribute-selectors)
    * Strictly limited to: `alt`, `href`, `poster`, `src`, `srcset`, `style` attributes
    * All [presence and value](https://www.w3.org/TR/selectors/#attribute-representation)
      selectors allowed (i.e. `[src]`, `[src=val]`, `[src~=val]`, `[src|=val]`)
    * All [substring matching](https://www.w3.org/TR/selectors/#attribute-substrings)
      selectors allowed (i.e. `[src^=val]`, `[src$=val]`, `[src*=val]`)
    * The [case sensitivity](https://www.w3.org/TR/selectors-4/#attribute-case) attribute
      is allowed

### Invocation restrictions

For the same required security reasons as text fragments, as well as to align with
it on the basic processing model, we suggest using the same restrictions as text
fragments (detailed in the
[spec](https://wicg.github.io/scroll-to-text-fragment/#restricting-the-text-fragment)).
In summary:

* Requires a user gesture/activation to have occurred
  * Or to have occurred and been specially passed-through a
    [client-side redirect](https://github.com/WICG/scroll-to-text-fragment/blob/master/redirects.md)
* Requires the document to be in a top-level browsing contexts (i.e. no iframes)
* Requires cross-document navigation, unless initiated by the user from the browser UI
  (i.e. no same-document navigation)
* For cross-origin navigation, requires that the browsing context be the
  [only one in its browsing context group](https://wicg.github.io/scroll-to-text-fragment/#ref-for-document-allowtextfragmentdirective⑥:~:text=If%20document%E2%80%99s%20browsing%20context%20is%20a,to%20true%20and%20abort%20these%20sub%2Dsteps.)
  (i.e. no other windows can script the document)

### Limitations

Some use cases remain difficult/impossible to select. Notably, a common pattern
is CSS background-image specified via CSS selectors
([example](https://www.tutorialspoint.com/how-to-create-a-hero-image-with-css)).
It is not clear how important/common these cases are and supporting them would either
require an expanded CSS selector syntax (based on DOM structure) or a new syntax
which would be less useful for other cases.

Our hypothesis is that most of these cases will actually have an `id` or `class`
attribute we could match on, or set the `background-image` using inline style.

## Extensions and alternatives considered

### Video timestamps

When linking to video sources, it may be desirable to specify additional properties
such as a time range to seek to or a specific track of a media element to play.
Some video services provide this capability by parsing a parameter in the URL, but
for arbitrary video sites we could allow adding
[Media Fragments](https://www.w3.org/TR/media-frags/#naming-time) to specify these
parameters for arbitrary videos. This could work by adding the `refinedBy`
capability (shown here outside a URL fragment context for clarity):

```
"selector": {
  "type": "CssSelector",
  "value": "video[src=example.mp4]",
  "refinedBy": {
    "type": "Fragment",
    "value": "t=123"
}
```

The interpretation being that an inner fragment selector of a media element be
applied to its inner resource.

Navigating to the above selector encoded in the `#:~:selector(...)` URL would not
only scroll the video into view, but also seek it to 123s.

### Content-based matching

There are cases where it may not be easy to construct a selector which is both
resilient to page layout changes while still selecting the desired content. We
could add an alternative type which would allow selecting based on the content
of the result using some form of image summarization.

This has the disadvantage that it would require loading the external resources
first before we could know whether it matches.

## FAQs

### Why use the WebAnnotations syntax?

There are a few advantages to reusing the already existing syntax offered by
WebAnnotations:

* We could decide to add more selectors in the future, either from the existing
  WebAnnotation set or new ones — this provides a well thought out and extensible
  framework.
* Some of the more advanced features may prove useful, for example, the `refinedBy`
  field. This could be used to select a video, then refine the selection using a
  media fragment to specify the seek time. A future extension could be to also allow
  the [spatial dimension](https://www.w3.org/TR/media-frags/#naming-space)
  to highlight, for example, only one particular face in a group
  picture, apart from the media fragment temporal dimension.
* The functional syntax does have some nice advantages over the `key=value` syntax in
  that it is easier to extend and nest.
* It already exists, so we don't have to reinvent the wheel.

The main downsides are that it is quite verbose and departs from the `key=value` syntax
of text fragments. We expect that CSS selectors are much less likely to be hand crafted,
so compactness is less of an issue here than in text fragments. The fact that it differs
from text fragments' syntax is unfortunate, but seems limited to aesthetic consequences.

### Why such limiting restrictions on CSS Selector?

Mainly for security reasons. See
[Scroll-To-Text Fragment Navigation Security Issues](https://docs.google.com/document/d/15HVLD6nddA0OaI8Dd0ayBP2jlGw5JpRD-njAyY1oNZo/edit).

Though the syntax is highly restricted, between this and text fragments, this
should allow users to target most kinds of content they are interested in.

Much of the CSS Selector syntax has to do with structural properties of a page which
are very powerful but may actually be harmful to the creation of resilient URLs
since structural properties of pages are more likely to change over time.

### Why not allow combinators?

We expect [combinators](https://www.w3.org/TR/selectors/#selector-combinator)
could be supported without compromising security. However, we expect this may
add more complexity than we need and may allow creation of more brittle URLs
that may break when pages change.

On the other hand, allowing combinators may allow for more resilient URLs if ancestors
of the real target have better identifying features.

We've currently left this out pending data that would indicate their necessity.

### What about ambiguous cases like the same image repeated on a page?

We are not sure how common this case is.

If this does turn out to be an issue, one potential option is to implement the
`refinedBy` field and allow restricting the selector to a subtree based on another
element's attribute. Another option could be to use the
[`:nth-of-type()` pseudo class](https://drafts.csswg.org/selectors-4/#nth-of-type-pseudo).
