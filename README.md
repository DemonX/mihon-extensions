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
| KumoManga | 🇹🇭 Thai | 1.4.8 | ✅ | [kumomanga.net](https://kumomanga.net) |
| NanoManga | 🇹🇭 Thai | 1.4.3 | ❌ | [nano-manga.com](https://nano-manga.com) |
| MangaJapan | 🇹🇭 Thai | 1.4.1 | ✅ | [มังงะญี่ปุ่น.com](https://xn--72cas2cj6a4hf4b5a8oc.com) |
| NiceOppaiTH | 🇹🇭 Thai | 1.4.2 | ✅ | [niceoppai.net](https://www.niceoppai.net) |

---

## KumoManga — kumomanga.net

Reads manga from [kumomanga.net](https://kumomanga.net/) (Thai WordPress-based manga site).

### Features
- 📚 Browse popular manga (sorted by chapter count)
- 🕒 Latest updates feed
- 🔍 Search by title
- 📖 Chapter reader with page images

### Notes on thumbnails
- **Latest Updates tab** — manga covers load automatically
- **Popular / Search tab** — covers appear after you open a manga's detail page once; Tachimanga caches them permanently after that
- **Manga detail page** — cover always loads when you open a manga

### How chapter pages work
Chapter images are served through a proxy (`bbb.webtoon168.com`) to bypass CDN restrictions — this is handled automatically by the extension.

### Changelog

| Version | Changes |
|---------|---------|
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
| 1.4.2 | Rename to NiceOppaiTH to avoid conflicts with other repos |
| 1.4.1 | Initial release |
