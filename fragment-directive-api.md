# Fragment Directive API

## Current Status

As of Oct 29, 2021: The API described below is available for experimentation in Chrome 97.0.4685.0 and newer behind a flag. Use `--enable-blink-features=TextFragmentAPI` to turn it on (or chrome://flags/#enable-experimental-web-platform-features which turns on all experimental features).

## Introduction

This document proposes a programmatic API through which authors can interact with text (and future) directives.

Today, when a page is loaded with a text directive such as `https://example.org#:~:text=foo,bar`, the author has no way[^1] to tell that a text directive was set or what text was highlighted. The fragment directive portion of the URL (everything in the fragment after and including `:~:`) is stripped from the URL when the document is loaded. This is done for two reasons:

1. _Compatibility_ - Some pages assume the fragment will always be of an expected form or entirely absent. Without stripping the fragment directive, these pages may break with a user-supplied directive feature.

2. _Privacy_ - Some directives may contain data that shouldn't be visible to page script. This isn't a concern for text directives since the directive will only contain content already on the page (and the page can tell where it's scrolled to). However, as an example, the [proposed](https://github.com/bokand/web-annotations/blob/main/URL-based-annotation.md) note directive uses the fragment directive to allow users to share comments with a friend. In that case, the destination page should not have access to the content.

Providing a structured API allows the browser to expose enough information and functionality to enable authors to extend and customize how different directives behave without violating either of the above goals.

[^1]: As noted in https://crbug.com/1096983, this is accidentally exposed via the performance API. This is a bug that we'd like to fix but some use cases are currently relying on this.

## Use cases

* Attach comments/responses to specific parts of text on a page or provide helpful UI - e.g. [Marginalia](https://indieweb.org/marginalia)

* Enable pages to easily create text directive links. The rules for how text is matched are [necessarily complicated](https://wicg.github.io/scroll-to-text-fragment/#find-a-range-from-a-text-directive); they must consider word boundaries, DOM node display types and visibility, and various nuances of how DOM is traversed. This API allows an author to let the browser generate a valid text directive URL for a given Range.

* Enable text directives in cross-origin iframes. To prevent [XS-Search attacks](https://wicg.github.io/scroll-to-text-fragment/#example-4d0b486d:~:text=A%20malicious%20page%20embeds%20a%20cross%2Dorigin%20victim%20in%20an%20iframe,its%20own%20document.), text directives are not applied when navigated from a cross-origin initiator. However, an iframe can navigate itself to a text directive. By allowing an embedder page to read the text directive, it can `postMessage()` it to a cross-origin document that's opted-in to this behavior, enabling deep linking to the inner frame (see examples section below).

## WebIDL

This is the IDL as implemented behind a flag in Chrome.

```WebIDL
// The following build on the existing but empty document.fragmentDirective
// See https://wicg.github.io/scroll-to-text-fragment/#feature-detectability

// === Current ===

[Exposed=Window]
interface FragmentDirective {
};

partial interface Document {
    [SameObject] readonly attribute FragmentDirective fragmentDirective;
};

// === Changes/Additions ===

[Exposed=Window]
interface FragmentDirective {
  // Array of parsed Directive objects, one for each term in the fragment
  // directive (i.e. currently, each `text=` term)
  readonly attribute FrozenArray<Directive> items;

  // TODO: add(Directive)?

  // Creates a SelectorDirective object that can be used to select the given
  // range/selection.
  Promise<SelectorDirective> createSelectorDirective(Range or Selection);
 };

enum DirectiveType { "text" };

// Interface common to all future Directive types.
[Exposed=Window]
interface Directive {
  readonly attribute DirectiveType type;
  DOMString toString();
  // TODO: remove()?
}

// Interface common to all selector Directive types (i.e. those that
// scroll/indicate some sub-portion of the document).
[Exposed=Window]
interface SelectorDirective : Directive {
  Promise<Range> getMatchingRange();
}

dictionary TextDirectiveOptions {
    DOMString prefix;
    DOMString textStart;
    DOMString textEnd;
    DOMString suffix;
};

// TODO: [Serializable]
[Exposed=Window]
interface TextDirective : SelectorDirective {
  constructor(TextDirectiveOptions);
  // TODO: constructor(DOMString directive_string);
  readonly attribute DOMString prefix;
  readonly attribute DOMString textStart;
  readonly attribute DOMString textEnd;
  readonly attribute DOMString suffix;
};
```

Why a `SelectorDirective` base-class, in addition to `Directive`? The [proposed](https://github.com/WICG/scroll-to-text-fragment/blob/main/EXTENSIONS.md#proposed-solution) CSS selector directive would behave very similarly to a text directive and allows `createSelectorDirective()` to return a `SelectorDirective`. OTOH, the proposed [note selector](https://github.com/bokand/web-annotations/blob/main/URL-based-annotation.md) would not fit this interface.

_TODO: Maybe `SelectorDirective` is unnecessary? Callers could always determine the directive type using `Directive.type` if they need to. Also, it may actually make sense for `note` to provide `getMatchingRange()`...)_

## Examples

### Marginalia-like use cases:

```JS
// Coming from a server-side WebMention:
const comment_text = "Great Point!";
const comment_url = "https://example.org/post.html#:~:text=My%20point";

const directive_string = extractTextDirective(comment_url); // "My%20point";

const directive = new TextDirective(directive_string);
const range = await directive.getMatchingRange();
attachCommentUI(comment_text, range);
```

### Generate a link for the user's selection

```JS
document.onselectionchange = () => {
  const selection = document.getSelection();
  const text_directive =
      await document.fragmentDirective.createSelectorDirective(selection);
  shareButton.onclick = () => {
    const url = `${window.location.href}#:~:${text_directive.toString()}`;
    navigator.clipboard.writeText(url);
  };
};
```

### Forward a text directive across origins

```JS
// Embedder document
const text_directives =
    document.fragmentDirectives.items.filter(i => i.type === "text"));

const message = {
  type: 'text-directives',
  directives: text_directives;
}

frames[0].postMessage(message);
```

In the cross-origin document:

```JS
//Embedee document
window.onmessage = (e) => {
  if (e.type === 'text-directives') {
    const strings = e.directives.map(i => i.toString());
    window.location.hash = `:~:${strings.join('&')}`;
  }
});
```

_TODO: setting `location.hash` isn't great. Consider adding `fragmentDirective.add(Directive)` and adding a `Directive.remove()`._

## FragmentDirective.items

The `items` array reflects the currently active directives on the page. Using text directives as an example, an entry should exist in `items` for a text directive as long a highlight is showing. If the user dismisses the highlight, it is removed from the array. Conversely, if the directive is removed from `items` programmatically (see next section), the highlight should be removed from the page.

## FragmentDirective as part of location.hash

Currently, script can add a directive by writing to `location.hash`:

```JS
location.hash = ":~:text=foo%20bar";
```

The snippet above will add a text directive to the page, highlighting "foo bar" and adding a `TextDirective` to `fragmentDirective.items`. However, this still runs the [fragment directive stripping steps](https://wicg.github.io/scroll-to-text-fragment/#process-and-consume-fragment-directive):

```JS
const value = ":~:text=foo%20bar";
location.hash = value;
console.log(location.hash);  // Output: ""
```

This is rather unintuitive and surprising.

There's also the question of what happens to existing directives in `fragmentDirective.items` when the hash is modified. In the cases below, suppose the user navigated to `https://example.org/blog.html#:~:text=acme`.

1. What should happen when script sets a hash with no fragment directive? (e.g. `location.hash = 'page1';`).
2. What should happen when script sets a hash with an unrelated directive? (e.g. `location.hash = ':~:note(href=notes.example.org)';`)
3. What should happen when script sets a hash with a text directive? (e.g. `location.hash = ':~:text=blog%20title';`)

That is, are changes to `location.hash` additive or do they replace existing directives?

For case 1, we almost certainly shouldn't affect existing directives as this would violate the _compatibility_ goal from the introduction. Pages often write to their hash for various reasons, these shouldn't interfere with user-supplied directives. That is, a page modifying its hash shouldn't remove text highlights.

For case 2, it also seems like we shouldn't remove the text directive. Directives of different types should behave independently. That is, adding an annotation to a page shouldn't clear text highlights.

In case 3, either behavior could work: a new highlight should be added and the existing one kept OR the new highlight replaces all existing ones. Though, if additive, it means there's no way to remove directives.

Another consideration: _Given that a page can add new directives, there should be a way to remove existing ones_. Using `location.hash` for this will necessarily lead to violating our intuition for how at least one of the above cases works.

### Proposed Behavior

Managing directives using `location.hash` leads to complicated, difficult-to-explain behaviors. Let's remove fragment directive processing from `location.hash` and add explicit APIs for doing this. E.g.

```JS
// Add a directive to the page
document.fragmentDirective.add(new TextDirective("foo%20bar"));
document.fragmentDirective.add(new TextDirective("second%20highlight"));

// Remove a directive
document.fragmentDirective.items[1].remove();
// Perhaps document.fragmentDirective.clear()?
```

Setting location.hash affects only the part of the fragment that isn't the fragment directive. E.g.

```JS
location.hash = ':~:text=foo%20bar';
console.log(location.hash);  // Output: "%3A%7E%3Atext=foo%20bar"
```

That is, setting a directive delimiter in `location.hash` percent-encodes it so that it doesn't turn into fragment directive.

The same behavior is used whenever a same-document navigation occurs:

```JS
console.log(location.href);  // Output: "https://example.com";
location = "https://example.com#:~:text=foo%20bar";
console.log(location.href):  // Output: "https://example.com%3A%7E%3Atext=foo%20bar"
```

In spec language: fragment directive processing from the URL occurs only when [navigating across documents](https://html.spec.whatwg.org/#navigating-across-documents).
