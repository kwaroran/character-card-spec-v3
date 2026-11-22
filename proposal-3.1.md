# Proposal: Character Card V3.1 localizations

**Status:** proposal for discussion, written against V3.0 at commit `f3a86af`. The normative text is `SPEC_V3.md`,
where each addition is marked **(3.1)**. This document explains the reasons for the additions and shows a sample;
where it describes a rule, the rule as it stands is under the `SPEC_V3.md` heading it names.

**In short.** V3.1 lets a card state its language and carry translations of every field the model reads, of its
lorebook entries and of its main pictures. It adds no field to `data`: the additions are a few standard keys
inside `extensions` and a few reader behaviours. A 3.1 card keeps the 3.0 layout, so an application that follows
V3.0 still loads a complete card in the base language.

This is the first of three parts, each meant to be offered and judged separately (§5). Every key in it is optional.

## 1. Why

Character Card V3 is the shared format for moving characters between applications. Its one multilingual feature
is `creator_notes_multilingual` (`SPEC_V3.md` §`creator_notes`, §`creator_notes_multilingual`). That leaves five
gaps:

1. **A card can't state its language.** The base fields have no language tag. The only hint is the rule that
   `creator_notes` count as `en` when the map has no `en` key, which is wrong for a card written in Korean,
   Japanese or Chinese.
2. **Only the creator's notes can be translated.** Name, nickname, description, personality, scenario, greetings,
   example messages, system prompt, post-history instructions and the lorebook (what the model actually reads) can't.
3. **Codes are bare ISO 639-1.** There's no `pt-BR`, no `zh-Hant`, and no fallback from one to the other.
4. **Translations carry no provenance.** Nothing says whether the author or a machine wrote one.
5. **Authors work around it inside the text.** Many cards put `[KR]` / `[EN]` sections in one field, so every
   language is sent to the model at once, wasting context and mixing the reply language.

## 2. How 3.1 works

### 2.1 Why `extensions` and not `data`

Where a new field can live depends on what today's applications do with a card. We read the source of RisuAI
(`origin/main` at `6db4b681`, 2026-11-21) and SillyTavern (1.19.1, `82b64a45`):

- **RisuAI** rebuilds the card from its own model on export and labels it `"3.0"`. Fields in `data` that it doesn't
  know are dropped. Every `extensions` key is kept, at card level and on each lorebook entry, except
  `extensions.risuai` and `extensions.depth_prompt`, which it writes from its own settings.
- **SillyTavern** keeps every field it doesn't know, in `data` and in `extensions`, through editing and export. On
  save it relabels the card `spec: "chara_card_v2"`, `spec_version: "2.0"` (and `chara_card_v3`, `"3.0"` in a PNG's
  `ccv3` chunk), and rewrites `extensions.depth_prompt` and `extensions.talkativeness` from its own editor.
- Neither uses `spec_version` to decide how to read a V3 card.
- One place is not safe: the lorebook's own `extensions`. SillyTavern rebuilds an embedded lorebook from its World
  Info file once the two are linked, keeping each entry's `extensions` and dropping the book's. V3.1 puts nothing
  there.

So a new field in `data` is lost after one pass through RisuAI, while one in the card's or an entry's `extensions`
survives both. Two more reasons point the same way. A change to RisuAI would reach only new builds, and the builds
already in use would go on dropping the field. And other applications haven't been checked; `extensions` is the
place V2 already asks every application to keep. V3 reserves `extensions` for application-specific data, so these
keys are an exception; the precedent is the depth prompt, which two applications already write under one name.

If you'd rather have proper `data` fields, the localizations move there unchanged (`data.localizations`); the asset
details would need RisuAI to keep extra keys on an asset, since its import rebuilds each asset as `type`, `uri`,
`name` and `ext`. I'll gladly rework the pull request that way.

### 2.2 The version label

A card that uses 3.1 is labelled `"3.1"` (`SPEC_V3.md` §`spec_version`), so that a 3.0 reader, by V3's own rule,
can tell the user that the card may use features it doesn't support. That is accurate: such a reader shows only
the base language, and shows `{{lang}}` as text. The `::` separator doesn't set the label, since every application
already reads it.

Readers find 3.1 content by its keys and not by the label, because the label doesn't survive: both applications
above relabel a card when they save it. A card relabelled `chara_card_v2` keeps the keys too, so they are read
there as well, while the card itself is still read as V2.

V3.0 already asks readers not to reject a card for its version. `ccardlib`'s schema source accepts any version
string, but its compiled validator and its TypeScript type still pin `"3.0"`, so the validator calls a 3.1 card
unknown. A companion pull request to `ccardlib` regenerates the compiled validator from the schema source and adds
the types of this proposal. RisuAI doesn't run that validator when it imports a card.

### 2.3 A suggested versioning policy

V3 doesn't say what a minor version may change. `SPEC_V3.md` keeps to one sentence, that a minor version only
adds, and marks `creator_notes_multilingual` deprecated in its own section. If it helps, here is a fuller policy
that the later parts would follow; it is a suggestion for the maintainer, not part of the specification text.

- **A minor version only adds:** standard `extensions` keys, asset types, macros, reader behaviours, and stages
  for older forms. It never removes or renames a field, makes one required, or changes the layout of `data`.
- **Two additions can't be ignored by an older reader:** a new macro, which it shows as written, and a behaviour
  that changes what an existing field or macro gives (`{{char}}` in a translated chat, for example). A card opts in
  to these by using them.
- **Anything else an older reader can't safely ignore needs a major version** and a new `spec` value. Applications
  accept a card by its `spec` value, so that is the only signal that reaches them.
- **Older forms.** When a new key takes over from a form already in use, the form goes through two stages.
  *Deprecated*: the new key is the one to use; the older form can still be read, and still be written beside the
  new key for applications that know only the older form, and the section on the form says when. *Legacy*: still
  accepted when a card is read, no longer written. A form moves from deprecated to legacy only in a later version,
  so none is legacy in 3.1. A form that belongs to one application and that the application goes on writing is
  not an older form and has no stage. In this part the one deprecated form is `creator_notes_multilingual` (§3.6).

### 2.4 Keeping what you don't use

A translation is lost the first time a card passes through an application that drops it, even if that application
shows only one language. So `SPEC_V3.md` asks every application to keep the standard keys when it exports a card,
including those it doesn't use (`SPEC_V3.md` §Standard `extensions` keys). V3 already lets an application keep data it
doesn't know in `extensions`; this turns that into a duty for these keys.

## 3. Localizations

### 3.1 A language for the base fields

`extensions.localizations.language` names the language of the base fields, the fields of `data` itself. Without
it, the only hint is 3.0's rule that `creator_notes` count as `en` when `creator_notes_multilingual` has no `en`
key. With it, the card's own language takes the place of `en` in that rule (`SPEC_V3.md` §`creator_notes`), so the
rule gives the right answer for a Korean card too. A card that doesn't state its language is read as in 3.0.

### 3.2 One structure for every field

```ts
extensions.localizations?: {
  language?: string
  locales?: Record<string /* BCP 47 tag */, Localization>
}

interface Localization {
  name?: string
  nickname?: string
  description?: string
  personality?: string
  scenario?: string
  first_mes?: string
  alternate_greetings?: string[]
  group_only_greetings?: string[]
  mes_example?: string
  system_prompt?: string
  post_history_instructions?: string
  creator_notes?: string
  meta?: {
    origin: 'author' | 'machine' | 'community'
    translator?: string     // a person's name or a model's
    based_on?: number       // see §3.10
  }
}
```

- A locale uses exactly the `data` field names, rather than a `*_multilingual` sibling for each field, so one rule
  covers every field and a new translatable field needs no new key.
- Tags are BCP 47 (RFC 5646), as on the web, in HTTP `Accept-Language` and on mobile platforms, so `pt-BR` and
  `zh-Hant` can be told apart from `pt` and `zh`.
- An empty string counts as "not translated", because editors commonly write `""` for a field left blank, and a
  blank should not hide the base text.
- `meta` answers the question a user asks of a translation: who made it, the author, a machine or someone else.

### 3.3 Choosing a locale

The lookup (`SPEC_V3.md` §Selecting a locale) is the one of RFC 4647: try the wanted tag, then drop its last subtag,
and so on, so `zh-Hant-TW` finds `zh-Hant` and then `zh`. At each step the card's own `language` counts as a
locale, which is why a more specific match on the base fields beats a broader translation. For example:

- With `language: "en"` and a locale `en-GB`, a user wanting `en-GB` gets the locale, and one wanting `en-US` gets
  the base fields.
- With `language: "zh-Hant"` and a locale `zh`, a user wanting `zh-Hant-TW` gets the base fields: Traditional
  Chinese is closer than a locale that may be Simplified.

A locale that translates neither `description` nor `first_mes` is for display only. Translators often start with
just a name and the creator's notes, and a chat built from such a locale would be the base text under a
translated name. Outside a chat, where only the name and notes are shown, it is used like any other.

### 3.4 One locale per chat

A chat's locale is fixed when the chat starts, and the whole prompt comes from that locale, or from the base fields
where it has nothing. Mixing text from two locales in one prompt is what the `[KR]` / `[EN]` workaround does today,
and it is the thing this part sets out to stop. The lorebook entries and the assets find their own translation from
the chat's locale, so an entry with only an `en` translation still serves a chat in `en-GB`.

### 3.5 Greetings align by index

Translated greetings follow the base greetings by index, so that choosing greeting *n* gives the same scene in every
language, and `@@is_greeting` decorators keep pointing at the right one. A missing translation at an index falls
back to the base greeting rather than shifting the others.

### 3.6 `creator_notes_multilingual`

The map does part of what `locales` does, with bare ISO 639-1 codes and for one field only, so it is deprecated in
favour of `locales[x].creator_notes` (`SPEC_V3.md` §`creator_notes_multilingual`). Neither SillyTavern nor RisuAI
reads or writes it today (RisuAI keeps languages as marked sections inside `creator_notes`), so the rules there are
for other 3.0 readers:

- A card can still carry the map for them, written from its locales.
- The base notes go under the card's own language in that map. A 3.0 reader shows `creator_notes` only when the
  map has no `en` key (`SPEC_V3.md` §`creator_notes`), so without it a non-English card's own notes would vanish
  as soon as an English translation was added.
- An application that adds `locales` to a card that has the map lifts the map into the locales first, so no
  translation is lost. Under the card's own language, the map's text is what 3.0 readers were shown, so it becomes
  the base notes. A card with no `language` is lifted whole, and the 3.0 rule still says which text is English.

### 3.7 Lorebook entries

```ts
entry.extensions.localizations?: Record<string, {
  keys?: string[]
  secondary_keys?: string[]
  content?: string
  name?: string
  comment?: string
}>
```

- Translated keys are added to the base keys rather than replacing them, because users often write in a mix of
  languages, and a name in the original script should still fire the entry.
- An entry has no `language` of its own: the card's applies, which also means that the entry's lookup stops at the
  card's language, as the asset lookup does. With `language: "zh-Hant"`, an entry that has only a `zh` translation
  uses its own Traditional Chinese text in a `zh-Hant-TW` chat (`SPEC_V3.md` §`extensions.localizations` of an
  entry).
- Decorators stay in the base `content` and apply in every language, so a translator can't change when an entry
  fires, and the timing logic lives in one place.
- The lorebook's own `name` and `description` are not translated in 3.1: the lorebook's `extensions` don't survive
  every application (§2.1).

### 3.8 Assets

```ts
extensions.asset_info?: Array<{
  type: string        // the asset's type
  name: string        // the asset's name
  lang?: string       // a BCP 47 tag
  variant_of?: string // the name of the asset this one replaces in that language
}>
```

- An asset has no `extensions` of its own, so what 3.1 adds about assets is a list at card level. It matches an
  asset by `type` and `name`, not by `uri`, because applications rewrite `uri` when they repack a card.
- A language variant has a name of its own (`main-en`) and points to the asset it replaces. A 3.0 reader then sees
  it as one more asset of that type, and its rules for `main` assets (`SPEC_V3.md` §`assets`) still pick the
  original. A variant is never chosen or counted in its own right; a card that also targets 3.0 names the original
  `main`.
- Variants are limited to `icon` and `background` in 3.1, because a 3.0 reader such as RisuAI adds every `emotion`
  asset to the character's emotions, so a variant such as `happy-en` would become one more emotion. `lang` alone,
  which only states the language of an asset's content, is fine on any type.
- The sample in §4 uses it for a background picture of a shop sign written in Korean.

### 3.9 Macros

- `{{char}}` follows the chat's locale, so a translated name is used in a translated chat. In a chat on the base
  fields it gives what it gives in 3.0.
- `{{lang}}` gives the chat's locale, so one system prompt can say "Reply in {{lang}}." for every language. An older
  reader shows it as written, which is why it sets the version label (§2.2).

**Separators.** V3.0 writes its macros with one colon after the name (`{{random:A,B}}`), but its own `pick`
example uses two (`{{pick::Hello,Hi,Hey}}`), and applications accept both: RisuAI and SillyTavern's current macro
parser take `::` or `:` after any macro's name. Of V3.0's macros, SillyTavern's earlier parser takes `::` only for
`random` and `pick`, so there `{{reverse::Hello}}` reverses `:Hello`. `SPEC_V3.md` (§`::` as a separator) writes
down what the current parsers do, and asks cards to keep to one colon and commas, which every parser reads.

One difference is left as it is: RisuAI splits the text after a single colon at every further colon, so
`{{reverse:12:30}}` reverses `12:30` by V3.0's definition and gives `21` in RisuAI.

### 3.10 `based_on`

`meta.based_on` records the card's `modification_date` when the locale was last brought up to date. It is a date
for people, and not a test for staleness: V3 asks applications to update `modification_date` on every export, and
some applications edit a card without updating it, so a lower `based_on` says nothing reliable. An application
that knows its own edit history can still use it as a hint. A reliable test needs a fingerprint of the base text,
which a later version can add.

### 3.11 Importing older cards

An importer can split bracketed in-text sections (`[KR]`, `## English`) into locales when it can tell the
languages apart reliably, which turns today's workaround into a translated card.

### 3.12 Out of scope

`tags` and `source` are not localized: tags are a search vocabulary, and translating them splits it. Localized tag
labels belong to a tag taxonomy, not to the card.

## 4. Sample: the same card in 3.0 and 3.1

A Korean author's card with an English translation, and Japanese creator's notes only.

### 4.1 Today, V3.0

What authors do now: both languages inside each field, bracket markers, and the creator's notes in two places.

```json
{
  "spec": "chara_card_v3",
  "spec_version": "3.0",
  "data": {
    "name": "서연 (Seo-yeon)",
    "nickname": "",
    "description": "[KR]\n서연은 오래된 헌책방의 주인이다. 조용하지만 책 이야기만 나오면 말이 많아진다.\n\n[EN]\nSeo-yeon runs an old second-hand bookshop. She is quiet until someone mentions a book.",
    "personality": "[KR] 차분함, 호기심, 고집 [EN] calm, curious, stubborn",
    "scenario": "[KR]\n비 오는 저녁, {{user}}가 서점에 들어온다.\n\n[EN]\nOn a rainy evening, {{user}} walks into the shop.",
    "first_mes": "[KR]\n\"어서 오세요. 우산은 거기 두시면 돼요.\"\n\n[EN]\n\"Welcome. You can leave your umbrella there.\"",
    "alternate_greetings": [
      "[KR]\n\"문 닫을 시간인데… 책 한 권만 보고 가실래요?\"\n\n[EN]\n\"We're about to close… one book before you go?\""
    ],
    "group_only_greetings": [],
    "mes_example": "<START>\n{{char}}: [KR] \"이 책, 초판이에요.\" [EN] \"This one's a first edition.\"",
    "system_prompt": "",
    "post_history_instructions": "",
    "creator_notes": "Bilingual card. Pick your language in the first message.",
    "creator_notes_multilingual": {
      "ko": "한/영 카드입니다. 첫 메시지에서 언어를 고르세요.",
      "ja": "韓国語・英語のカードです。"
    },
    "tags": ["slice of life", "bookshop"],
    "creator": "example-author",
    "character_version": "1.2",
    "source": [],
    "creation_date": 1730000000,
    "modification_date": 1731000000,
    "extensions": {},
    "assets": [
      { "type": "icon", "uri": "ccdefault:", "name": "main", "ext": "png" },
      { "type": "background", "uri": "embeded://assets/shop_sign_ko.png", "name": "main", "ext": "png" }
    ],
    "character_book": {
      "entries": [
        {
          "keys": ["초판", "first edition"],
          "content": "[KR] 서연은 초판본을 팔지 않는다. [EN] Seo-yeon never sells first editions.",
          "enabled": true,
          "insertion_order": 100,
          "use_regex": false,
          "extensions": {}
        }
      ]
    }
  }
}
```

Problems visible here: the model gets both languages in every field; `creator_notes` is English but the card is
Korean, so the `en` rule happens to hold only because the author wrote the notes in English; the Korean shop-sign
background is shown to everyone; and nothing marks the Japanese notes as a machine translation.

### 4.2 Proposed, V3.1

```json
{
  "spec": "chara_card_v3",
  "spec_version": "3.1",
  "data": {
    "name": "서연",
    "nickname": "",
    "description": "서연은 오래된 헌책방의 주인이다. 조용하지만 책 이야기만 나오면 말이 많아진다.",
    "personality": "차분함, 호기심, 고집",
    "scenario": "비 오는 저녁, {{user}}가 서점에 들어온다.",
    "first_mes": "\"어서 오세요. 우산은 거기 두시면 돼요.\"",
    "alternate_greetings": [
      "\"문 닫을 시간인데… 책 한 권만 보고 가실래요?\""
    ],
    "group_only_greetings": [],
    "mes_example": "<START>\n{{char}}: \"이 책, 초판이에요.\"",
    "system_prompt": "",
    "post_history_instructions": "",
    "creator_notes": "한/영 카드입니다. 언어는 앱에서 고르세요.",
    "creator_notes_multilingual": {
      "ko": "한/영 카드입니다. 언어는 앱에서 고르세요.",
      "en": "Korean/English card. Choose the language in your app.",
      "ja": "韓国語・英語のカードです。"
    },
    "tags": [
      "slice of life",
      "bookshop"
    ],
    "creator": "example-author",
    "character_version": "1.2",
    "source": [],
    "creation_date": 1730000000,
    "modification_date": 1731000000,
    "extensions": {
      "localizations": {
        "language": "ko",
        "locales": {
          "en": {
            "name": "Seo-yeon",
            "description": "Seo-yeon runs an old second-hand bookshop. She is quiet until someone mentions a book.",
            "personality": "calm, curious, stubborn",
            "scenario": "On a rainy evening, {{user}} walks into the shop.",
            "first_mes": "\"Welcome. You can leave your umbrella there.\"",
            "alternate_greetings": [
              "\"We're about to close… one book before you go?\""
            ],
            "mes_example": "<START>\n{{char}}: \"This one's a first edition.\"",
            "creator_notes": "Korean/English card. Choose the language in your app.",
            "meta": {
              "origin": "author",
              "based_on": 1731000000
            }
          },
          "ja": {
            "name": "ソヨン",
            "creator_notes": "韓国語・英語のカードです。",
            "meta": {
              "origin": "machine",
              "translator": "example-mt",
              "based_on": 1730000000
            }
          }
        }
      },
      "asset_info": [
        {
          "type": "background",
          "name": "main-en",
          "lang": "en",
          "variant_of": "main"
        }
      ]
    },
    "assets": [
      {
        "type": "icon",
        "uri": "ccdefault:",
        "name": "main",
        "ext": "png"
      },
      {
        "type": "background",
        "uri": "embeded://assets/shop_sign_ko.png",
        "name": "main",
        "ext": "png"
      },
      {
        "type": "background",
        "uri": "embeded://assets/shop_sign_en.png",
        "name": "main-en",
        "ext": "png"
      }
    ],
    "character_book": {
      "entries": [
        {
          "keys": [
            "초판"
          ],
          "content": "서연은 초판본을 팔지 않는다.",
          "enabled": true,
          "insertion_order": 100,
          "use_regex": false,
          "extensions": {
            "localizations": {
              "en": {
                "keys": [
                  "first edition"
                ],
                "content": "Seo-yeon never sells first editions."
              }
            }
          }
        }
      ]
    }
  }
}
```

Everything new is inside `extensions`: the card's `data` holds only 3.0 fields, in Korean. The English sign is a
language variant of `main` under its own name (§3.8), so the card still has one `main` background. The `ko` key in
`creator_notes_multilingual` repeats the base notes for 3.0 readers (§3.6).

### 4.3 What each reader gets

| Reader | Wanted | Prompt text | Lorebook fires on | Background |
|---|---|---|---|---|
| 3.0 app | any | Korean base fields only | `초판` | `main` (Korean sign) |
| 3.1 app | `ko` | Korean base fields | `초판` | `shop_sign_ko` (`main`) |
| 3.1 app | `en-GB` | English fields (lookup `en-GB` → `en`) | `초판` or `first edition`; English content inserted | `shop_sign_en` (the `en` variant of `main`) |
| 3.1 app | `ja` | Korean base fields: the `ja` locale has only a name and notes, so it is for display. The card's page shows the name `ソヨン` and the Japanese notes, marked machine-translated | `초판` | `shop_sign_ko` (`main` has no `ja` variant) |

### 4.4 Field-by-field difference

| 3.0 sample | 3.1 sample |
|---|---|
| No language stated | `extensions.localizations.language: "ko"` |
| `[KR]…[EN]…` inside description, personality, scenario, greetings, examples | Base fields Korean only; English under `locales.en` |
| Name `서연 (Seo-yeon)` | Base `서연`; `locales.en.name` `Seo-yeon`, `ja.name` `ソヨン`; `{{char}}` follows the chat locale |
| `creator_notes` in English, `creator_notes_multilingual` `ko`, `ja` | `creator_notes` in Korean (the card's language); `locales.*.creator_notes`; `creator_notes_multilingual` written for 3.0 readers, with the base notes under `ko` |
| No provenance | `meta.origin`, `translator`, `based_on` per locale |
| Lorebook keys of both languages in one list, content mixed | Base keys + the entry's `extensions.localizations.en.keys`; content per locale |
| One background for everyone | `main`, and an English variant of it (`main-en`) listed in `extensions.asset_info` |

## 5. Later parts

Two more parts follow as separate pull requests, each built the same way (standard keys in `extensions`, nothing
new in `data`) and each optional:

- **Part B, standard fields:** narration, the user's point of view, the depth prompt, a rating, a licence,
  talkativeness, emotion names, and pictures inside text.
- **Part C, documented application forms:** SillyTavern's and RisuAI's text-replacement scripts and variables,
  described as they are so that one application can read what another wrote.

## 6. Checklist for implementers

A list of what an application does to support this part. It adds no rule: each line is a reminder that points to
the heading in `SPEC_V3.md` that states the rule, and how strong each rule is is said there. Every key is
optional, so after §6.1 an application works through only the lines for what it supports.

### 6.1 Every application, whatever it supports

| | Do | In `SPEC_V3.md` |
|---|---|---|
| ☐ | Keep every standard key when a card is exported, including those you don't use | Standard `extensions` keys |
| ☐ | Find 3.1 content by its keys, on a card with any `spec_version`, never by the label; read the keys on a card relabelled `chara_card_v2` too | `spec_version` |
| ☐ | As in V3.0, don't reject a card for its `spec_version`; a validator accepts any `3.x` | `spec_version` |

### 6.2 Localizations

**Reading a card**

| | Do | In `SPEC_V3.md` |
|---|---|---|
| ☐ | Read `extensions.localizations`: the card's `language` and its `locales`; compare language tags without regard to case | `extensions.localizations` |
| ☐ | Choose a chat's locale by the lookup, passing over display-only locales; run it without that exception outside a chat | Selecting a locale |
| ☐ | Fix the chat's locale when the chat starts; never put text chosen for two chat locales into one prompt | Selecting a locale |
| ☐ | Use the base field for each value the locale lacks; an empty string is not a translation | `extensions.localizations` |
| ☐ | Match translated greetings to the base greetings by index | `extensions.localizations` |
| ☐ | Replace `{{char}}` (and `<char>`, `<bot>`) with the chat locale's `nickname`, else its `name`, else the base values | `{{char}}` |
| ☐ | On a card that has `locales`, take creator notes from there and from `creator_notes`, and don't read `creator_notes_multilingual` | `creator_notes_multilingual` |
| ☐ | On a card with a `language` and no `locales`, read `creator_notes_multilingual` as in V3.0 with the card's language in place of `en` | `creator_notes` |
| ☐ | For each lorebook entry, find its translation from the chat's locale, stopping at the card's `language`; match on the base keys plus that translation's; don't act on a decorator in translated content, and leave it out of the prompt | `extensions.localizations` of an entry |
| ☐ | Read `extensions.asset_info`; after choosing an `icon` or `background` as V3.0 says, look for its language variant; never choose or count a variant in its own right | `extensions.asset_info` |
| ☐ | Replace `{{lang}}` with the chat's locale | `{{lang}}` |
| ☐ | Accept `::` after a macro's name wherever V3.0 writes `:` | `::` as a separator |
| ☐ | Don't treat a locale as out of date because `based_on` is lower than `modification_date` | `extensions.localizations` |

**Writing a card**

| | Do | In `SPEC_V3.md` |
|---|---|---|
| ☐ | Write `spec_version` `"3.1"` when the card uses a 3.1 key or `{{lang}}`; a card with nothing of 3.1 stays `"3.0"` | `spec_version` |
| ☐ | Write locale keys as well-formed BCP 47 tags in their usual form; write no locale for the card's own `language` | `extensions.localizations` |
| ☐ | Give each language variant of an `icon` or `background` a name of its own, with `lang` and `variant_of` in `asset_info` | `extensions.asset_info` |
| ☐ | Keep decorators in an entry's base `content` only | `extensions.localizations` of an entry |
| ☐ | Use one colon and commas in V3.0's macros, which every application reads | `::` as a separator |
| ☐ | Optional: write `creator_notes_multilingual` for 3.0 readers, with the base notes under the base language's key | `creator_notes_multilingual` |
| ☐ | When adding `locales` to a card that has `creator_notes_multilingual`, lift the map into `locales` first | `creator_notes_multilingual` |
