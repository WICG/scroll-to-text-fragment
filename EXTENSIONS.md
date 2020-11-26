# Alternate content types

## Introduction

The existing [scroll-to-text-fragment
spec](https://github.com/flackr/scroll-to-text-fragment.git) enables links to
specific textual content within a page, however there are many kinds of
non-textual content which may also be of interest. This document explores
several use cases and proposes methods by which they may be addressed.

## Use cases

There are several content types users may be trying to view When following a
link to see some particular content. Primarily, these are:

* Text
* Images
* Videos

In addition to the [use cases presented for text
content](README.md#motivating-use-cases), there are many use cases where the
content of interest is images, video, or some element on the page.

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

## Proposed solution

Allow specifying a limited form of selector as part of the URL fragment using
the same delimiter established by scroll-to-text:

https://example.com#:~:selector=img[url="example.png"]

Navigating to the above URL would be roughly equivalent to inserting the
following script:

```js
document.addEventListener('DOMContentLoaded', () => {
  document.querySelector('img[url="example.png"]').scrollIntoView();
});
```

However, browsers would be free to emphasize the selected element in some way as
to direct the user's attention to it.

## Extensions and alternatives

### Video timestamps

When linking to video sources it may be desirable to specify a timestamp to seek
to. Some video services provide this capability by parsing a parameter in the
URL, but for arbitrary video sites we could allow adding a seek parameter to
seek to a given timestamp in the video. For example:

https://example.com#:~:selector=video[url="movie.mp4"],seek=123

Navigating to the above URL would not only scroll the video in to view, but also
seek it to 123s.

### Content-based matching

There are cases where it may not be easy to construct a selector which is both
resilient to page layout changes while still selecting the desired content. We
could add an alternative type which would allow selecting based on the content
of the result using some form of image summarization.

This has the disadvantage that it would require loading the external resources
first before we could know whether it matches.
