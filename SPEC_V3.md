# Character Card V3 Specification

This document describes the Character Card V3 specification (shorted as CCv3).

# Keywords

- *MUST*: This word, or the terms "REQUIRED" or "SHALL", mean that the definition is an absolute requirement of the specification.
- *MUST NOT*: This phrase, or the phrase "SHALL NOT", mean that the definition is an absolute prohibition of the specification.
- *SHOULD*: This word, or the adjective "RECOMMENDED", mean that there may exist valid reasons in particular circumstances to ignore a particular item, but the full implications must be understood and carefully weighed before choosing a different course.
- *SHOULD NOT*: This phrase, or the phrase "NOT RECOMMENDED" mean that there may exist valid reasons in particular circumstances when the particular behavior is acceptable or even useful, but the full implications should be understood and the case carefully weighed before implementing any behavior described with this label.
- *MAY*: This word, or the adjective "OPTIONAL", mean that an item is truly optional.
- *MAY NOT*: This phrase, or the adjective "forbidden", mean that an item is truly optional.
- *IN ANY CASE*: This phrase mean that the application must do the action described in the sentence in any case, even it violates the other rules in the specification.
- Application: The application that uses the Character Card V3 specification.
- Character Editor: The editor that is used to create and edit the Character Card V3 objects.
- Creator Note: The note that are *SHOULD* be very discoverable for bot user.
- regex pattern: A Regular Expression pattern that is used to match the text, for example, `/hello/i`.


# Embedding Methods

## PNG/APNG

CharacterCardV3 objects could be embedded in PNG or APNG files. The CharacterCardV3 object *MUST* be embedded in the PNG or APNG file as a tEXt chunk. The tEXt chunk *MUST* be named `ccv3` and the value of the tEXt chunk *MUST* be the JSON string of the CharacterCardV3 object, with utf-8 -> base64 encoding.

Application *MAY* backfill Character Card V2 objects from the PNG/APNG files in `chara` chunk. however, Application backfilling Character Card V2 objects from the PNG/APNG files *SHOULD* add a warning to the user that it is backfilled on `creator_notes` field in the TavernCardV2 object. like: `This character card is Character Card V3, but it is loaded as a Character Card V2. Please use a Character Card V3 compatible application to use this character card properly.` unless its exported only for Character Card V2, not for Character Card V3. The backfilled `chara` chunk *SHOULD* be trimmed on import.

if the application detects both `chara` and `ccv3` chunk, the application *SHOULD* use the `ccv3` chunk.

PNG/APNG files *MAY* have additional tEXt chunks that works like assets embedded in CHARX files. the tEXt chunk *MUST* be named `chara-ext-asset_:{path}` and the value of the tEXt chunk *MUST* be the base64 encoded binary data. and it could be accessed by `__asset:{path}` URI. however, implementing this on new applications *SHOULD* be avoided, and CHARX files *SHOULD* be used instead.

## JSON

CharacterCardV3 objects could be embedded in JSON files. The JSON file *MUST* be a CharacterCardV3 object.

## CHARX

CharacterCardV3 objects could be embedded in CHARX files. The CHARX file is a file format that is used to store the CharacterCardV3 object. The CHARX file is a zip file that contains the CharacterCardV3 object as a JSON file, and with other assets embedded. The CHARX file *MUST* have a JSON file that contains the CharacterCardV3 object. The JSON file *MUST* be named `card.json` at the root of the zip file. . the assets embedded in the CHARX file can be accessed by `embeded://path/to/asset.png` URI (Not `embedded://`). the path is case sensitive, and sepearated by `/`. CHARX file uses `.charx` file extension.

CHARX file *SHOULD* follow this rule saving the assets:
- if the asset is a image like `.png` or `.avif`, the asset *SHOULD* be saved at 'assets/{type}/images/' directory.
- if the asset is a audio like `.mp3`, the asset *SHOULD* be saved at 'assets/{type}/audio/' directory.
- if the asset is a video like `.mp4` or `.webm`, the asset *SHOULD* be saved at 'assets/{type}/video/' directory.
- if the asset is a Live2d model, the assets *SHOULD* be saved at 'assets/{type}/l2d/' directory
- if the asset is a 3d model like `.mmd` or `.obj`, the asset *SHOULD* be saved at 'assets/{type}/3d/' directory.
- if the asset is a AI model like `.safetensors` or `.ckpt` or `.onnx`, the asset *SHOULD* be saved at 'assets/{type}/ai/' directory.
- if the asset is a font like `.otf` or `.ttf`, the asset *SHOULD* be saved at 'assets/{type}/fonts/' directory.
- if the asset is a programing code like `.lua` or `.js`, the asset *SHOULD* be saved at 'assets/{type}/code/' directory.
- if the asset is a other type of file, the asset *SHOULD* be saved at 'assets/{type}/other/' directory.

This *MUST NOT* taken to mean that the application must support these features and the directories would exist always.

How {type} is determined *SHOULD* be where the asset is used. If the asset is used as a icon, the {type} *SHOULD* be `icon`. if the asset is used as a background, the {type} *SHOULD* be `background`. if the asset is used as a emotion, the {type} *SHOULD* be `emotion`. if the asset is used as a user icon, the {type} *SHOULD* be `user_icon`. if the asset is used as a other type of asset, the {type} *SHOULD* be `other`. platform specific assets *MAY* use platform specific {type}.

Application specific data can be stored as a JSON file in root of the zip file. frontends *MAY* use this file to store the data that is not related to the CharacterCardV3 object.

Zip file *SHOULD NOT* be encrypted and *SHOULD* only use characters inside ASCII range for the file name and the file path inside the zip file to prevent compatibility issues.

Application's *MAY* reject the CHARX file if the file is too large, or the file is corrupted, or the file is not a valid zip file, or the file is encrypted and the application does not support decryption, or the file is encrypted and the password is incorrect or something else that the application does not support.

# JSON Objects in Character Card V3

## CharacterCard Object

The CharacterCard object is a JSON object that contains the information about the character card. CharacterCard object in Character Card V3 is a superset of TavernCardV2 object in Character Card V2, except the fields that are changed or removed.

For the TavernCardV2 object in Character Card V2, see [Character Card V2 Specification](https://github.com/malfoyslastname/character-card-spec-v2/blob/main/spec_v2.md).

This document describes the CharacterCard object fields which are not present in TavernCardV2 in Character Card V2, or the fields that are changed in Character Card V3.

The CharacterCardV3 object can be described as this typescript interface:

```ts

interface CharacterCardV3{
  spec: 'chara_card_v3'
  spec_version: '3.0'
  data: {
    // fields from CCV2
    name: string
    description: string
    tags: Array<string>
    creator: string
    character_version: string
    mes_example: string
    extensions: Record<string, any>
    system_prompt: string
    post_history_instructions: string
    first_mes: string
    alternate_greetings: Array<string>
    personality: string
    scenario: string

    //Changes from CCV2
    creator_notes: string
    character_book?: Lorebook

    //New fields in CCV3
    assets?: Array<{
      type: string
      uri: string
      name: string
      ext: string
    }>
    nickname?: string
    creator_notes_multilingual?: Record<string, string>
    source?: string[]
    group_only_greetings: Array<string>
    creation_date?: number
    modification_date?: number
  }
}
```

For future versions of the specification, the application *SHOULD* ignore the fields that are not present in the specification, but not reject the import of CharacterCard object. The application *MAY* save the fields that are not present in the specification so it can be exported safely.

This *MUST* not taken to mean that the application can add their own fields to the CharacterCard object. The application *MUST* follow the specification. for application specific data, the application *MAY* save the data in the `extensions` field, which is specified in V2 specification.

### `spec`

The value for this field *MUST* be `"chara_card_v3"`. applications *SHOULD NOT* consider the card is following the Character Card V3 specification if the value of this field is not equal to `"chara_card_v3"`.

### `spec_version`

The value for this field *MUST* be `"3.0"`. This field is used to determine the version of the Character Card object. applications *SHOULD NOT* reject the Character Card object if the value of this field is not equal to `"3.0"` for backward compatibility and future versions of the specification.

How the `spec_version` is determined as a newer version is by parsing the string as a float. if the float is bigger than `3.0`, the application *SHOULD* consider the Character Card object as a newer version of the specification. if the float is smaller than `3.0`, the application *SHOULD* consider the Character Card object as an older version of the specification.

If the application does not support the newer version of the specification, the application *SHOULD* alert the user that the Character Card object is created with a newer version of the specification, and the application *MAY* not support the new features that are added in the newer version of the specification. however, the application *SHOULD* support importing of it.

If the application is older than the version of the specification, the application *SHOULD* fill the missing fields with default values. default fields are:

- Since version `"3.0"` is the first version of the specification, this wouldn't happen on this version of the specification. no default fields are defined.

### `character_book`

The value of this field *MUST* be a Lorebook object or undefined. if this field is present, the application *SHOULD* consider this field as a character specific lorebook. applications MUST use the character lorebook by default and Character editors MUST save character lorebooks in the specified format. Character lorebook *SHOULD* stacked with can be defined as a global lorebook.

### `creator_notes`

The value of this field *MUST* be a string. this value *MUST* considered as creator notes if `creator_notes_multilingual` is undefined. if `creator_notes_multilingual` is present, the application *SHOULD* considered this as a creator note for `en` language, if `creator_notes_multilingual` does not have a key for `en` language. if is not, the application *SHOULD* ignore this field.

### `nickname`

The value of this field *MUST* be a string or undefined. if the value is present, the syntax `{{char}}`, `<char>` and `<bot>` *SHOULD* be replaced with the value of this field in the prompt instead of the `name` field.

### `creator_notes_multilingual`

The value of this field *MUST* be a object or undefined. if this field is present, the application *MUST* consider this field as a multilingual creator notes. the key of the object *MUST* be a language code in ISO 639-1, without region code. the value of the object *MUST* be a string. the application *SHOULD* display the creator notes in the language that the user's client is set to. the application *MAY* provide language selection for creator notes.

### `source`

the value of this field *MUST* be a array string or undefined. if the value is present, the application *SHOULD* determine as an array of the ID or a HTTP/HTTPS URL that points to the source of the character card.

The field *SHOULD NOT* be editable by the user. If the `source` is a URI, the application *MAY* provide a way to open the value of the `source` field in a new tab.

applications *SHOULD* only append elements to the `source` field and *SHOULD NOT* remove or modify the elements in the `source` field if the element isn't added by application. elements appended by application *MAY* be editable by the application.

However, if it significantly slows down the application, or it makes the application hard to use, or the source is harmful, the application *MAY* remove the elements in the `source` field. if the application removes the elements in the `source` field, the application *SHOULD* alert the user that the source is removed.

### `assets`

The value of this field *MUST* be an array of objects or undefined. if this field is undefined, the application *MUST* behave as if the value is this array:

```ts
[{
  type: 'icon',
  uri: 'ccdefault:',
  name: 'main',
  ext: 'png'
}]
```

the value *SHOULD* be determine as an array of character's assets. the object *MUST* have a `type` field and a `uri` field and a `name` field and a `uri` ext. the `type` field *MUST* be a string, and the `uri` field *MUST* be a string. the `type` field *MUST* be a type of the asset. the `uri` field *MUST* be a HTTP or HTTPS URL of the asset or base64 data URL or 'path for embedded assets' or `ccdefault:`. how the path for embedded assets formated *SHOULD* follow this format: `embeded://path/to/asset.png`. this format is case sensitive, and sepearated by `/`. if the URI field is `ccdefault:`, the application *SHOULD* use the default asset for the type. how the default asset is determined is up to the application except it is specified in the specification. if the URI field is HTTP or HTTPS URL or base64 data URL, the application *SHOULD* use the asset from the URL. applications *MAY* ignore elements with HTTP url in the `uri` which isn't HTTPS for security reasons. applications *MAY* check if the asset URI is valid and the asset is accessible. applications *MAY* ignore base64 data URLs if the application does not support the format of the asset, or the asset is too large. applications *MAY* add more URI type support like `file://`, `ftp://` etc. but the URI type *SHOULD* be a valid URI type. and if the URI type is not supported by the application, the application *MAY* ignore the asset. however, the application *SHOULD* support `embeded://` and `ccdefault:` URI types which are defined in the specification.

Where the each assets would be used is determined by the `type` field. 
- If the `type` field is `icon`, the asset *SHOULD* be used as an icon or protrait of the character. if one of the assets is `icon` type, the application *SHOULD* use the asset as the icon of the character card. if there is multiple `icon` type assets, the application *SHOULD* use the main icon or let the user choose the icon, or change dynamically by the application. how the main icon choosed is below on the specification. if `uri` field is `ccdefault:`, and the card is PNG/APNG embedded, `ccdefault:` *SHOULD* point the PNG/APNG file itself or modified version of the PNG/APNG file.
- If the `type` field is `background`, the asset *SHOULD* be used as a background of the character card. if one of the assets is `background` type, the application *SHOULD* use the asset as the background of the character card. if there is multiple `background` type assets, the application *SHOULD* use the main background as the background of the character card or let the user choose the background, or change dynamically by the application. how the main background choosed is below on the specification.
- If the `type` field is `user_icon`, the asset *SHOULD* be used as a user's icon. if one of the assets is `user_icon` type, the application *SHOULD* use the asset as the user's icon. if there is multiple `user_icon` type assets, the application *MAY* let the user choose the icon. if `uri` field is `ccdefault:`, and the card is PNG/APNG embedded, `ccdefault:` *SHOULD NOT* point the PNG/APNG file itself or modified version of the PNG/APNG file. if the application supports feature known as "persona", the application *SHOULD* disable persona feature if one or more of the assets is `user_icon` type. the usage as a icon *MAY* be toggleable by the user, or application's persona settings.
- If the `type` field is `emotion`, the asset *SHOULD* be used as an emotion or expression of the character. how this asset would be handled is up to the application.

This *MUST NOT* taken to mean that the application must support these features. The application *MAY* ignore the assets if the application does not support the feature that the asset is used for, determined by the `type` field.

`name` field *MUST* be a string. this field *MAY* be used to identify the asset. this field *SHOULD NOT* be used on prompt engining. if `type` is `emotion`, `name` *SHOULD* be used to identify the emotion like `happy`, `sad`, `angry` etc, and use element that `type` is `neutral` for default if it exists, if the application supports emotions. if `type` is `user_icon`, `name` *SHOULD* be used as a name of the user. if `type` is `icon`, and `name` is `main`, the application *MUST* use the asset as the main icon of the character card if icon is supported on the application. if there is more then one elements with `type` field is `icon`, there *MUST* be one element with `name` field is `main`, this *MUST NOT* be less or more than one. if `type` is `background`, and `name` is `main`, the application *MUST* use the asset as the main background of the character card if background is supported on the application. if there is more then one elements with `type` field is `background`, there *MUST* not be more than one element with `name` field is `main` with `background` type. if there is no element with `name` field is `main` with `background` type, the application *SHOULD* use the first element with `background` type as the main background.

for other types, it is up to the application how to use and write the name field. for example, `name` field *MAY* be used as a name of the background like `forest`, `city`, `space` etc.

`ext` field *MUST* be a string. this field *MUST* be a file extension of the asset for checking the format of the asset. this field *MUST* be in lowercase and this field *MUST* be a valid file extension without `.` like `png`. if the application does not support the file extension, the application *MAY* ignore the asset. `ext` may also be `unknown` for unknown file extensions.
Applications can decide what format to support. however, applications *SHOULD* support at least png, jpeg and webp format. if `uri` field is `ccdefault:`, this field *SHOULD* be ignored.

If the applicaiton determines that the asset is not valid or accessible or does not support the feature that the asset is used for, the application *MAY* ignore the asset, but the application *SHOULD* keep the asset data so it can be exported safely. if it is impossible or hard to save the asset data, the application *MAY* not save the asset data and do not export the asset data when exporting the CharacterCard object, but it *MUST* alert the user that the asset is not saved.

applications *MAY* add more types of assets, but added types *SHOULD* start with `x_` to prevent conflicts with the types defined in the specification.

### `group_only_greetings`

The value of this field *MUST* be an array of string. this field *MUST* be present. this field *MAY* be empty array. this field *MUST* be used to define the greetings that the character card would use.

This value *SHOULD* be considered as the additional greetings that the character card would use, only for group chats.

### `creation_date`

The value of this field *MUST* be a number or undefined. this field *MAY* be used to determine the creation date of the character card. the value *MUST* be a unix timestamp in seconds. application *SHOULD* add this field when the character card is created. application *SHOULD NOT* allow the user to edit this field. application *SHOULD NOT* modify this field if the value is already present. the time *MUST* be Unix timestamp in seconds, in UTC timezone. application *MAY* put `0` instead to this field to determine that the creation date is unknown for privacy reasons and more.

### `modification_date`

The value of this field *MUST* be a number or undefined. this field *MAY* be used to determine the modification date of the character card. the value *MUST* be a unix timestamp in seconds. application *SHOULD* add or modify this field when the character card is exported, and *MAY* modify this field when the character card is modified. application *SHOULD NOT* allow the user to edit this field. the time *MUST* be Unix timestamp in seconds, in UTC timezone. application *MAY* put `0` instead to this field to determine that the modification date is unknown for privacy reasons and more.

## Lorebook Fields

These fields are inside the Lorebook object.

Lorebook object is a JSON object that contains the information about the lorebook. Lorebook object is used to define the lorebook entries and the lorebook itself.

The Lorebook object can be described as this typescript interface:
```ts
type Lorebook = {
  name?: string
  description?: string
  scan_depth?: number 
  token_budget?: number
  recursive_scanning?: boolean
  extensions: Record<string, any>
  entries: Array<{
    keys: Array<string>
    content: string
    extensions: Record<string, any>
    enabled: boolean
    insertion_order: number
    case_sensitive?: boolean

    //V3 Additions
    use_regex: boolean

    //On V2 it was optional, but on V3 it is required to implement
    constant?: boolean

    // Optional Fields
    name?: string
    priority?: number
    id?: number|string
    comment?: string

    selective?: boolean
    secondary_keys?: Array<string>
    position?: 'before_char' | 'after_char'
  }>
}
```


If the application supports export and import of the Lorebook alone, it *SHOULD* be JSON and follow this format, represented as a typescript interface:

```ts
{
  spec: 'lorebook_v3',
  data: Lorebook
}
```

For future versions of the specification, the application *SHOULD* ignore the fields that are not present in the specification, but not reject the import of Lorebook object. The application *MAY* save the fields that are not present in the specification so it can be exported safely.

This *MUST* not taken to mean that the application can add their own fields to the Lorebook object. The application *MUST* follow the specification. for application specific data, the application *MAY* save the data in the `extensions` field, which is specified in V2 specification.

### `name`

The value of this field *MUST* be a string or undefined. this field *MAY* be used to identify the Lorebook. this field *SHOULD NOT* be used on prompt engining.

### `description`

The value of this field *MUST* be a string or undefined. this field *MAY* be used to add comments to the Lorebook. this field *SHOULD NOT* be used on prompt engineering.

### `scan_depth`

The value of this field *MUST* be a number or undefined. if this field is present, checking a match in `keys` field in the lorebook entries *SHOULD* be considered as a match only if recent `scan_depth` messages contain the keys. If the context isn't chat based, or the client's environment doesn't support chat logs, or is impossible to determine the recent messages, the application *SHOULD* ignore this field, and it would be up to the application how to handle the lorebook scanning.


### `token_budget`

The value of this field *MUST* be a number or undefined. if this field is present, the application *SHOULD* remove the lowest priority lorebook fields if the total token count of the lorebook fields exceeds the value of this field. if this field is undefined, or `priority` and `insertion_order` field is not present in the lorebook entries, or it is impossible to determine the total token count how the application removes lorebook fields is up to the application.

### `recursive_scanning`

The value of this field *MUST* be a boolean or undefined. if this field is true, the application *MAY* consider the lorebook entries as a match if other lorebook entries' `content` field is a match, regardless of `scan_depth`. if this field is false, the application *MUST NOT* consider the lorebook entries as a match if other lorebook entries' `content` field is a match. if this field is undefined, the application can decide how to handle recursive scanning.

### `entries`

The value of this field *MUST* be an array of objects. this field *MUST* be present. this field *CAN* be empty array.

## `entries` Fields

These fields are inside each entry in the `entries` array.

### `keys`

The value of this field *MUST* be a array of strings.

The lorebook field would be considered as a match if the chat log contains one of the strings in `keys` and `scan_depth`'s conditions and decorator's conditions are met. how the prompt matches specifically is up to the application, unless other fields and decorators specify otherwise.

### `secondary_keys`

This value *MUST* be a multiple values separated by a comma, as strings. if this value is present and `selective` is true,, the lorebook field *SHOULD NOT* considered as a match if the chat log do not contains one of the strings in value. how the prompt matches specifically is up to the application, unless other fields and decorators specify otherwise.

decorator that modifies `keys` field's behavior also modifies `secondary_keys` field. if `use_regex` is true, this field *SHOULD* be ignored.

### `selective`

The value of this field *MUST* be a boolean. if this value is used with `secondary_keys` field.

### `content`

The value of this field *MUST* be a string. If the field is considered as a match, the application *MUST* add this field to the prompt.

This does not mean if the lorebook field matches multiple times, the content field would be added multiple times. The application *MUST* only add the content field once.

If the content field is empty, the application *MUST* not add anything to the prompt, regardless of the lorebook field matches or not.

Content field can contain decorators. If the content field contains decorators, the application *SHOULD* follow the rules defined in the decorators, and trim the decorators from the content field before adding it to the prompt. the newlines before and after the decorators *SHOULD* be trimmed as well.

### `use_regex`

The value of this field *MUST* be a boolean. if this value is true, the lorebook field would considered as a match if the chat log matches one of the regex patterns in `keys`, instead of checking if the chat log contains one of the strings in `keys`. if the value in `keys` has invalid regex patterns the application *MUST* consider the lorebook field as not a match.

Applications *MAY* use only one regex pattern in the `keys` field which is in the first index of the array for performance reasons.

### `enabled`

The value of this field *MUST* be a boolean. if this value is false, the lorebook field *MUST NOT* be considered as a match IN ANY CASE.

### `insertion_order`

The value of this field *MUST* be a number. this determines the order of the lorebook fields in the prompt. the lower the number, it would be added to the prompt earlier. if the value of this field is the same, the order of the lorebook fields *SHOULD* determinded by the application.

if this field is present, and the `priority` field is not present, application *MAY* remove the lowest `insertion_order` lorebook fields first if the lorebook fields reaches `token_budget` limit.

if decorator that modifies position of the content field is present, the application *MAY* ignore or modify the behavior of this field.

### `case_sensitive`

The value of this field *MUST* be a boolean or undefined. if this value is true, the keys in the lorebook field *SHOULD* be case sensitive. if this field is false, the keys in the lorebook field *SHOULD* be case insensitive. if this field is undefined, the application *MAY* consider the lorebook field as case sensitive or case insensitive.

### `constant`

The value of this field *MUST* be a boolean or undefined. if this value is true, regardless what the `keys` and `secondary_keys` fields are, the lorebook field *MUST* be considered as a match if the decorator's conditions are met. if `use_regex` is true, this field *SHOULD* be ignored.

## Optional `entries` Fields

These fields are inside each entry in the `entries` array. These field are optional. application *MAY* implement these fields and ignore them. however, even if the application does not implement these fields, the application *SHOULD* save these fields so it can be exported safely.

### `name`

> AgnAI, Risu Compatibility

The value of this field *MUST* be a string or undefined. this field *MAY* be used to identify the lorebook field. this field *SHOULD NOT* be used on prompt engining.

### `id`

> ST, Risu Compatibility

The value of this field *MUST* be a string, number or undefined. this field *MAY* be used to identify the lorebook field. this field *SHOULD NOT* be used on prompt engining.

### `comment`

> ST, Risu Compatibility

The value of this field *MUST* be a string or undefined. this field *MAY* be used to add comments to the lorebook field. this field *SHOULD NOT* be used on prompt engineering.

### `priority`

> AgnAI Compatibility

The value of this field *MUST* be a number or undefined. if this field is present, the application *MAY* remove the lowest priority lorebook fields first if the lorebook fields reaches `token_budget` limit. if this field is undefined, applications *MAY* follow the `insertion_order` field instead.

## Decorators

decorators are a way to add more complex features to the lorebook entries without adding more fields. Decorators are added to the content field, and the application *SHOULD* follow the rules defined in the decorators.

Decorators are a string that starts with `@@` and ends with a newline. it can contain values that are separated by a comma.

If the decorator name is not recognized by the application, or the value is not valid, the application *SHOULD* ignore the decorator. applications *MAY* add more decorators for their own use. however, the added decorators *SHOULD* start with `@@` and should only use latin characters and underscores for the name, and named by snake_case.

If there is multiple decorators with same name, the application *SHOULD* only consider the first decorator with the name, unless its a custom decorator made by the application, or if its defined in the specification to be used multiple times.

Decorators are used like this:

```
@@decorator_with_string_value value
@@another_decorator_with_string_value value
@@decorator_with_number_value 4
@@decorator_with_boolean_value true
@@decorator_with_multiple_values value1,value2,value3
@@decorator_without_value

Some lorebook content
```

Decorators can have fallback decorators. if the decorator is not recognized by the application, or decorator is ignored, the application *SHOULD* check if the fallback decorator is present. if the fallback decorator is present, the application *SHOULD* consider the fallback decorator instead of the original decorator, if not, the application *SHOULD* ignore the fallback decorator.

fallback decorators uses three `@` instead of two, and it would be right after the original decorator. for example, if you want `@@risu_only_decorator 4` to be fallback of `@@activate_after 4`, and `@@depth 5` to be fallback of `@@instruct_depth 5`, you would write it like this:

```
@@risu_only_decorator 4
@@@activate_after 4
@@depth 5
@@@instruct_depth 5
```

This way, if the application recognize `@@risu_only_decorator`, it would consider the `@@risu_only_decorator`, and if it doesn't, it would consider the `@@activate_after`.

applications *SHOULD* support multiple fallback decorators like below. this way, it would be checked from top to bottom. for this example below, if the application doesn't recognize `@@risu_only_decorator`, it would check if the application recognize `@@agn_only_decorator`, and if it doesn't, it would check if the application recognize `@@activate_after`. applications *MAY* limit chainings of fallback decorators to certain number, but it *SHOULD* be at least 5.

```
@@risu_only_decorator 4
@@@agn_only_decorator 4
@@@activate_every 4
```

On backfilling V2, the application *SHOULD* remove all decorators.

### `@@activate_only_after`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, the lorebook entry *SHOULD NOT* considered as a match until the chat log's assistant (or character's) message count is equal to or greater than the value of this decorator. If the context isn't chat based, the application *SHOULD* ignore this decorator.

If how chat logs are counted are impossible to determine, instead of following the above rule, the application *SHOULD NOT* consider the lorebook entry as a match after user's input is received [decorator's value]th time. if that isn't also possible, the application *SHOULD* ignore this decorator.

### `@@activate_only_every`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, the lorebook entry *SHOULD NOT* be considered as a match if the chat log's assistant (or character's) message count divided by the value of this decorator has a remainder. If the context isn't chat based, the application *SHOULD* ignore this decorator.

If how chat logs are counted are impossible to determine, the application *SHOULD* activate.

If how chat logs are counted are impossible to determine, instead of following the above rule, the application *SHOULD NOT* consider the lorebook entry as a match after user's input is received [decorator's value]th time, and reset the counter after the lorebook entry is considered as a match in this decorator's conditions. if that isn't also possible, the application *SHOULD* ignore this decorator.

### `@@keep_activate_after_match`

This decorator does not have a value. if this decorator is present, the lorebook entry *SHOULD* considered as a match *IN ANY CASE* if the lorebook entry is considered as a match before more than once. if checking previous matches is impossible, the application *SHOULD* ignore this decorator.

### `@@dont_activate_after_match`

This decorator does not have a value. if this decorator is present, the lorebook entry *SHOULD NOT* considered as a match *IN ANY CASE* if the lorebook entry is considered as a match before more than once. if checking previous matches is impossible, the application *SHOULD* ignore this decorator.

### `@@depth`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, the `content` field *SHOULD* be added to the prompt chat logs [decorator's value]th message counted from most recent message to the oldest message. If the context isn't chat based, the application *SHOULD* ignore this decorator. If the value of this decorator is greater than the total message count, the application *SHOULD* add the content field before the oldest message. if the value of this decorator is less than 1, the application *SHOULD* add the content field after the most recent message. If decorator `@@after` is present, the application *SHOULD* ignore this decorator.

if decorator's value is 0, and `@@role` decorator's value is `assistant`, and client's environment supports prefill, the application *SHOULD* insert as a prefill message. if there is more than one prefill message, and the client's environment only supports one prefill message, the application *SHOULD* use concat all prefill messages. this behavior *MAY* be enabled or disabled by the user.

Additionally, if decorator is value is 0, and [it's role is not `assistant` or client's environment does not support prefill], the application *MAY* able users to position the content position in the prompt.

if applications are not possible to insert the content field into the chat logs, the application *SHOULD* follow this rule instead:
- if the value of this decorator is greater than the total message count, the application *SHOULD* put the content field into the prompt, in a low priority position.
- else, the application *SHOULD* put the content field into the prompt, in a high priority position.

How the high priority and low priority positions are located is determined by the application.

If `@@position` decorator is present, the application *SHOULD* ignore this decorator.

### `@@instruct_depth`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, the `content` field *SHOULD* be added to the prompt chat logs [decorator's value]th tokens counted from most recent text to oldest text. If the context is chat based,  the application *SHOULD* ignore this decorator. If the value of this decorator is greater than the total token count, the application *SHOULD* add the content field before the oldest text. if the value of this decorator is less than 1, the application *SHOULD* add the content field after the most recent text.

if decorator's value is 0, and client's environment supports prefill, the application *SHOULD* insert as a prefill message. this behavior *MAY* be enabled or disabled by the user or the application.

Additionally, if decorator is value is 0, and client's environment does not support prefill, the application *MAY* able users to position the content position in the prompt.

if applications are not possible to insert the content field into the chat logs, the application *SHOULD* follow this rule instead:
- if the value of this decorator is greater than the total message count, the application *SHOULD* put the content field into the prompt, in a low priority position.
- else, the application *SHOULD* put the content field into the prompt, in a high priority position.

How the high priority and low priority positions are located is determined by the application.

If `@@position` decorator is present, the application *SHOULD* ignore this decorator.

### `@@reverse_depth`

Same as `@@depth`, but the counting is reversed. it *MUST* behave like `@@depth <total message count> - [decorator's value]`. if the context is chat based, the application *SHOULD* ignore this decorator.

If `@@position` or `@@depth` decorator is present, the application *SHOULD* ignore this decorator.

### `@@reverse_instruct_depth`

Same as `@@instruct_depth`, but the counting is reversed. it *MUST* behave like `@@instruct_depth <total token count> - [decorator's value]`. if the context is chat based, the application *SHOULD* ignore this decorator.

If `@@position` or `@@instruct_depth` decorator is present, the application *SHOULD* ignore this decorator.

### `@@role`

This decorator's value *MUST* be "assistant", "system" or "user". if this decorator is present, the lorebook entry *SHOULD* consider `context` field as a role of decorator's value. If the context isn't chat based, or the client's environment doesn't support roles, the application *SHOULD* ignore this decorator.

### `@@scan_depth`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, regardless of the `scan_depth` field in the lorebook, checking a match in `keys` field in the lorebook entries *SHOULD* only if the recent [decorator's value] messages contain the keys. If the context isn't chat based, or is impossible to determine the recent messages, the application *SHOULD* ignore this decorator.

### `@@instruct_scan_depth`

The value of this decorator *MUST* be a number, and it does not have multiple values. if this decorator is present, regardless of the `scan_depth` field in the lorebook, tchecking a match in `keys` field in the lorebook entries *SHOULD* only if the recent [decorator's value] tokens contain the keys. If the context is chat based, or is impossible to determine the recent tokens, the application *SHOULD* ignore this decorator.

### `@@is_greeting`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, the lorebook entry *MUST NOT* considered as a match if the active greeting's index is not equal to the value of this decorator. the index *MUST* start from 0. for example: if it uses default greeting (`first_msg`), the value of this decorator *MUST* be 0. if it uses first element of `alternate_greetings`, the value of this decorator *MUST* be 1, and if it uses second element of `alternate_greetings`, the value of this decorator *MUST* be 2, and so on.

If the application does not support greetings, or checking the active greeting is not possible, the application *SHOULD* ignore this decorator.

### `@@position`

This decorator's value *MUST* be a string. if this decorator is present, the lorebook's `content` field's value *SHOULD* be added to corresponding position in the prompt.

applications can choose what positions to support. all positions are optional. however, if the application  doesn't support the value of this decorator, the application *SHOULD* ignore this decorator.

The corresponding position is determined as follows:
- if the value of this decorator is `"after_desc"`, the lorebook's `content` field's value *SHOULD* be added after the `description` field of CharacterCardV3 Object.
- if the value of this decorator is `"before_desc"`, the lorebook's`content` field's value *SHOULD* be added before the `description` field of CharacterCardV3 Object.
- if the value of this decorator is `personality`, the lorebook's `content` field's value *SHOULD* be added on the personality section of the prompt. if the personality section does not exist, the application *SHOULD* ignore this decorator.
- if the value of this decorator is `scenario`, the lorebook's `content` field's value *SHOULD* be added on the scenario section of the prompt. if the background section does not exist, the application *SHOULD* ignore this decorator.

applications *MAY* add more positions. if the value is not recognized by the application, the application *SHOULD* ignore this decorator. how the corresponding position is determined *MAY* be editable by the user or the application.

### `@@ignore_on_max_context`

This decorator does not have a value. if this decorator is present, the lorebook entry *SHOULD NOT* considered a match when context reaches maximum tokens, or trimmed first when max context is reached, regardless of the `priority` field and `token_budget` field. This does not mean it doesn't take effect of `priority` field and `token_budget` field.

If the application cannot check the max context is reached, the application *SHOULD* ignore this decorator.

### `@@additional_keys`

This decorator's value *MUST* be a multiple values separated by a comma, as strings. if this decorator is present, the lorebook field *SHOULD NOT* considered as a match if the chat log do not contains one of the strings in decorator's value. how the prompt matches specifically is up to the application, unless other fields and decorators specify otherwise.

if `use_regex` is true, instead of the behavior above, the lorebook field *SHOULD* be considered as a match if the chat log matches one of the value, which *SHOULD* be considered as a regex pattern. if the value in decorator's value has invalid regex patterns the application *MUST* consider the lorebook field as not a match. however, applications *MAY* choose to ignore this field for performance reasons if `use_regex` is true.

decorator that modifies `keys` field's behavior also modifies `additional_keys` field, regardless of `use_regex` field.

This decorator can be used multiple times.

### `@@exclude_keys`

This decorator's value *MUST* be a multiple values separated by a comma, as strings. if this decorator is present, the lorebook field *SHOULD NOT* considered as a match if the chat log contains one of the strings in decorator's value. how the prompt matches specifically is up to the application, unless other fields and decorators specify otherwise.

decorator that modifies `keys` field's behavior also modifies `exclude_keys` field. if `use_regex` is true, this field *SHOULD* be ignored.

### `@@is_user_icon`

This decorator's value *MUST* be a string and it does not have multiple values. if this decorator is present, the lorebook entry *SHOULD NOT* be considered as a match if the active user icon's `name`, specified in the `asset` field in the CharacterCardV3 Object, is not equal to the value of this decorator. if the application does not support user icons, the application *SHOULD* ignore this decorator.

### `@@dont_activate`

This decorator does not have a value. if this decorator is present, the lorebook field *SHOULD NOT* be considered as a match *IN ANY CASE* if `@@activate` decorator is not present. if `@@activate` decorator is present, this decorator *SHOULD* be ignored.

### `@@activate`

This decorator does not have a value. if this decorator is present, the lorebook field *SHOULD* be considered as a match *IN ANY CASE*.

### `@@disable_ui_prompt`

This decorator's value *MUST* be a string. if this decorator is present, the application *MAY* disable the UI prompt which type is equal to the value of this decorator.

types or the UI prompts are:
- `post_history_instructions`
- `system_prompt`

This *MUST* not taken to mean that the application should support these UI prompt types. if the application does not support the UI prompts, the application *SHOULD* ignore this decorator.

applications *MAY* add more types of UI prompts. if the value is not recognized by the application, the application *SHOULD* ignore this decorator.

## Curly Braced Syntaxes

Curly braced syntaxes, shortly CBS, also known as macros, are used to replace the values in the prompt to specific values.

applications are allowed to use these CBS everywhere they want, but the applications *MUST* replace the CBS with the value that are defined in the spec. applications *MAY* add more CBS. however, the added CBS *MUST* start with `{{` and end with `}}` to prevent conflicts with plain text and HTML. the CBS *SHOULD* be detected case insensitive.

### `{{char}}`

This CBS *MUST* be replaced with the value of the `nickname` field in the CharacterCard object. however, if the `nickname` field is undefined or empty string, the value of this CBS *MUST* be replaced with the value of the `name` field in the CharacterCard object.

### `{{user}}`

This CBS *MUST* be replaced with the client's set display name for user or current persona.

### `{{random:A,B,C...}}`

This CBS *MUST* be replaced with a random value from the values that are separated by a comma. for example, `{{random:Hello,Hi,Hey}}` *MUST* be replaced with either `Hello`, `Hi` or `Hey`. `,` can be escaped with `\,`.

### `{{pick:A,B,C...}}`

This CBS *MUST* be replaced with a random value from the values that are separated by a comma. for example, `{{pick::Hello,Hi,Hey}}` *MUST* be replaced with either `Hello`, `Hi` or `Hey`. `,` can be escaped with `\,`. the client *SHOULD* make effort to make the value of this CBS replaced with same value for the same prompt.

### `{{roll:N}}`

This CBS *MUST* be replaced with a random number between 1 and N. for example, `{{roll:6}}` *MUST* be replaced with a random number between 1 and 6. N can also start with `d` as case insensitive. for example, `{{roll:d6}}` *MUST* be replaced with a random number between 1 and 6.

### `{{// A}}`

This CBS *MUST* be replaced with an empty string, and *SHOULD NOT* be displayed to the user, regardless of the value of A. this value *SHOULD NOT* be used on matching lorebook entries.

### `{{hidden_key:A}}`

Same as `{{// A}}`, but the value *SHOULD* be used on matching lorebook entries if its scanned recursively.

### `{{comment: A}}`

This CBS *MUST* be replaced with an empty string when sending as a prompt. and *SHOULD* be displayed to the user as inline comment with content A if its in message. this value *SHOULD NOT* be used on matching lorebook entries.


### `{{reverse:A}}`

This CBS *MUST* be replaced with the reversed version of the value of A. for example, `{{reverse:Hello}}` *MUST* be replaced with `olleH`.
