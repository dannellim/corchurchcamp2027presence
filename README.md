# COR Church Camp 2027 · Presence

Save-the-date website for **COR Church Camp 2027** by Chapel of the Resurrection (复活堂), organised by the El-Pizo cell group. The site is in English and Simplified Chinese.

- **When:** Saturday 29 – Monday 31 May 2027 (3 days, 2 nights)
- **Where:** Four Points by Sheraton Desaru, Bandar Penawar, Johor, Malaysia

**Live site:**

| Link | Opens |
|---|---|
| https://dannellim.github.io/corchurchcamp2027presence/en/ | English |
| https://dannellim.github.io/corchurchcamp2027presence/zh/ | 中文 (Simplified Chinese) |
| https://dannellim.github.io/corchurchcamp2027presence/ | The language of the visitor's phone |

## How the two languages work

There is one page, `index.html`, so both languages always share the same design. The English text is written in the HTML. Each label carries a key, such as `data-i18n="hero.title"`, and the Chinese for every key is in the `ZH` table in the script at the bottom of `index.html`. Attributes such as image descriptions and links use `data-i18n-alt`, `data-i18n-href` and so on.

- The **English / 中文** switch at the top of the page changes the labels in place.
- `?lang=zh` or `?lang=en` on the link picks the language. With neither, the page follows the phone's language.
- `en/` and `zh/` are short links that open the page in that language. Each has its own link preview for WhatsApp.

To change wording, edit the English in the HTML or the Chinese in the `ZH` table. A key missing from the table simply shows the English.

## What's here

| File | What it is |
|---|---|
| `index.html` | The whole site: one self-contained page (no build step, no external scripts or fonts) |
| `en/index.html`, `zh/index.html` | Short links that open the site in English or Chinese |
| `assets/logo.svg` | Camp badge (Presence · Desaru, Malaysia) |
| `assets/og.png`, `assets/og-zh.png` | Link preview images (English, Chinese) for WhatsApp and social media |
| `assets/cor-church-camp-2027.ics`, `assets/cor-church-camp-2027-zh.ics` | Calendar invites (English, Chinese) for Apple Calendar and Outlook |
| `assets/hotel.jpg`, `pool.jpg`, `room.jpg` | Hotel photos © Four Points by Sheraton Desaru (Marriott) |
| `assets/committee.jpg` | Camp committee group photo |
| `assets/qr-google.svg`, `qr-apple.svg`, `qr-waze.svg` | QR codes for directions to the hotel |
| `assets/qr-website-en.svg`, `qr-website-zh.svg` | QR codes that open the English or Chinese site |

## Updating the site

Edit the files and push to `main`. GitHub Pages republishes automatically within a minute or two.
