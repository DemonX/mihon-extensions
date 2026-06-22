# DemonX Mihon Extensions

Custom Mihon/Tachimanga extension repository for Thai manga.

## Add to Mihon / Tachimanga

**More → Settings → Browse → Extension repos** → Add:

```
https://raw.githubusercontent.com/DemonX/mihon-extensions/main/index.json
```

Then go to **Browse → Extensions** and install **KumoManga**.

---

## Extensions

| Name | Language | Version | NSFW |
|------|----------|---------|------|
| KumoManga | 🇹🇭 Thai | 1.4.8 | ✅ |

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

---

## Changelog

| Version | Changes |
|---------|---------|
| 1.4.8 | Fix manga list page timeout; remove slow per-category thumbnail requests from browse/search |
| 1.4.7 | Per-category thumbnail requests (reverted — caused timeout) |
| 1.4.6 | Embed covers in browse/search listings via fetchThumbnailMap |
| 1.4.5 | Fix Observable.zip iOS incompatibility in manga details |
| 1.4.4 | Fix "No pages found" — regex on raw HTML + noscript bypass; backward-compat URL fix |
| 1.4.3 | Fix CDN blocking via bbb.webtoon168.com proxy for chapter pages |
| 1.4.2 | Fix missing manga title default causing silent serialization error |
| 1.4.1 | Initial release |
| NanoManga | 🇹🇭 Thai | 1.4.1 | ❌ |
