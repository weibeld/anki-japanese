# Anki Japanese

Anki deck with sub-decks for learning Japanese:

1. Vocabulary
   1. `vocab-read`
   1. `vocab-speak`
   1. `vocab-write`
1. Kanji
   - TODO: kanji to keyword (keyword: Heisig and possibly own)
   - TODO: keyword to kanji
   - TODO: kanji to kaki-kata
   - Radicals (not priority)
     - Radical to keyword
     - Keyword to radical
     - Kanji to radical (both 214 and 69)
1. Grammar
   - TODO: grammar item to explanation

## Export and import

**Export:**

1. _Anki > File > Export..._
   - Export format: _Anki Deck Package (.apkg)_
   - Include: `japanese`
   - Uncheck _Include scheduling information_
1. Click _Export..._

> **Note:** the above has to be done from the main Anki window (small window showing all decks), not from the _Browse_ window (which shows all notes, note types, etc.).

**Import:**

1. Import deck
   - _Anki > File > Import..._
   - Select the `japanese.apkg` file
   - Follow import dialog
1. Install media files
   - Copy content of [`media`](media) folder into the Anki [media folder](https://docs.ankiweb.net/files.html#file-locations) (`~/Library/Application\ Support/Anki2/User\ 1/collection.media`)
   
> **Note:** the above will add the `japanese` deck to the current collection.

## JavaScript debugging

1. Make sure the [AnkiWebView Inspector](https://ankiweb.net/shared/info/31746032) add-on is installed
1. In the card preview window, right-click on any element in a preview window and select _Inspect_

## Custom fonts

Custom fonts can be installed directly into Anki. In that case, they will be automatically synced to AnkiDroid too:

1. Download the font as a `.ttf` file
1. Prepend an underscore to file name and copy it into the [media folder](https://docs.ankiweb.net/files.html#file-locations)
   - For example:
     ```
     mv KanjiStrokeOrders_v4.004.ttf _KanjiStrokeOrders_v4.004.ttf
     mv _KanjiStrokeOrders_v4.004.ttf ~/Library/Application\ Support/Anki2/User\ 1/collection.media
     ```
1. Declare the font in the card template CSS as follows:
    ```css
    @font-face {
      font-family: MyName;
      src: url("_KanjiStrokeOrders_v4.004.ttf");
    }
    ```

    > **Note:** `MyName` may be any arbitrary name.
1. Use the font in the card template CSS as follows:
    ```css
    font-family: MyName;
    ```

Resources:

- Anki: <https://docs.ankiweb.net/templates/styling.html#installing-fonts>
- AnkiDroid: <https://ankidroid.org/manual.html#customFonts>

## KanjiAPI data

Kanji data (readings, meanings, JLPT level, etc.) is obtained from [KanjiAPI](https://kanjiapi.dev/):

- Base URL: <https://kanjiapi.dev/v1/kanji/>
  - Example: <https://kanjiapi.dev/v1/kanji/日>
- Full data: <https://kanjiapi.dev/kanjiapi_full.zip>

The data is used by storing a static copy of the full API data (split into individual files per kanji) in the [media folder](https://docs.ankiweb.net/files.html#file-locations).

### Data format

The format of the downloadable full API data (see above) is as follows:

```json
{
  "kanjis": {
    "日": {...},
    "本": {...},
    ...
  },
}
```

> **Note:** the `{...}` objects are the objects returned by the `/kanji/` API endpoint.

### Data processing commands

**Transforming into list:**

```bash
cat kanjiapi_full.json | jq '.kanjis | to_entries' >data.json
```

> **Note:** this transforms the KanjiAPI data into a JSON list of objects with `"key"` and `"value"` fields, where `"key"` is the kanji and `"value"` is the  API entry for that kanji. This makes the further processing simpler.

**Counting entries:**

```bash
cat data.json | jq length
```

> **Note:** at the time of this writing, there were 13,108 entries in the data.

**Listing CJK Compatibility code block entries:**

```bash
cat data.json | jq '[ .[] | select(.value.unihan_cjk_compatibility_variant) ]'
```
> **Note:** the above command lists all entries with the `unihan_cjk_compatibility_variant` field. At the time of this writing, 75 of the 13,108 entries have this field. Entries with this field are code points in the [CJK Compatibility](https://en.wikipedia.org/wiki/CJK_Compatibility) code block. These kanjis already have a coresponding "real" kanji in the data and they are regarded as duplicates of these "real" kanjis by many text processors (see [KanjiAPI documentation](https://github.com/onlyskin/kanjiapi.dev?tab=readme-ov-file#list-of-jinmeiyo-kanji)). Therefore, it's best to filter out all the entries with the `unihan_cjk_compatibility_variant` field.

**Filtering out CJK Compatibility code block entries:**

```bash
cat data.json | jq '[ .[] | select(.value.unihan_cjk_compatibility_variant == null) ]' >data-clean.json
```

> **Note:** the above creates a cleaned data set which does not contain any entries with the `unihan_cjk_compatibility_variant` field. At the time of this writing, 13,033 entries remain in this cleaned data set.

**Splitting into files:**

```bash
cat data-clean.json | jq -r '.[] | "\(.key)=\(.value)"' |
  while IFS='=' read key value; do
    echo "$value" >_japanese_"$key".json
  done
```

> **Note:** the above creates a separate JSON file for each entry. The file name is `_japanese_<kanji>.json` (e.g. `_japanese_日.json`) and the content is the the KanjiAPI data for the corresponding kanji.

## Notes

### Export: Collection Package (.colpkg) vs. Deck Package (.apkg)

See [documentation](https://docs.ankiweb.net/exporting.html):

- A Collection Package exports all decks and all media (even the media that is not used in any cards)
- A Deck Package exports either all or only a single deck
- A Collection Package exports all media in the [media folder](https://docs.ankiweb.net/files.html#file-locations), even files that are not used by any card
- A Deck Package exports only the media that is used in any of the cards of the exported decks
- When importing a Collection Package, all existing Anki content is replaced with the content of the Collection Package
  - Except the existing media in the [media folder](https://docs.ankiweb.net/files.html#file-locations), which is not deleted before importing (see [documentation](https://docs.ankiweb.net/exporting.html#collection-colpkg))
- When importing a Deck Package, the contained deck(s) is/are added to the existing collection
- The idea of a Collection Package is to export and import the entire Anki content (e.g. for sharing or backup)
  - In the case of sharing, a Collection Package may, for example, be imported into a separate [profile](https://docs.ankiweb.net/profiles.html) in Anki
- The idea of a Deck Package is to export individual decks, mainly for sharing
  - A Deck Package can be imported into an existing collection

## Dev log

### 2025-03-19

- Media
  - Prepend `_` KanjiAPI JSON files in media folder to prevent Anki from listing them as "unused files" in the Check Media window (see [documentation](https://docs.ankiweb.net/media.html#checking-media))
  - To distinguish the media of this project in the media folder from media from other decks:
    - Subfolders in the media folder are not allowed (there will be a corresponding message in the Check Media window when attempting to do so)
    - Solution: add common prefix to all media of this project (currently `_japanese_*`)
- Sharing strategy
  - Parent deck `japanese` with all specific decks (`vocab-read`, `vocab-write`, etc.) as sub-decks
  - Export parent deck as Deck Package (`.apkg`). This is the usual way for sharing decks on [Shared Decks](https://ankiweb.net/shared/decks) (see [documentation](https://docs.ankiweb.net/contrib.html#sharing-decks-publicly)).
  - KanjiAPI data files are regarded by Anki as "unused" (because they are accessed by JavaScript and not referenced directly in the note fields). Therefore, request these data files to be installed separately in the media folder by potential users.
    - The custom font seems to be included in the Deck Package (`.apkg`) even though it's only referenced from CSS too

### 2025-03-18

- Regarding incorporation of KanjiAPI (<https://kanjiapi.dev/>) data in cards
  - Cannot make JavaScript web requests from cards (sandboxed environment)
  - Complete KanjiAPI data is 98 MB in size and contains 13,108 kanjis
  - Envisioned solution: split KanjiAPI data into separate files (one file for each kanji), place these files in Anki's media folder, then access the relevant files from the cards.

### 2025-03-02

- Installed font [Kanji stroke order font v4.004](https://sites.google.com/site/nihilistorguk/) (see [Custom fonts](#custom-fonts) above)
