# Anki Japanese

Anki collection package for learning Japanese:

1. Vocabulary
1. Kanji
1. Grammar

## Export and import

**Export:**

1. _Anki > File > Export..._
   - Export format: _Anki Collection Package (.colpkg)_
   - Include: _Include media_
1. Click _Export..._

> **Note:** the above has to be done from the main Anki window (small window showing all decks), not from the _Browse_ window (which shows all notes, note types, etc.).

**Import:**

1. _Anki > File > Import..._
1. Select the `japanese.colpkg` file
1. Follow import dialog

> **Note:** the above will replace the entire local Anki content with the content of the imported `.colpkg` file.

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
    echo "$value" >"$key".json
  done
```

> **Note:** the above creates a separate JSON file for each entry. The file name is `<kanji>.json` (e.g. `日.json`) and the content is the the KanjiAPI data for the corresponding kanji.

## Dev log

### 2025-03-18

- Regarding incorporation of KanjiAPI (<https://kanjiapi.dev/>) data in cards
  - Cannot make JavaScript web requests from cards (sandboxed environment)
  - Complete KanjiAPI data is 98 MB in size and contains 13,108 kanjis
  - Envisioned solution: split KanjiAPI data into separate files (one file for each kanji), place these files in Anki's media folder, then access the relevant files from the cards.

### 2025-03-02

- Installed font [Kanji stroke order font v4.004](https://sites.google.com/site/nihilistorguk/) (see [Custom fonts](#custom-fonts) above)
