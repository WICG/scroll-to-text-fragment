#### 2.1. What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

The targetText value that will be added to the URL fragment contains a portion of text that is expected to be in the page. This information is either already on the page, or isn't on the page but was expected to be, so shouldn't be a privacy concern.

#### 2.2. Is this specification exposing the minimum amount of information necessary to power the feature?

The feature itself does not expose information. The URL creator (e.g. a website providing a reference or a user sharing a targetText URL) adds some amount of information to the URL using this feature, but this is information expected to be in the target page. We have restrictions in place to prevent exfiltration of page contents using this feature:
- Restricted to top-level frames (no iframes)
- Restricted to non-same-page activations
- Restricted to pages without an opener (no window.open)

#### 2.3. How does this specification deal with personal information or personally-identifiable information or information derived thereof?

It doesn't - The user could theoretically place PII in the targetText, exposing it to the page, but this would imply they expect the information to appear on the page and want to link to it.

#### 2.4. How does this specification deal with sensitive information?

It doesn't - See previous question.

#### 2.5. Does this specification introduce new state for an origin that persists across browsing sessions?

No.

#### 2.6. What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

None.

#### 2.7. Does this specification allow an origin access to sensors on a user’s device

No.

#### 2.8. What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

Just the targetText value as addressed in question 1.

#### 2.9. Does this specification enable new script execution/loading mechanisms?

No.

#### 2.10. Does this specification allow an origin to access other devices?

No.

#### 2.11. Does this specification allow an origin some measure of control over a user agent’s native UI?

Similar to an element fragment anchored URL, the origin can cause the UA to change scroll position on navigation. We propose restricting the feature to non-same-doc full navigations, so an origin can't use this to manipulate the scroll position on the current page.

#### 2.12. What temporary identifiers might this this specification create or expose to the web?

None.

#### 2.13. How does this specification distinguish between behavior in first-party and third-party contexts?

Third-party contexts cannot access the URL fragment, so there should be no way to access the targetText. The most a cross-origin iframe can check is document.referrer, but document.referrer does not include the URL fragment.

#### 2.14. How does this specification work in the context of a user agent’s Private \ Browsing or "incognito" mode?

The feature works the same in a private browsing context, and does not reveal private browsing mode or provide information to correlate private browsing activity.

#### 2.15. Does this specification have a "Security Considerations" and "Privacy Considerations" section?

Yes.

#### 2.16. Does this specification allow downgrading default security characteristics?

No.
