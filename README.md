
# Personal Journal Convention for Nostr

**Status:** Informal convention  
**Scope:** Private, single-author journaling and diary entries  
**Standards used:** NIP-23, NIP-17, NIP-44  
**No new kinds. No new crypto. No relay changes.**

---

## Overview

This document defines a lightweight convention for using **Nostr as a private personal journal or diary**, where:

- Only the author can read the content
- Entries sync across devices
- Content is encrypted end-to-end
- Social clients can safely ignore it

This is **not** a social post, DM, or group message.  
It represents **personal state**, stored on public infrastructure as encrypted data.

---

## Design principles

- Use existing NIPs only
- Private by default
- Non-social intent
- Fail gracefully if ignored
- Avoid premature standardisation

This convention adds **intent signalling**, not protocol capability.

---

## Core approach

| Aspect | Choice |
| --- | --- |
| Content kind | `kind:30023` (NIP-23 long-form content) |
| Visibility | Private by default |
| Encryption | NIP-44 |
| Transport | NIP-17 gift-wrap to self |
| Replaceability | Parameterised replaceable via `d` tag |
| Intent signalling | `["intent","personal-journal"]` |

---

## Inner journal event (unencrypted)

This event is **never published directly**.  
It is encrypted and wrapped before being sent to relays.

### Required fields

- `kind`: `30023`
- `pubkey`: author pubkey
- `created_at`: unix timestamp
- `tags`:
  - `["d","<entry-id>"]`
  - `["intent","personal-journal"]`

### Optional tags

- `["title","<short title>"]`
- `["mood","<free text or enum>"]`
- `["topic","<repeatable>"]`
- `["lang","en"]`

### Content

Plaintext Markdown.

### Example (inner event)

```json
{
  "kind": 30023,
  "pubkey": "<npub>",
  "created_at": 1766012345,
  "tags": [
    ["d","2025-12-18"],
    ["intent","personal-journal"],
    ["title","Thursday wrap-up"],
    ["mood","calm"]
  ],
  "content": "Today felt slower. Cycling helped clear my head."
}
````

---

## Encryption and publishing

1. Serialize the inner event as JSON
2. Encrypt using **NIP-44**
3. Wrap using **NIP-17** (`kind:1059`)
4. Set the recipient `p` tag to **your own pubkey**
5. Publish only the wrapped event

Relays store opaque ciphertext only.

---

## Entry identity and updates

### Entry ID (`d` tag)

Recommended patterns:

* One entry per day: `YYYY-MM-DD`
* Multiple per day: `YYYY-MM-DD#<short-random>`

Because this is parameterised replaceable:

* Reposting with the same `d` replaces the prior version
* Edit history is not leaked

---

## Client behaviour guidelines

Clients that recognise this convention SHOULD:

* Hide entries from public timelines
* Disable likes, replies, reposts
* Render entries in a dedicated “Journal” or “Private Notes” view

Clients that do not recognise it will typically ignore encrypted events. Nothing breaks.

---

## Privacy considerations

What remains public:

* Timestamps
* Relay usage
* Posting frequency

What remains private:

* Content
* Titles
* Mood and topics

Deletion remains advisory only, consistent with Nostr norms.

Users requiring zero metadata leakage should not use Nostr for storage.

---

## Non-goals

This convention intentionally does **not** address:

* Group or shared journals
* Friends-only visibility
* Drafts or scheduling
* Tasks, reminders, or analytics
* New kinds or permission systems

These may build on similar patterns later.

---

## FAQ

### Why not encrypted DMs to myself?

DMs imply conversational intent and do not support clean replacement semantics.
Journals are non-social and benefit from replaceable entries.

---

### Why not create a new kind?

New kinds fragment tooling.
`kind:30023` already supports long-form, replaceable content. This convention adds intent only.

---

### Does this leak private data onto public relays?

Content is encrypted.
Metadata such as timestamps and relay usage remain visible and are explicitly documented.

---

### What if a client ignores this convention?

Nothing breaks.
The encrypted content remains private and simply is not rendered specially.

---

### Is this a NIP?

No.
This is an informal convention intended to gather real usage and feedback before any standardisation.

---

## Status

This convention is:

* Opt-in
* Ignorable
* Forward-compatible
* Immediately usable

If multiple independent clients implement it and users rely on it, formalisation can be discussed later.


