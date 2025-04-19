
# 📚 StacksPress -  Content Smart Contract

A Clarity smart contract for decentralized curation, voting, and rewarding of community-submitted content. The system supports content submission, community appraisals (upvotes/downvotes), user credibility tracking, tipping, and moderation features, all governed by a designated protocol administrator.

---

## Features

- Submit original content with a headline, hyperlink, and topic.
- Curate content with upvotes/downvotes that affect user credibility.
- Tip (reward) content originators using STX.
- Flag inappropriate content.
- Retrieve curated items, user votes, and credibility.
- Admin-only functions for platform governance and configuration.

---

##  Constants

| Constant | Description |
|---------|-------------|
| `PROTOCOL_ADMINISTRATOR` | Default admin set to transaction sender at contract deployment. |
| `ERR_*` | Error codes for various failure scenarios. |
| `MIN_HYPERLINK_LENGTH` | Minimum acceptable hyperlink length. |
| `MAX_UINT` | Maximum representable uint value in Clarity. |

---

##  Data Variables

| Variable | Type | Description |
|----------|------|-------------|
| `submission-charge` | `uint` | STX cost required to submit content. |
| `aggregate-submissions` | `uint` | Counter for all submitted items. |
| `content-topics` | `list` | List of allowable content topics (max 10). |

---

##  Data Maps

| Map | Keys | Values | Description |
|-----|------|--------|-------------|
| `curated-items` | `{ item-identifier: uint }` | Item metadata including author, headline, URL, etc. |
| `participant-appraisals` | `{ participant: principal, item-identifier: uint }` | Tracks each user’s appraisal on specific content. |
| `participant-credibility` | `{ participant: principal }` | Accumulates credibility scores per user. |

---

## Helper Functions

| Function | Visibility | Purpose |
|----------|------------|---------|
| `item-exists` | Private | Checks if a content item exists. |
| `not-none` | Private | Checks if an optional value is `some`. |
| `retrieve-item-if-valid` | Private | Gets item if appraisal is non-negative. |
| `enumerate` | Private | Generates list of item IDs. |
| `is-non-zero` | Private | Filters out zero values from a list. |

---

##  Public Functions

###  Content Interaction

#### `contribute-item`
Submit new content.
```clojure
(contribute-item (headline) (hyperlink) (topic))
```
- Validates topic, hyperlink length, and charges `submission-charge`.
- Transfers fee to `PROTOCOL_ADMINISTRATOR`.
- Assigns unique `item-identifier`.

#### `appraise-item`
Vote (up/down) on content.
```clojure
(appraise-item (item-identifier) (appraisal))
```
- `appraisal` must be `1` (upvote) or `-1` (downvote).
- Tracks user vote and adjusts both the item’s score and user credibility.

#### `reward-originator`
Tip the content creator.
```clojure
(reward-originator (item-identifier) (gratuity-amount))
```
- Transfers STX to the original contributor.
- Updates total gratuity for the content.

#### `flag-item`
Flag content as inappropriate.
```clojure
(flag-item (item-identifier))
```
- Increments flag count unless the user is the content originator.

---

###  Read-only Functions

| Function | Description |
|----------|-------------|
| `retrieve-item-details` | Fetch metadata for a given item. |
| `retrieve-participant-appraisal` | View appraisal submitted by a user for a specific item. |
| `retrieve-aggregate-submissions` | Total number of curated items. |
| `retrieve-participant-credibility` | Check a user's credibility score. |
| `retrieve-top-items` | Return top curated content based on positive appraisals. |
| `get-item-ids` | Generate a list of item identifiers. |

---

##  Admin Functions

> All functions restricted to the `PROTOCOL_ADMINISTRATOR`.

| Function | Description |
|----------|-------------|
| `adjust-submission-charge` | Change the STX fee to submit content. |
| `expunge-item` | Delete an existing curated item. |
| `introduce-topic` | Add a new topic category to `content-topics`. |

---

##  Validations and Constraints

- Headlines, hyperlinks, and topics must be non-empty.
- Topics must exist in the `content-topics` list.
- STX balance must be sufficient for submissions and gratuities.
- Flags cannot be submitted by original content creators.
- Only administrators can modify global parameters or delete content.

---

##  Credibility System

- Users earn or lose credibility with each appraisal.
- Credibility is tracked in `participant-credibility` by summing upvotes/downvotes.

---

##  Events (Print Logs)

Logs emitted for transparency and off-chain indexing:

- `new-item`
- `appraisal`
- `reward`
- `flag`
- `fee-change`
- `item-expunged`
- `new-topic`

---

##  Example Usage

```clojure
;; Submit new item
(contribute-item "AI Breakthrough" "https://example.com/ai" "Technology")

;; Upvote item #1
(appraise-item u1 1)

;; Downvote item #2
(appraise-item u2 -1)

;; Tip content originator
(reward-originator u1 u50)

;; Admin: Adjust fee
(adjust-submission-charge u15)

;; Admin: Add new topic
(introduce-topic "Health")

;; Admin: Delete item
(expunge-item u2)
```

---
