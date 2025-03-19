# Anki Japanese

Anki collection package for learning Japanese:

1. Vocabulary
1. Kanji
1. Grammar

## Export and import

Export:

1. _Anki > File > Export..._
  - Export format: _Anki Collection Package (.colpkg)_
  - Include: _Include media_
1. _Export..._

> **Note:** the above has to be done from the main Anki window (small window showing all decks), not from the _Browse_ window (which shows all notes, note types, etc.).

Import:

1. _Anki > File > Import..._
1. Select the `japanese.colpkg` file
1. Follow import dialog

> **Note:** the above will replace the entire local Anki content with the content of the imported `.colpkg` file.

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

Resources:

- Anki: <https://docs.ankiweb.net/templates/styling.html#installing-fonts>
- AnkiDroid: <https://ankidroid.org/manual.html#customFonts>

## JavaScript debugging

1. Make sure the [AnkiWebView Inspector](https://ankiweb.net/shared/info/31746032) add-on is installed
1. In the card preview window, right-click on any element in a preview window and select _Inspect_

## See also

- [KanjiAPI Notes](#kanjiapi.md)

## Dev log

### 2025-03-18

- Regarding incorporation of KanjiAPI (<https://kanjiapi.dev/>) data in cards
  - Cannot make JavaScript web requests from cards (sandboxed environment)
  - Complete KanjiAPI data is 98 MB in size and contains 13,108 kanjis
  - Envisioned solution: split KanjiAPI data into separate files (one file for each kanji), place these files in Anki's media folder, then access the relevant files from the cards.

### 2025-03-02

- Installed font [Kanji stroke order font v4.004](https://sites.google.com/site/nihilistorguk/) (see [Custom fonts](#custom-fonts) above)
