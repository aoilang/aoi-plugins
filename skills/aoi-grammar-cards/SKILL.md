---
name: aoi-grammar-cards
description: Notice meaningful grammar errors in the user's English, save them as 小葵（Aoi） grammar cards with the plugin tools, and show the correction in the reply. Use in ordinary conversations when the user writes English; ignore minor typos and stylistic preferences.
---

# 小葵（Aoi） Grammar Cards

Carry out the user's request as usual. Treat English feedback as a small add-on, never as a reason to skip, redirect, or replace the requested work.

## Save the cards first

When the user writes in English, check their own wording for clear, meaningful
grammar mistakes. Do not correct minor typos, punctuation nits, acceptable
variants, or choices that are only stylistic. Do not treat quoted text, code, or
text supplied for editing as evidence of the user's own English unless they are
clearly presenting it as their writing.

As your first action, before planning or answering the main request, call
`create_english_learning_cards` with every issue you found, grouped by source
sentence, so they are saved while the task is still running. That call renders
the panel from the cards you pass, before the save returns, so do not also call
`show_english_learning_cards` for the same cards. This applies to every message
that contains the learner's own English, including one-line replies: do not wait
to be asked for feedback. The same panel is available
from the host sidebar and thread panel when those surfaces exist.

### Grouping

- A **corrected sentence** is one self-complete source sentence and its fully
  corrected text. All issues in that sentence belong to the same sentence.
- A message can contain several independent sentences. Send each as a separate
  `sentences` entry; only put issues from the same source sentence together.
- A self-complete clause can be its own source when it stands independently.
  Do not split a single sentence merely because it contains commas.
- At most four cards per message, ordered by where each issue appears in the
  learner's text.

### Payload

Pass a `sentences` array. Each entry has:

- `corrected`: the whole source sentence. Wrap only the words you remove in
  `~~double tildes~~` and only the words that replace them in `**double
  asterisks**`. Include every correction for that source sentence, not just one
  issue. Keep each unchanged word once, without markup. When the correction only
  adds words, use bold alone; when it only removes words, use strikethrough alone.
- `source_text`: the learner's exact source sentence without markup.
- `language`: the learner's native language, e.g. `Chinese`.
- `cards`: one entry per grammar issue in that sentence:
  - `title`: the grammar pattern in the learner's native language followed by
    the English grammar term, e.g. `主谓一致 · Subject-verb agreement`.
  - `rule`: one sentence in the learner's native language.
  - `example`: a short English example.

Example:

```json
{
  "sentences": [
    {
      "corrected": "My plugin and skill ~~is~~ **are** working.",
      "source_text": "My plugin and skill is working.",
      "language": "Chinese",
      "cards": [
        {
          "title": "主谓一致 · Subject-verb agreement",
          "rule": "由 and 连接的两个主语是复数，be 动词用 are。",
          "example": "My tests and docs are ready."
        }
      ]
    }
  ]
}
```

Re-submitting the same corrected sentence updates it and upserts its cards, so
a retry is safe. A call that fails with authentication or permission errors
must not stop the user's task.

## Show the card panel

`create_english_learning_cards` already renders the panel for the cards it
saves. Call `show_english_learning_cards` with no arguments only when the user
asks to review the library or open the panel again; never call it right after
saving the same cards, or the conversation gets two panels. The same panel can
be opened from the host sidebar and thread panel when those surfaces exist.
Keep the reply focused on the task and do not repeat the card contents.

If the host does not render the panel, show the fallback note at the top of the
reply, before the answer. Show the corrected sentence once, then list every
card in that sentence:

**English note**
“[Full corrected sentence with ~~removed~~ and **corrected** words].”
- [native pattern] · [English grammar term]：[rule] 例句：“[example]”
- [native pattern] · [English grammar term]：[rule] 例句：“[example]”

Use `规则：` and `例句：` for Chinese learners, the equivalent labels for other
native languages, or `Rule:` and `For example:` for English. When the message
has several independent source sentences, repeat the block once per sentence,
in the order the sentences appear. Do not number the blocks or add an
introduction. If the message has no important issue, omit the cards entirely
and do not call the tools.

## Review the library

When the user asks about their cards, saved corrections, or progress:

- `show_english_learning_cards` opens the card panel in the conversation and
  provides the sidebar/thread entrypoint.
- `list_english_learning_cards` lists corrected sentences with their nested
  cards; filter with `query` or `corrected_type_id`.
- `get_english_learning_card` fetches one corrected sentence and its cards by
  id. The id is the corrected sentence id, not an individual card id.
- `delete_english_learning_card` deletes the corrected sentence and its cards.
- `get_profile` returns the linked 小葵（Aoi） account, including its avatar.

The tools only ever return the cards of the authenticated user. Never invent
card ids or fabricate saved data; if a tool fails, say so plainly.
