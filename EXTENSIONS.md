# Alternate content types

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

Images are often collected from other sites with attribution (e.g. Wikipedia
articles, Pinterest, Edge collections) and link back to the original content
page. Having the ability to scroll to the image would greatly decrease the
friction in finding that image in its original context.

### Image search engines

Image search engines often provide the ability to view the image in the context
of the original page. When the image is not at the top of the page this results
in an inconvenient experience, where you do not even have the ability to use the
find-in-page feature since there is no way to search for an image.

Search engines could use this extension to provide a link for users to scroll to
the relevant image in the target page.

### Sharing a specific image or video

Just as with text, when referencing some rich content on a web page it is
desirable to be able to link directly to it. It is often the case on sites with
many images or videos that it could be non-trivial to find the content of
interest after navigating.

## Principles

To enable links to non-textual content, we need to specify the content to scroll
to. Here we follow the same principles as with textual content:

1.   Specify the content to scroll to, rather than where the content lies in the
     structure of the page.
1.   The simplest form of the specifier should work for most content and
     web pages.
1.   However, additional syntax may be necessary to work for other cases. This
     additional syntax should only be used when necessary and may not be able to
     specify contrived or manufactured examples, but should extend coverage
     considerably past the most simple syntax.

## Proposed solution

Allow specifying a limited form of selector as part of the URL fragment using
the same delimiter established by scroll-to-text:

https://example.com#:~:selector=img[src="example.png"]

Navigating to the above URL would cause the browser to indicate the first
instance of the matched element. The exact details of what a browser should do
once it finds the match are beyond the scope of this proposal. However, browsers
would be free to emphasize the selected element in some way as to direct the
user's attention to it.

As with the scroll-to-text fragment directive, the user agent should process and
remove the directive from the URL fragment which is exposed to the page.

## Extensions and alternatives

### Video timestamps

When linking to video sources it may be desirable to specify additional
properties such as a time range to seek to or a specific track of a media
element to play. Some video services provide this capability by parsing a
parameter in the URL, but for arbitrary video sites we could allow adding [Media
Fragments](https://www.w3.org/TR/media-frags/#media-fragment-syntax) to specify
these parameters for arbitrary videos. For example:

https://example.com#:~:selector=video[src="movie.mp4%23t=123"]

Navigating to the above URL would not only scroll the video in to view, but also
seek it to 123s (`#` is percent-encoded as `%23`).

### Content-based matching

There are cases where it may not be easy to construct a selector which is both
resilient to page layout changes while still selecting the desired content. We
could add an alternative type which would allow selecting based on the content
of the result using some form of image summarization.

This has the disadvantage that it would require loading the external resources
first before we could know whether it matches.
