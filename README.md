# Anki Japanese

Anki deck package for Japanese words.

## Export and import

Export:

> _Anki > Notes > Export Notes..._

Import:

> Double-click on `anki-japanese.apkg` file and follow import instructions.

## Custom fonts

Custom fonts can be installed directly into Anki. In that case, they will be automatically synced to AnkiDroid too.


To install and use a custom font, proceed as follows:

1. Prepend an underscore to the font's `.ttf` file and copy it into `~/Library/Application\ Support/Anki2/User\ 1/collection.media`
   - For example:
     ```
     ~/Library/Application\ Support/Anki2/User\ 1/collection.media/_KanjiStrokeOrders_v4.004.ttf
     ```
1. In the CSS file of the card templates, declare the custom font as follows:
    ```css
    @font-face {
      font-family: MyName;
      src: url("_KanjiStrokeOrders_v4.004.ttf");
    }
    ```
1. Now, you can use the font in the CSS file as follows:
    ```css
    font-family: MyName;
    ```

> **Note:** `MyName` may be any arbitrary name.

Instructions:

- Anki: <https://docs.ankiweb.net/templates/styling.html#installing-fonts>
- AnkiDroid: <https://ankidroid.org/manual.html#customFonts>

2025-03-02 installed font:

- [Kanji stroke order font v4.004](https://sites.google.com/site/nihilistorguk/)

## Debugging

1. Make sure the [AnkiWebView Inspector](https://ankiweb.net/shared/info/31746032) add-on is installed.
1. Right-click on any element in a preview window and select _Inspect_

## Dev notes

### 2025-03-18

- Regarding incorporation of KanjiAPI (<https://kanjiapi.dev/>) data in cards
  - Cannot make JavaScript web requests from cards (sandboxed environment)
  - Complete KanjiAPI data is 98 MB in size and contains 13,108 kanjis
  - Envisioned solution: split KanjiAPI data into separate files (one file for each kanji), place these files in Anki's media folder, then access the relevant files from the cards.
