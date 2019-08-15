# Scroll-To-Text using a URL fragment directive

## Introduction

To enable users to easily navigate to specific content in a web page, we propose adding support for specifying a text snippet in the URL. When navigating to such a URL, the browser will find the first instance of the text snippet in the page and bring it into view.

Web standards currently specify support for scrolling to anchor elements with name attributes, as well as DOM elements with ids, when [navigating to a fragment](https://html.spec.whatwg.org/multipage/browsing-the-web.html#scroll-to-fragid). While named anchors and elements with ids enable scrolling to limited specific parts of web pages, not all documents make use of these elements, and not all parts of pages are addressable by named anchors or elements with ids.

### Current Status

This feature is currently implemented as an experimental feature in Chrome 74.0.3706.0 and newer. It is not yet shipped to users by default. Users who wish to experiment with it can use chrome://flags#enable-text-fragment-anchor. The implementation is incomplete and doesn't necessarily match the specification in this document.

### A Note on Specifications / "Why Not Use The Standardization Process?"

Our intent is definitely for this to be part of the standards process, interoperable with other browsers, with feedback and ideas from the broader community. This document is meant to serve as an explainer to that end and to serve as a __starting point__ for those discussions.

Likewise, the experimental implementation is used to prove the viability of the concept, help us iterate on ideas, and help inform design and standards work. Once we're satisfied that we understand the space sufficiently, this work will move into the appropriate standardization forum.

### Motivating Use Cases

When following a link to read a specific part of a web page, finding the relevant part of the document after navigating can be cumbersome. This is especially true on mobile devices, where it can be difficult to find specific content when scrolling through long pages or using the browser's "find in page" feature. Fewer than 1% of clients use the "Find in Page" feature in Chrome on Android.

To enable scrolling directly to a specific part of a web page, we propose generalizing the existing support for scrolling to elements based on the fragment identifier. We believe this capability could be used by a variety of websites (e.g. search engine results pages, Wikipedia reference links), as well as by end users when sharing links from a browser.

#### Search Engines

Search engines, which link to pages that contain content relevant to user queries, would benefit from being able to scroll users directly to the part of the page most relevant to their query.

For example, Google Search currently links to named anchors and elements with ids when they are available.  For the query "lincoln gettysburg address sources", Google Search provides a link to the named anchor [#Lincoln’s_sources](https://en.wikipedia.org/wiki/Gettysburg_Address#Lincoln's_sources) for the [wikipedia page for Gettysburg Address](https://en.wikipedia.org/wiki/Gettysburg_Address) as a "Jump to" link:

![Example "Jump to" link in search results](jumpto.png)

However, there are many pages with relevant passages with no named anchor or id, and search engines cannot provide a "Jump to" link in such cases.

#### Citations / Reference links

Links are sometimes used as citations in web pages where the author wishes to substantiate a claim by referencing another page (e.g. references in wikipedia). These reference pages can often be large, so finding the exact passage that supports the claim can be very time consuming. By linking to the passage that supports their underlying claim, authors can make it more efficient for readers to follow their overall argument.

#### Sharing a specific passage in a web page

When referencing a specific section of a web page, for example as part of sharing that content via email or on social media, it is desirable to be able to link directly to the specific section. If a section is not linkable by a named anchor or element with id, it is not currently possible to share a link directly to a specific section. Users may work around this by sharing screenshots of the relevant portion of the document (preventing the recipient of the content from engaging with the actual web page that hosts the content), or by including extra instructions to scroll to a specific part of the document (e.g. "skip to the sixth paragraph"). We would like to enable users to link to the relevant section of a document directly. Linking directly to the relevant section of a document preserves attribution, and allows the user following the URL to engage directly with the original publisher.

## Proposed Solution

### tl;dr

Allow specifying text to scroll and highlight in the URL:

https://example.com##targetText=prefix-,startText,endText,-suffix

Using this syntax

```
##targetText=[prefix-,]textStart[,textEnd][,-suffix]

              context  |-------match-----|  context
```
_(Square brackets indicate an optional parameter)_

Navigating to such a URL will find the first instance of the specified targetText, surrounded by the (optionally provided) prefix and suffix, and scroll it into view during a navigation. The snippet will be highlighted using a mechanism similar to the browser’s Find-In-Page feature.

This will work only if the user provided a gesture, for a full “new-page” navigation, and be disabled in iframes. All text matching will be performed on word boundaries for security reasons (where possible).

The targetText is delimited by a double-hash to indicate that it is a _fragment directive_ that the user agent should process and then remove from the URL fragment that is exposed to the site. This solves the problem of sites relying on the URL fragment for routing/state, see [issue #15](https://github.com/WICG/ScrollToTextFragment/issues/15). This also allows the URL fragment to still contain an element ID that can be scrolled into view in case there's no targetText match found:

https://en.wikipedia.org/wiki/Cat#Characteristics##targetText=Claws-,Like%20almost,the%20Felidae%2C,-cats

Note that the delimiter for the targetText is still an open question. The hash symbol is currently not a valid [URL code point](https://url.spec.whatwg.org/#url-code-points), and thus would require an amendment to the URL spec.

### Background

We propose generalizing [existing support](https://html.spec.whatwg.org/multipage/browsing-the-web.html#find-a-potential-indicated-element) for scrolling to elements as part of a navigation by adding support for specifying a text snippet in the URL. We modify the [indicated part of the document](https://html.spec.whatwg.org/multipage/browsing-the-web.html#the-indicated-part-of-the-document) processing model to allow using a text snippet as the indicated part. The user agent would then follow the existing logic for [scrolling to the fragment identifier](https://html.spec.whatwg.org/multipage/browsing-the-web.html#scroll-to-the-fragment-identifier) as part of performing a navigation.

This extends the existing support for scrolling to anchor elements with name attributes, as well as DOM elements with ids, to scrolling to other textual content on a web page. Browsers first attempt to find an element that matches the fragment using the existing support for elements with id attributes and anchor elements with name attributes. If no matches are found, browsers then will process the text snippet specification.

The Open Annotation specification already specifies a [TextQuoteSelector](https://www.w3.org/TR/annotation-model/#text-quote-selector). This proposal has been made similar to the TextQuoteSelector in hopes that we can extend and reuse that processing model rather than inventing a new one, albeit with a stripped down syntax for ease of use in a URL.

### Additional Considerations

 * Highlighting - In addition to scrolling the targeted snippet into view, the browser should highlight it to the user.
 * Multiple highlights - It is desirable to be able to highlight multiple snippets on a page.
 * Cross-element matching - The user may wish to highlight text that spans multiple paragraphs, list items, or table entries. In cases where possible, we should allow specifying text snippets across these boundaries.
 * Non-uniqueness of text - The text to be highlighted may not be unique on the page. The solution must account for this by allowing ways to disambiguate matches on a page.

### Identifying a Text Snippet
Specify a text snippet that should be scrolled into view on page load:

https://en.wikipedia.org/wiki/Cat##targetText=Claws-,Like%20almost,the%20Felidae%2C,-cats

```
##targetText=[prefix-,]textStart[,textEnd][,-suffix]

              context  |-------match-----|  context
```
_(Square brackets indicate an optional parameter)_

Though existing HTML support for id and name attributes specifies the target element directly in the fragment, most other mime types make use of this x=y pattern in the fragment, such as [Media Fragments](https://www.w3.org/TR/media-frags/#media-fragment-syntax) (e.g. #track=audio&t=10,20), [PDF](https://tools.ietf.org/html/rfc3778#section-3) (e.g. #page=12) or [CSV](https://tools.ietf.org/html/rfc7111#section-2) (e.g. #row=4).

The _targetText_ keyword will identify a block of text that should be scrolled into view. The provided text is be percent-decoded before matching. Dash (-), ampersand (&), and comma (,) characters in text snippets must be percent-encoded to avoid being interpreted as part of the targetText syntax.

The [URL standard](https://url.spec.whatwg.org/) specifies that a fragment can contain [URL code points](https://url.spec.whatwg.org/#url-code-points), as well as [UTF-8 percent encoded characters](https://url.spec.whatwg.org/#utf-8-percent-encode). Characters in the [fragment percent encode set](https://url.spec.whatwg.org/#fragment-percent-encode-set) must be percent encoded.

There are two kinds of terms specified in the targetText value: the _match_ and the _context_. The match is the portion of text that’s to be scrolled to and highlighted. The context is used only to disambiguate the match and is not highlighted.

Context is optional, it need not be provided. However, the targetText must always specify a match term.

#### Match
A match can be specified either with one argument or two.

If the match is provided using two arguments, the left argument is considered the starting snippet and the right argument is considered the ending snippet (e.g. targetText=_startText_,_endText_). In this case, the browser will perform a search for a block of text that starts with _startText_ and ends with _endText_. If multiple blocks match the first in DOM order is chosen (i.e. find the first occurence of startText, from there find the next occurence of endText). When a match is specified with two arguments, we allow highlighting text that spans multiple elements.

If the match is specified as a single argument, we consider it an exact match search (e.g. targetText=_textSnippet_). The browser will highlight the first occurence of exactly the _textSnippet_ string. In this case, the text cannot span multiple elements.

<table><tr><td>
E.g. Given:
            
 * Text1
 * Text2
 * Text3
 * Text4

`targetText=Text2,Text4` will highlight all items except the first:

* Text1
* __Text2__
* __Text3__
* __Text4__

`targetText=Text2` will highlight just the second item:

* Text1
* __Text2__
* Text3
* Text4

</td></tr></table>

#### Context
To disambiguate non-unique snippets of text on a page, targetText arguments can specify optional _prefix_ and _suffix_ terms. If provided, the match term will only match text that is immediately preceded by the _prefix_ text and/or immediately followed by the _suffix_ text (allowing for an arbitrary amount of whitespace in between). Immediately preceded, in these cases, means there are no other text nodes between the match and the context term in DOM order. There may be arbitrary whitespace and the context text may be the child of a different element (i.e. searching for context crosses element boundaries).

If provided, the prefix must end (and suffix must begin) with a dash (-) character. This is to disambiguate the prefix and suffix in the presence of optional parameters. It also leaves open the possibility of extending the syntax in the future to allow multiple context terms, allowing more complicated context matching across elements.

If provided, the prefix must be the first argument to targetText. Similarly, the suffix must be the last argument to targetText.

<table><tr><td>
            
For example, suppose we want to perform the following highlight:

* header1
  * text1
* __header1__
  * text2
  * text3

Since the text “header1” is ambiguous, we must provide a suffix to disambiguate it:

`targetText=header1,-text2`

</td></tr></table>


<table><tr><td>
            
Here’s an example where we need both pieces of context:

Interstellar
* Christopher Nolan Director

Inception
* __Christopher Nolan__ Director

Inception
* Christopher Nolan Writer

`targetText=Inception-,Christopher Nolan,-Director`

_Note: The space in “Christopher Nolan” would have to be percent-encoded since spaces aren’t valid in a URL. However, most browsers will do this for you automatically._

</td></tr></table>

<table><tr><td>

If the snippet is unique enough, we could provide no context:

Here is a __Superduper unique__ string 

`targetText=Superduper unique`

</td></tr></table>

### Processing Model

We'd like to reuse as much of the processing model - how the text search is performed, how white space is collapsed, etc. - in the Web Annotation’s TextQuoteSelector as possible, potentially adding enhancements to that specification. Notably, we’d need to add the ability to specify text using a starting and ending snippet to TextQuoteSelector.

If we cannot find a match that meets all the requirements in the targetText arguments, no scrolling or highlighting is performed. 

If an attacker can determine if a page has scrolled, this feature could be used to detect the presence of arbitrary text on the page. To prevent brute force attacks to guess important words on a page (e.g. passwords, pin codes), matches and prefix/suffix will only be matched on word boundaries. E.g. “range” will match “mountain range” but not “color orange” nor “forest ranger”.  Word boundaries are simple in languages with spaces but can become more subtle in languages without breaks (e.g. Chinese). A library like ICU [provides support](http://userguide.icu-project.org/boundaryanalysis#TOC-Word-Boundary) for finding word boundaries across all supported languages based on the Unicode Text Segmentation standard. Some browsers already allow word-boundary matching for the window.find API which allows specifying wholeWord as an argument. We hope this existing usage can be leveraged in the same way.

### Highlight
The UA will highlight the passage of text specified by targetText. Use cases exist for more precise control over the highlight, ex:
 * Highlight but don’t scroll into view
 * Scroll but don’t highlight

We won’t support these features in the initial version but would like to leave the option open for future extension.

We allow highlighting multiple snippets by providing additional targetTexts in the _fragment directive_, separated by the ampersand (&) character. Each targetText is considered independent in the sense that failure to find a match in one does not affect highlighting of any other targetTexts. e.g.:

```
example.com##targetText=foo&targetText=bar&targetText=bas
```

will highlight “foo”, “bar”, and “baz” and scroll “foo” into view, assuming all appear on the page.

Only the left-most, successfully matched, targetText will be scrolled-into-view and used as the CSS target. That is, if “foo” did not appear anywhere on the page but “bar” does, we scroll “bar” into view.

Each target text will start searching from the top of the page independently so that we may allow highlighting a snippet above the one that was scrolled into view.

### :target

For element-id based fragments (e.g. https://en.wikipedia.org/wiki/Cat#Anatomy), navigation causes the identified element to receive the `:target` CSS pseudo-class. This is a nice feature as it allows the page to add some customized highlighting or styling for an element that’s been targeted. For example, note that navigating to a citation on a Wikipedia page highlights the citation text: https://en.wikipedia.org/wiki/Cat#cite_note-Linaeus1758-1

The question is how we should treat the `:target` CSS pseudo-class with the targetText fragment directive.
An element-id fragment will target a complete and unique Element on the page. A text snippet may only be a portion of the text in an Element.

We can start with setting :target on the parent Element of the first (i.e. the match we scroll to) match. If we find this causes ambiguity or confusion we can simply avoid setting `:target`. 

## Alternatives Considered

### targetText 0.1

A prior revision of this document contained a somewhat similar proposal. The main difference in the updated proposal is that it adds context terms to targetText. This helps to allow disambiguating text on a page as well as brings this proposal more in-line with the Open Annotation's [TextQuoteSelector](https://www.w3.org/TR/annotation-model/#text-quote-selector). Many use cases and details were considered while iterating on the initial revision. The updated proposal is a sum of lessons learned and improved understanding as we experimented with and considered the initial version and its limitations

### CSS Selector Fragments

Our initial idea, explored in some detail, was to allow encoding a CSS selector in the URL fragment. The selector would determine which element on the page should be the "indicated element" in the [navigating to a fragment](https://html.spec.whatwg.org/multipage/browsing-the-web.html#scroll-to-fragid) steps. In fact, this explainer is based on @bryanmcquade's original [CSS Selector Fragment explainer](https://github.com/bryanmcquade/scroll-to-css-selector).

The main drawback with this approach was making it secure. Allowing scroll on load to a CSS selector allows several ways an attacker could exfiltrate hidden information (e.g. CSRF tokens) from the page. One such attack is demonstrated [here](https://blog.sheddow.xyz/css-timing-attack/) but others were quickly discovered as well.

Trying to pare down the allowable set of primitives to make selectors secure turned out to be quite complex. Text snippets, which can be searched asynchronously and are generally less security sensitive, became our preferred solution. As an additional bonus, we expect text snippets to be more stable and easier to understand by non-technical users.

### Increase use of elements with named anchors / id attributes in existing web pages

As an alternative, we could ask web developers to include additional named anchor tags in their pages, and reference those new anchors. There are two issues that make this less appealing. First, legacy content on the web won’t get updated, but users consuming that legacy content could still benefit from this feature. Second, it is difficult for web developers to reason about all of the possible points other sites might want to scroll to in their pages. Thus, to be most useful, we prefer a solution that supports scrolling to any point in a web page.

### JavaScript-based API (instead of URL fragment)

We also considered specifying the target element via a JavaScript-based navigation API, such as via a new parameter to location.assign(). It was concluded that such an API is less useful, as it can only be used in contexts where JavaScript is available. Sharing a link to a specific part of a document is one use case that would not be possible if the target element was specified via a JavaScript API. Using a JavaScript API is also less consistent than existing cases where a scroll target is specified in a URL, such as the existing support in HTML, as well as support for other document formats such as PDF and CSV.

## Future Work

One important use case that's not covered by this proposal is being able to scroll to an image. A nearby text snippet can be used to scroll to the image but it depends on the page and is indirect. We'd eventually like to support this use case more directly.

A potential option is to consider this just one of many available [Open Annotation selectors](https://www.w3.org/TR/annotation-model/#selectors). Future specification and implementation work could allow using selectors other than TextQuote to allow targetting various kinds of content.

Another avenue of exploration is allowing users to specify highlighting in more detail. This was touched on briefly in the sections above. There are cases where the user may wish to highlight multiple snippets of text. For technical reasons, a text match across block-level elements may be difficult for a browser to implement. Allowing the user to specify multiple highlights would allow highlighting multiple paragraphs or bullet points. There are also cases where the user may wish to prevent highlights altogether, as in the image search case described above.

We've thought about these cases insofar as making sure our proposed solution doesn't preclude these enhancements in the future. However, the work of actually realizing them will be left for future iterations of this effort.

## Additional Considerations

### Web and Browser Compatibility

As noted in [issue #15](https://github.com/WICG/ScrollToTextFragment/issues/15), web pages could potentially be using the fragment to store parameters, e.g. `http://example.com/#name=test`. If sites don't handle unexpected tokens when processing the fragment, this feature could break those sites. In particular, some frameworks use the fragment for routing. This is solved by the user agent hiding the ##targetText part of the fragment from the site, but browsers that do not have this feature implemented would still break such sites.

For pages that don't process the fragment, a browser that doesn't yet support this feature will attempt to process the fragment and _fragment directive_ (i.e. ##targetText) using the existing logic to find a [potential indicated element](https://html.spec.whatwg.org/multipage/browsing-the-web.html#find-a-potential-indicated-element). If a fragment exists in the URL alongside the _fragment directive_, the browser may not scroll to the desired fragment due to the confusion with parsing the _fragment directive_.  If a fragment does not exist alongside the _fragment directive_, the browser will just load the page and won't initiate any scrolling.  In either case, the browser will just fall back to the default behavior of not scrolling the document.

### Security

Care must be taken when implementing this feature so that it cannot be used to exfiltrate information across origins. Note that an attacker can navigate a page to a cross-origin URL with a targetText _fragment directive_. If they can determine that the victim page scrolled, they can infer the existence of any text on the page.

A related attack is possible if the existence of a match takes significantly more or less work than non-existence. An attacker can navigate to a targetText _fragment directive_ and time how busy the JS thread is; a high load may imply the existence or non-existence of an arbitrary text snippet. This is a variation of a documented [proof-of-concept](https://blog.sheddow.xyz/css-timing-attack/).

For these reasons, we've determined a set of restrictions to ensure an attacker cannot use this feature to exfiltrate arbitrary information from the page:

- Restrict feature to user-gesture initiated navigations
    - This prevents programmatic, repeated guesses which could be used to probe a user's data without any interaction from the user 
- Limit the feature to top-level browsing contexts (i.e. no iframes)
    - This prevents an attacker from embedding and hiding a sensitive page which can be probed
    - The attacker can still open a new window using `window.open`, however, this is more visible to the user and less practical on mobile devices
- Limit feature to full (non-same-page) navigations
    - The noise of a full navigation mitigates the timing attacks somewhat 
- Match only on word boundaries
    - Prevents an attacker from repeatedly probing to determine a sensitive word, e.g. "Password: a", "Password: ad, etc."
    - “quick brown fox” would be matched from either “quick” or “quick brown” but neither “ick brown” nor “quick brow” would match.
    - Not all languages have visible word boundaries, for example, Chineese. We'll follow the rules set out in [Unicode Standard Annex #29](http://www.unicode.org/reports/tr29/#Word_Boundaries) supplemented by a word dictionary as done by the ICU Project's [boundary analysis](http://userguide.icu-project.org/boundaryanalysis)
- For highlighting - a visual-only indicator should be used (i.e. don’t cause selection), styled by the UA
    - Prevents drag-and-drop or copy-paste attacks

While these may seem overly-restrictive, we believe they don't impede the main use-cases. We'd like to start off cautious and re-examine if interesting but blocked use-cases arise.

### Privacy

While the feature itself does not expose privacy related information, the targetText value may contain sensitive information. This is why the _fragment directive_ is designed so the user agent processes and then removes the _fragment directive_, so that the site does not have access to this information.

### Relation to existing support for navigating to a fragment

Browsers currently support scrolling to elements with ids, as well as anchor elements with name attributes. This proposal is intended to extend this existing support, to allow navigating to additional parts of a document. As Shaun Inman [notes](https://shauninman.com/archive/2011/07/25/cssfrag) (in support of CSS selector fragments), this feature is "not meant to replace more concise, author-designed urls" using id attributes, but rather "enables a site’s users to address specific sub-content that the site’s author may not have anticipated as being interesting".

## Related Work / Additional Resources

### Using CSS Selectors as Fragment Identifiers

Simon St. Laurent and Eric Meyer [proposed](http://simonstl.com/articles/cssFragID.html) using CSS Selectors as fragment identifiers (last updated in 2012). Their proposal differs only in syntax used: St. Laurent and Meyer proposed specifying the CSS selector using a ```#css(...)``` syntax, for example ```#css(.myclass)```. This syntax is based on the XML Pointer Language (XPointer) Framework, an "extensible system for XML addressing" ... "intended to be used as a basis for fragment identifiers". XPointer does not appear to be supported by commonly used browsers, so we have elected to not depend on it in this proposal.

[Shaun Inman](https://shauninman.com/archive/2011/07/25/cssfrag) and others later implemented browser extensions using this #css() syntax for Firefox, Safari, Chrome, and Opera, which shows that it is possible to implement this feature across a variety of browsers.

The [Open Annotation Community Group](https://www.w3.org/community/openannotation/) aims to allow annotating arbitrary content. There is significant overlap in our goal of specifying a snippet of text in a resource. Our work has been informed specifically by prior efforts at selecting arbitrary textual content for an annotation.

Scroll Anchoring

* [https://drafts.csswg.org/css-scroll-anchoring/](https://github.com/WICG/ScrollAnchoring/blob/master/explainer.md)
* [https://docs.google.com/document/d/1YaxJ0cxFADA_xqUhGgHkVFgwzf6KXHaxB9hPksim7nc/edit](https://docs.google.com/document/d/1YaxJ0cxFADA_xqUhGgHkVFgwzf6KXHaxB9hPksim7nc/edit)

Scroll to text

* [https://indieweb.org/fragmention](https://indieweb.org/fragmention)
* [http://zesty.ca/crit/draft-yee-url-textsearch-00.txt](http://zesty.ca/crit/draft-yee-url-textsearch-00.txt)
* [http://1997.webhistory.org/www.lists/www-talk.1995q1/0284.html](http://1997.webhistory.org/www.lists/www-talk.1995q1/0284.html)

Other

* [https://en.wikipedia.org/wiki/Fragment_identifier#Examples](https://en.wikipedia.org/wiki/Fragment_identifier#Examples)
* [https://www.w3.org/TR/2017/REC-annotation-model-20170223/](https://www.w3.org/TR/2017/REC-annotation-model-20170223/)
