# Anki Japanese

Anki deck package for Japanese words.

## Export

_Anki > Notes > Export Notes..._

## Import

Double-click on `anki-japanese.apkg` file and follow import instructions.

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
