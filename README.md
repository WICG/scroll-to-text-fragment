# Scroll-To-Text using a URL fragment

## Introduction

To enable users to easily navigate to specific content in a web page, we propose adding support for specifying a text snippet in the URL fragment. When navigating to a URL with such a fragment, the browser will find the first instance of the text snippet in the page and bring it into view.

Web standards currently specify support for scrolling to anchor elements with name attributes, as well as DOM elements with ids, when [navigating to a fragment](https://html.spec.whatwg.org/multipage/browsing-the-web.html#scroll-to-fragid). While named anchors and elements with ids enable scrolling to limited specific parts of web pages, not all documents make use of these elements, and not all parts of pages are addressable by named anchors or elements with ids.

## Current Status

This feature is currently implemented as an experimental feature in Chrome 74.0.3706.0 and newer. It is not yet shipped to users by default. Users who wish to experiment with it can use chrome://flags#enable-text-fragment-anchor.

## A Note on Specifications / "Why Not Use The Standardization Process?"

Our intent is definitely for this to be part of the standards process, interoperable with other browsers, with feedback and ideas from the broader community. This document is meant to serve as an explainer to that end and to serve as a __starting point__ for those discussions.

Likewise, the experimental implementation is used to prove the viability of the concept, help us iterate on ideas, and help inform design and standards work. Once we're satisfied that we understand the space sufficiently, this work will move into the appropriate standardization forum.

## Motivating Use Cases

When following a link to read a specific part of a web page, finding the relevant part of the document after navigating can be cumbersome. This is especially true on mobile, where it can be difficult to find specific content when scrolling through long articles or using the browser's "find in page" feature. Fewer than 1% of clients use the "Find in Page" feature in Chrome on Android.

To enable scrolling directly to a specific part of a web page, we propose generalizing the existing support for scrolling to elements based on the fragment identifier. We believe this capability could be used by a variety of websites (e.g. search engine results pages, Wikipedia reference links), as well as by end users when sharing links from a browser.

### Search Engines

Search engines, which link to pages that contain content relevant to user queries, would benefit from being able to scroll users directly to the part of the page most relevant to their query.

For example, Google Search currently links to named anchors and elements with ids when they are available.  For the query "lincoln gettysburg address sources", Google Search provides a link to the named anchor [#Lincoln’s_sources](https://en.wikipedia.org/wiki/Gettysburg_Address#Lincoln's_sources) for the [wikipedia page for Gettysburg Address](https://en.wikipedia.org/wiki/Gettysburg_Address) as a "Jump to" link:

![Example "Jump to" link in search results](jumpto.png)

However, there are many pages with relevant sub passages with no named anchor or id, and search engines cannot provide a "Jump to" link in such cases.

### Referencing / sharing a specific passage in a web page

When referencing a specific section of a web page, for example as part of sharing that content via email or on social media, it is desirable to be able to link directly to the specific section. If a section is not linkable by a named anchor or element with id, it is not currently possible to share a link directly to a specific section. Users may work around this by sharing screenshots of the relevant portion of the document (preventing the recipient of the content from engaging with the actual web page that hosts the content), or by including extra instructions to scroll to a specific part of the document (e.g. "skip to the sixth paragraph"). We would like to enable users to link to the relevant section of a document directly. Linking directly to the relevant section of a document preserves attribution, and allows the user following the URL to engage directly with the original publisher.

## Proposed Solution

We propose generalizing [existing support](https://html.spec.whatwg.org/multipage/browsing-the-web.html#find-a-potential-indicated-element) for scrolling to elements as part of a navigation by adding support for specifying a text snippet in the URL fragment. We modify the [indicated part of the document](https://html.spec.whatwg.org/multipage/browsing-the-web.html#the-indicated-part-of-the-document) processing model to allow using a text snippet as the indicated part. The user agent would then follow the existing logic for [scrolling to the fragment identifier](https://html.spec.whatwg.org/multipage/browsing-the-web.html#scroll-to-the-fragment-identifier) as part of performing a navigation.

This extends the existing support for scrolling to anchor elements with name attributes, as well as DOM elements with ids, to scrolling to other textual content on a web page. Browsers first attempt to find an element that matches the fragment using the existing support for elements with id attributes and anchor elements with name attributes. If no matches are found, browsers then attempt to parse the fragment as a text snippet specification.

### Encoding the text snippet in the URL fragment

We propose encoding a text snippet in the URL fragment, prefixed with the ```targetText=``` string. Since text can contain characters invalid in a URL (e.g. spaces), and some characters are reserved by the fragment syntax (e.g. ',') the text must be percent encoded. For example, ```#targetText=My%20Heading``` would cause the first occurance of "My Heading" on the page to be selected as the indicated part of the document. The currently required characters to be percent encoded are ' ' (%20) and ',' (%2C).

Though existing HTML support for id and name attributes specifies the target element directly in the fragment, most other mime types make use of this x=y pattern in the fragment, such as [Media Fragments](https://www.w3.org/TR/media-frags/#media-fragment-syntax) (e.g. #track=audio&t=10,20), [PDF](https://tools.ietf.org/html/rfc3778#section-3) (e.g. #page=12) or [CSV](https://tools.ietf.org/html/rfc7111#section-2) (e.g. #row=4).

### Extended Syntax

There are interesting use cases where the specified text may be quite long, potentially one or more entire paragraphs. Specifying such long snippets in a URL can be cumbersome and unwieldy. To avoid exploding URL lengths, this syntax can be easily extended to allow specifying both a start and end piece. This effectively allows a simple form of wildcarding such that it will match the first text block of the form ```startPiece.*endPiece```. To specify the text snippet in this way, simply provide two arguments to targetText, separated by a comma. Example:

```
www.example.com#targetText=the%20lazy%20dog,brown%20fox
```

Would match "The lazy dog jumped over the quick brown fox".

### URL fragment encoding

In order to specify a text snippet in the fragment of a web page URL, the author must encode the selector in a way that produces a valid URL fragment.

The [URL standard](https://url.spec.whatwg.org/) specifies that a fragment can contain [URL code points](https://url.spec.whatwg.org/#url-code-points), as well as [UTF-8 percent encoded characters](https://url.spec.whatwg.org/#utf-8-percent-encode). Characters in the [fragment percent encode set](https://url.spec.whatwg.org/#fragment-percent-encode-set) must be percent encoded.

### Visual Indicators

In addition to scrolling, the user agent may wish to emphasize the targetted text snippet to the user to help direct their attention. This can be done by providing a visual cue, such as highlighting the targetted snippet.

Users may wish to customize the behavior of the visual indicator. Two examples we've come across are:

- Turning off highlighting - Users may wish to scroll some non-text element into view (e.g. image). If the element doesn't have an id/name, users may use the surrounding text instead. Highlighting the text could be misleading in this case.
- Multiple highlights - Users may wish to highlight multiple items on the page, for instance, multiple bullet points in a list.

While we consider these useful additions, these capabilities are out of scope for our initial proposal, save that we'd like to leave the syntax open to possible future extensions like this.

## Alternatives Considered

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

Another avenue of exploration is allowing users to specify highlighting in more detail. This was touched on briefly in the sections above. There are cases where the user may wish to highlight multiple snippets of text. For technical reasons, a text match across block-level elements may be difficult for a browser to implement. Allowing the user to specify multiple highlights would allow highlighting multiple paragraphs or bullet points. There are also cases where the user may wish to prevent highlights altogether, as in the image search case described above.

We've thought about these cases insofar as making sure our proposed solution doesn't preclude these enhancements in the future. However, the work of actually realizing them will be left for future iterations of this effort.

## Additional Considerations

### Browser compatibility and fallback behavior

A browser that doesn’t yet support this feature will attempt to match the specified text snippet in the URL fragment using the existing logic to find a [potential indicated element](https://html.spec.whatwg.org/multipage/browsing-the-web.html#find-a-potential-indicated-element). A fragment-encoded text selector is prefixed with ‘targetText=’, which is unlikely to appear in an id or name attribute, so we do not expect a matching element to be found in these cases. Thus, browsers that do not support this feature should fall back to the default behavior of not scrolling the document.

### Web Compatibility

Web pages could potentially be using the fragment to store parameters, e.g. `http://example.com/#name=test`. If sites are already using ```targetText``` in the URL fragment for their own purposes, or if they don't handle unexpected tokens, this feature could break those sites. In particular, some frameworks use the fragment for routing.

We should investigate how common these use cases are and what the failure modes are. If unexpecteded parameters are simply ignored, this would be a compatible change. If they cause error pages or visible failures, this would be a cause for concern.

We should also measure how likely a naming collision would be using [HTTPArchive](https://httparchive.org/). i.e. Check to see if `targetText` is available enough to use as a reserved token.

### Security

Care must be taken when implementing this feature so that it cannot be used to exfiltrate information across origins. Since an attacker can navigate a page to a cross-origin URL fragment, if they can determine that the victim page scrolled, they can infer the existence of any text on the page.

A related attack is possible if the existence of a match takes significantly more or less work than non-existence. An attacker can navigate to a fragment and time how busy the JS thread is; a high load may imply the existence or non-existence of an arbitrary text snippet. This is a variation of a documented [proof-of-concept](https://blog.sheddow.xyz/css-timing-attack/).

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

### Relation to existing support for navigating to a fragment

Browsers currently support scrolling to elements with ids, as well as anchor elements with name attributes. This proposal is intended to extend this existing support, to allow navigating to additional parts of a document. As Shaun Inman [notes](https://shauninman.com/archive/2011/07/25/cssfrag) (in support of CSS selector fragments), this feature is "not meant to replace more concise, author-designed urls" using id attributes, but rather "enables a site’s users to address specific sub-content that the site’s author may not have anticipated as being interesting".

## Related Work / Additional Resources

### Using CSS Selectors as Fragment Identifiers

Simon St. Laurent and Eric Meyer [proposed](http://simonstl.com/articles/cssFragID.html) using CSS Selectors as fragment identifiers (last updated in 2012). Their proposal differs only in syntax used: St. Laurent and Meyer proposed specifying the CSS selector using a ```#css(...)``` syntax, for example ```#css(.myclass)```. This syntax is based on the XML Pointer Language (XPointer) Framework, an "extensible system for XML addressing" ... "intended to be used as a basis for fragment identifiers". XPointer does not appear to be supported by commonly used browsers, so we have elected to not depend on it in this proposal.

[Shaun Inman](https://shauninman.com/archive/2011/07/25/cssfrag) and others later implemented browser extensions using this #css() syntax for Firefox, Safari, Chrome, and Opera, which shows that it is possible to implement this feature across a variety of browsers.

Scroll Anchoring

* [https://drafts.csswg.org/css-scroll-anchoring/](https://github.com/WICG/ScrollAnchoring/blob/master/explainer.md)
* [https://docs.google.com/document/d/1YaxJ0cxFADA_xqUhGgHkVFgwzf6KXHaxB9hPksim7nc/edit](https://docs.google.com/document/d/1YaxJ0cxFADA_xqUhGgHkVFgwzf6KXHaxB9hPksim7nc/edit)

Scroll to text

* [http://zesty.ca/crit/draft-yee-url-textsearch-00.txt](http://zesty.ca/crit/draft-yee-url-textsearch-00.txt)
* [https://indieweb.org/fragmention](https://indieweb.org/fragmention)

Other

* [https://en.wikipedia.org/wiki/Fragment_identifier#Examples](https://en.wikipedia.org/wiki/Fragment_identifier#Examples)
* [https://www.w3.org/TR/2017/REC-annotation-model-20170223/](https://www.w3.org/TR/2017/REC-annotation-model-20170223/)
