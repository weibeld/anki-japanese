# KanjiAPI Notes

- Home: <https://kanjiapi.dev/>
- API base URL: <https://kanjiapi.dev/v1/kanji/>
  - Example: <https://kanjiapi.dev/v1/kanji/日>
- Full data download: <https://kanjiapi.dev/kanjiapi_full.zip>

## API data

### Full data format

```json
{
  "kanjis": {
    "日": {...},
    "本": {...},
    ...
  },
}
```

> **Note:** the `{...}` objects are the objects returned by the above API endpoint.

### Full data processing commands

Transforming into list:

```bash
cat kanjiapi_full.json | jq '.kanjis | to_entries' >data.json
```

> **Note:** this transforms the KanjiAPI data into a JSON list of objects with `"key"` and `"value"` fields, where `"key"` is the kanji and `"value"` is the  API entry for that kanji. This makes the further processing simpler.

Counting entries:

```bash
cat data.json | jq length
```

> **Note:** at the time of this writing, there were 13,108 entries in the data.

Listing CJK Compatibility code block entries:

```bash
cat data.json | jq '[ .[] | select(.value.unihan_cjk_compatibility_variant) ]'
```
> **Note:** the above command lists all entries with the `unihan_cjk_compatibility_variant` field. At the time of this writing, 75 of the 13,108 entries have this field. Entries with this field are code points in the [CJK Compatibility](https://en.wikipedia.org/wiki/CJK_Compatibility) code block. These kanjis already have a coresponding "real" kanji in the data and they are regarded as duplicates of these "real" kanjis by many text processors (see [KanjiAPI documentation](https://github.com/onlyskin/kanjiapi.dev?tab=readme-ov-file#list-of-jinmeiyo-kanji)). Therefore, it's best to filter out all the entries with the `unihan_cjk_compatibility_variant` field.

Filtering out CJK Compatibility code block entries:

```bash
cat data.json | jq '[ .[] | select(.value.unihan_cjk_compatibility_variant == null) ]' >data-clean.json
```

> **Note:** the above creates a cleaned data set which does not contain any entries with the `unihan_cjk_compatibility_variant` field. At the time of this writing, 13,033 entries remain in this cleaned data set.

Splitting into files:

```bash
cat data-clean.json | jq -r '.[] | "\(.key)=\(.value)"' |
  while IFS='=' read key value; do
    echo "$value" >"$key".json
  done
```

> **Note:** the above creates a separate JSON file for each entry. The file name is `<kanji>.json` (e.g. `日.json`) and the content is the the KanjiAPI data for the corresponding kanji.
