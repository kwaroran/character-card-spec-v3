> in this document, we don't describe how it should be implemented. For that, you should check the [SPEC_V3.md](SPEC_V3.md) document. **This file SHOULD NOT be used as a only reference for implementing CCv3.**
> concepts.md is outdated. please check the [SPEC_V3.md](SPEC_V3.md) document for the latest information.

# Concepts of CCv3

Character Card V3 specification is for standarlizing and expending advanced features of character cards. It is designed to be used in various platforms and applications. It is based on the [CCv2](https://github.com/malfoyslastname/character-card-spec-v2/blob/main/spec_v2.md) specification and extends it with new features.

# New Features

This is a list of new features that are added to the CCv3 specification. 

## New Embedding Methods

Old embeding method, JSON and PNG, is still supported in the CCv3 specification.

XCHAR embeding method is added to the specification. XCHAR is a new format just for storing character cards. It is a archive format that contains assets and metadata of the character card. It is designed to be used in applications that needs to store external assets and metadata in a single file to share or store.

method that allows embed external assets and metadata in PNG files also added to the specification, but this method is not recommended to use in new applications, just for legacy support, and it is not required to implement. use XCHAR instead for new applications.

## New Fields

### assets

Assets field is added to the specification to store the assets of the character card. This contains icon, background, user icon, emotion and inlay assets of the character card.

This allows botmakers to make characters with alternative icons, embeded backgrounds, embeded emotions (also known as expressions), and something like embeded persona, without worrying compatibility issues.

However, this field's features is not required to implement, and it is up to the application to support it.

### group_only_greetings

Group only greetings field is added to the specification to store greetings that are only active in group chats. this would be helpful for applications to make greetings that are only active in group chats.

### nickname

Nickname field is added to the specification to use alternative name in `{{char}}`, instead of the name field.

### creator_notes_multilingual

Creator notes multilingual field is added to the specification to store creator notes in multiple languages, which would allow botmakers to store creator notes in multiple languages.

### source

Source field is added to the specification to store the source of the character card. this would be helpful for applications to locate where the character card is from.

## creation_date

Creation date field is added to the specification to store the creation date of the character card. this would be helpful for applications to locate when the character card is created.

## modification_date

Modification date field is added to the specification to store the modification date of the character card. this would be helpful for applications to locate when the character card is modified.

## Lorebook & Decorators

The main and biggest change in the CCv3 specification is the lorebook change. The lorebook can use `@@decorator` syntax to put advanced features, like positioning, activate after a N turn, and more, to make the lorebook as a powerful, unified field to insert prompts.

Decorators are made for core users to make advanced prompts, without needing to make a new field for, it which makes newcomers confused, by making raw input for the decorators, without needing to make a UI for every single decorator.

If the decorators is conditional decorators (like make active after a N turn), to make the lorebook entry active, the application should check the all condition to make the lorebook entry active (AND).

```
@@depth 5
@@ignore_on_max_context

Prompt text here
```

This is a example of a prompt with decorators. we can see that the prompt would be positioned at depth of 5, and it will ignore the prompt if the context is at max.

applications are allowed to implement their own decorators too, so something like `@@activate_after_emotion` could be implemented can be implemented by the application.

botmakers can make fallbacks for the decorators which has one more `@`. So if the application does not support the decorator, it will fallback to the default behavior.

```
@@activate_after_emotion
@@@instruct_depth 100
@@@depth 5

Prompt text here
```

This is a example of a prompt with a fallback decorator. if the application does not support `@@activate_after_emotion`, it will fallback to the default behavior, which is `@@instruct_depth 5`. and if the application does not support `@@instruct_depth` (like not a instruct model), it will fallback to `@@depth 5`.

However, `description` field still exists, since it is used to make simple character cards, and it is used more then inserting as prompts nowadays.

Also, lorebook export format and field's behavior is standardized in the specification, unlike the CCv2 specification which does not have a standardized lorebook export format and field's behavior.

## New Fields in Lorebook

### constant

This field is a boolean that makes lorebook entry always active regardless of the key values.

Technically, this field exists in the CCv2 specification, but it was a optional field to implement, in CCv3, it is a required field to implement.

### use_regex

This field is a boolean that makes lorebook entry use regex to match the key values so botmakers can use regex to match more complex key values.

Applications are allowed to terminate/ignore the lorebook entry if the regex is too complex, or if the regex is vulnerable to ReDoS attacks.

We recommend to use `re2` regex engine, if you are going to implement this field on server side.

## Decorators

### @@activate_only_after N

This decorator makes lorebook entry only active after a N turns of the conversation.

This can be implemented as as: if user's message count is not greater than N, then the lorebook entry would not be active.

### @@activate_only_every N

This decorator makes lorebook entry only active every N turns of the conversation.

This can be implemented as as: if user's message count is not divisible by N, then the lorebook entry would not be active.

### @@keep_activate_after_match

This decorator makes lorebook entry active after a match, and keeps active until the conversation ends.

### @@dont_activate_after_match

This decorator makes lorebook entry inactive after a match, and keeps inactive until the conversation ends.

### @@depth N

This decorator makes lorebook entry positioned at a specific depth, like unstandardized `depth_prompt` field.

### @@instruct_depth

Same as `@@depth`, but for non-chat context, and checking as token depth.

### @@reverse_depth

Same as `@@depth`, but for counting from the top.

### @@reverse_instruct_depth

Same as `@@reverse_depth`, but for non-chat context, and checking as token depth.

### @@role

This decorator gives a role to the lorebook entry like "user" or "assistant".

### @@scan_depth N

This decorator modify the scan depth for only this lorebook entry.

### @@instruct_scan_depth N

Same as `@@scan_depth`, but for instruct models.

### @@is_greeting N

This decorator makes lorebook entry only active only for specific greeting.

@@is_greeting 0 would be the default greeting, and @@is_greeting 1 would be the first alternative greeting, and @@is_greeting 2 would be the second alternative greeting, and so on.

### @@position

This decorator makes lorebook entry positioned at a specific position, like position field in the CCv2 specification, but with more positioning options.

### @@ignore_on_max_context

This decorator makes lorebook entry ignore if the context reaches max context.

### @@additional_keys N,N...

This decorator makes lorebook entry active only if the additional keys are matched too like `secondary_keys` field in the CCv2 specification.

### @@exclude_keys N,N...

unlike `@@additional_keys`, this decorator makes lorebook entry inactive if the exclude keys are matched.

### @@is_user_icon N

This decorator makes lorebook entry active only for specific user icon, which are embeded in the character card.

this would be helpful for botmaker making something known as "persona" embeded in the character card.

### @@dont_activate

This decorator makes lorebook entry not activate in any case, even if the key values are matched, and other decorators' conditions are met.

this would be helpful for fallbacks, or just making lorebook entry as a note.

### @@activate

This decorator makes lorebook entry activate in any case, even if the key values are not matched, or other decorators' conditions are not met.

this would be helpful for fallbacks.


### @@disable_ui_prompt

This decorator is used for disabling application set `system_prompt` or `post_history_instructions`

## Curly Braced Syntax

New Curly braced Syntax (CBS), or macro syntax has been added to the CCv3 specification. Some were already in the many applications, but it wasn't standardized. theses are standardized in the CCv3 specification.

### `{{random:A,B,C...}}`

Random CBS selects a random value from the list.

### `{{pick:A,B,C...}}`

Same as `{{random:A,B,C...}}`, but it application would make effort to make the same value on same conditions.

### `{{roll:N}}`

Roll CBS rolls a dice with N sides.

### `{{// A}}`

This is a comment CBS, which is used to make comments.

### `{{/// A}}`

Same as `{{// A}}`, but it can be used in the lorebook matching.

### `{{comment:A}}`

It would be displayed as a inline comment in the message, and not be sent to AI model.

### `{{reverse:A}}`

Reverse CBS reverses the string, used for spoilers, or something like that.