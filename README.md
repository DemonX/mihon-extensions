# DemonX Mihon Extensions

Custom Mihon/Tachimanga extension repository for Thai manga.

---

## Add to Your App

### Mihon (Android)
**More → Settings → Browse → Extension repos** → Add:
```
https://raw.githubusercontent.com/DemonX/mihon-extensions/main/index.min.json
```

### Tachimanga / Aniyomi (iOS)
**More → Settings → Browse → Extension repos** → Add:
```
https://raw.githubusercontent.com/DemonX/mihon-extensions/main/index.min.json
```

Then go to **Browse → Extensions** and install the extension(s) you want.

---

## Extensions

| Name | Language | Version | NSFW | Site |
|------|----------|---------|------|------|
| KumoManga | 🇹🇭 Thai | 1.4.10 | ✅ | [kumomanga.com](https://kumomanga.com) |
| NanoManga | 🇹🇭 Thai | 1.4.3 | ❌ | [nano-manga.com](https://nano-manga.com) |
| MangaJapan | 🇹🇭 Thai | 1.4.1 | ✅ | [มังงะญี่ปุ่น.com](https://xn--72cas2cj6a4hf4b5a8oc.com) |
| NiceOppaiTH | 🇹🇭 Thai | 1.4.4 | ✅ | [niceoppai.net](https://www.niceoppai.net) |
| MangaNeko | 🇹🇭 Thai | 1.4.2 | ❌ | [manga-neko.com](https://manga-neko.com) |
| OreManga | 🇹🇭 Thai | 1.4.3 | ✅ | [oremanga.net](https://www.oremanga.net) |

---

## KumoManga — kumomanga.com

Reads manga from [kumomanga.com](https://kumomanga.com/) (Thai manga site, rebuilt as Next.js SSR).

### Features
- 📚 Browse popular manga
- 🕒 Latest updates feed
- 🔍 Search by title
- 📖 Chapter reader with page images

### Changelog

| Version | Changes |
|---------|---------|
| 1.4.10 | Full rewrite — kumomanga.com relaunched as Next.js site (HTML scraping, new URL structure) |
| 1.4.8 | Fix manga list page timeout; remove slow per-category thumbnail pre-fetch |
| 1.4.7 | Per-category thumbnail requests (reverted — caused timeout) |
| 1.4.6 | Embed covers in browse/search listings |
| 1.4.5 | Fix Observable.zip iOS incompatibility in manga details |
| 1.4.4 | Fix "No pages found" — regex on raw HTML + noscript bypass |
| 1.4.3 | Fix CDN blocking via bbb.webtoon168.com proxy for chapter pages |
| 1.4.2 | Fix missing manga title default causing silent serialization error |
| 1.4.1 | Initial release |

---

## NanoManga — nano-manga.com

Reads manga from [nano-manga.com](https://nano-manga.com/) (Thai Madara-theme WordPress site).

### Features
- 📚 Browse popular manga
- 🕒 Latest updates feed
- 🔍 Search by title
- 📖 Chapter reader with page images

### Changelog

| Version | Changes |
|---------|---------|
| 1.4.3 | Fix chapter list URL (missing slash before ajax/chapters/) |
| 1.4.2 | Fix search results using wrong HTML selector |
| 1.4.1 | Initial release |

---

## MangaJapan — มังงะญี่ปุ่น.com

Reads manga from [มังงะญี่ปุ่น.com](https://xn--72cas2cj6a4hf4b5a8oc.com/) (Thai WordPress "mangastream" theme site, 18+).

### Features
- 📚 Browse popular manga
- 🕒 Latest updates feed
- 🔍 Search by title
- 📖 Chapter reader with page images

### Notes
- Chapter images are loaded from `ts_reader.run(...)` JavaScript — handled automatically.
- Contains adult/hentai/doujin content (NSFW).

### Changelog

| Version | Changes |
|---------|---------|
| 1.4.1 | Initial release |

---

## NiceOppaiTH — niceoppai.net

Reads manga from [niceoppai.net](https://www.niceoppai.net/) (Thai custom WordPress manga site, NSFW).

### Features
- 📚 Browse popular manga
- 🕒 Latest updates feed
- 🔍 Search by title
- 📖 Chapter reader with page images
- ⚠️ Contains adult content (NSFW)

### Changelog

| Version | Changes |
|---------|---------|
| 1.4.4 | Fix chapter image loading for chapters using obfuscated JS (p,a,c,k,e,d packer) rendering |
| 1.4.3 | Fix search: use autocomplete GET API (/wpm-ajx/mng-sch-lst/?q=) — no Cloudflare blocking |
| 1.4.2 | Rename to NiceOppaiTH to avoid conflicts with other repos |
| 1.4.1 | Initial release |

---

## MangaNeko — manga-neko.com

Reads manga from [manga-neko.com](https://manga-neko.com/) (Thai manga site using the MangaReader theme).

### Features
- 📚 Browse popular manga
- 🕒 Latest updates feed
- 🔍 Search by title
- 📖 Chapter reader with page images

### Notes
- Images are loaded from `ts_reader.run()` JSON data embedded in the chapter page
- Chapter dates use Thai month names (auto-translated to English for parsing)

### Changelog

| Version | Changes |
|---------|---------|
| 1.4.1 | Initial release |

---

## OreManga — oremanga.net

Reads manga from [oremanga.net](https://www.oremanga.net/) (Thai manga site with a large catalog, NSFW).

### Features
- Browse popular manga
- Latest updates feed
- Search by title
- Advanced search with filters (status, type, order, genre)
- Chapter reader with page images
- Thai date parsing (absolute and relative dates)

### Changelog

| Version | Changes |
|---------|---------|
| 1.4.3 | Fix reader: select images from `div.reader-area-main`; collect both plain `<img>` and `<canvas data-url>` elements in DOM order |
| 1.4.2 | Fix manga listing: correct URL pattern, item selector (`div.flexbox2-item`), field names (`order`/`title`), chapter/detail selectors |
| 1.4.1 | Initial release |
