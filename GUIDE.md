# Extension Build & Publish Guide

> **For AI assistants in new sessions:** Read this entire file first. It contains all paths, commands, and context needed to continue work without asking the user.

---

## ⚠️ Golden Rule

**Every code change — no matter how small — MUST be fully rebuilt and republished before the task is considered done.**

This means after every bug fix or feature change:
1. Bump `extVersionCode` in `build.gradle`
2. Build the APK (`./gradlew :src:LANG:EXTNAME:assembleRelease`)
3. Update `index.json` (apk filename, code, version)
4. Update `README.md` (version in extensions table + add changelog entry)
5. `git push` to GitHub

The user installs from GitHub. If changes are not pushed, the user sees no difference and the fix is wasted.

---

## Environment Overview

| Resource | Path |
|----------|------|
| Extension source files | `/Users/kittiponkanda/Works/Personal/comic/src/th/` |
| Build environment (keiyoushi clone) | `/tmp/extensions-source/` |
| GitHub repo working directory | `/tmp/mihon-extensions/` |
| GitHub repo URL | `https://github.com/DemonX/mihon-extensions` |
| Mihon index URL | `https://raw.githubusercontent.com/DemonX/mihon-extensions/main/index.json` |

> ⚠️ `/tmp/` directories are wiped on reboot. If missing, re-clone:
> ```bash
> git clone https://github.com/keiyoushi/extensions-source /tmp/extensions-source
> git clone https://github.com/DemonX/mihon-extensions /tmp/mihon-extensions
> ```

---

## Current Extensions

| Name | Site | Lang | Version | extVersionCode | NSFW | Source ID |
|------|------|------|---------|---------------|------|-----------|
| KumoManga | https://kumomanga.net | th | 1.4.8 | 8 | ✅ | 67285793621557 |
| NanoManga | https://nano-manga.com | th | 1.4.3 | 3 | ❌ | 7438925132 |
| MangaJapan | https://xn--72cas2cj6a4hf4b5a8oc.com | th | 1.4.1 | 1 | ✅ | 5210794566 |

Source files:
- `/Users/kittiponkanda/Works/Personal/comic/src/th/kumomanga/`
- `/Users/kittiponkanda/Works/Personal/comic/src/th/nanomanga/`
- `/Users/kittiponkanda/Works/Personal/comic/src/th/mangajapan/`

---

## Step 1 — Investigate the Website

### Detect Madara theme (WordPress manga theme)
```bash
# Look for madara AJAX endpoint
curl -s "https://SITE.com/wp-admin/admin-ajax.php" -d "action=madara_load_more" -o /dev/null -w "%{http_code}"
# 200 or 400 → Madara theme ✅

# Confirm in HTML source
curl -s "https://SITE.com" | grep -i "madara\|wp-manga\|page-item-detail" | head -5
```

Signs of Madara in HTML:
- `div.page-item-detail` in manga listings
- `div.post-title`, `div.summary_image` on manga pages
- `li.wp-manga-chapter` in chapter lists
- `div.reading-content img.wp-manga-chapter-img` in chapter reader

### Detect plain WordPress REST API (categories = manga, posts = chapters)
```bash
curl -s "https://SITE.com/wp-json/wp/v2/categories?per_page=5" | python3 -m json.tool | head -20
# Returns JSON array with id, name, slug, count → plain WordPress structure
```
Example: kumomanga.net uses this pattern.

### Check if chapter images are CDN-blocked
```bash
# Find image CDN domain in chapter page source, then test:
curl -I "https://IMAGE-CDN.com/path/to/image.jpg"
# PROTOCOL_ERROR or empty reply → CDN blocks non-browser clients (TLS fingerprint)
# Fix: look for proxy URL in page HTML source (e.g. bbb.webtoon168.com/file/cmd3.weimanga.com/...)
```

### Check if images are inside `<noscript>` tags
```bash
curl -s "https://SITE.com/chapter-url/" | grep -o '<noscript>.*</noscript>' | head -3
# If images are in noscript → Jsoup .select("img") won't work
# Fix: use raw response.body.string() + Regex
```

---

## Step 2 — Choose Extension Pattern

| Site Type | Pattern |
|-----------|---------|
| Madara theme | `themePkg = 'madara'` (3-line Kotlin class) |
| Plain WordPress REST API | Custom `HttpSource` using WP REST API |
| Custom site | Custom `HttpSource` with Jsoup scraping |

---

## Step 3 — Create Extension Files

```bash
# Replace LANG (e.g. th), EXTNAME (e.g. mynewmanga), CLASSNAME (e.g. MyNewManga)
EXT=/Users/kittiponkanda/Works/Personal/comic/src/LANG/EXTNAME
mkdir -p $EXT/res/mipmap-{hdpi,mdpi,xhdpi,xxhdpi,xxxhdpi}
mkdir -p $EXT/src/eu/kanade/tachiyomi/extension/LANG/EXTNAME

# Copy icon placeholder from an existing extension, then replace with site favicon
cp /Users/kittiponkanda/Works/Personal/comic/src/th/kumomanga/res/mipmap-hdpi/ic_launcher.png $EXT/res/mipmap-hdpi/
cp /Users/kittiponkanda/Works/Personal/comic/src/th/kumomanga/res/mipmap-mdpi/ic_launcher.png $EXT/res/mipmap-mdpi/
cp /Users/kittiponkanda/Works/Personal/comic/src/th/kumomanga/res/mipmap-xhdpi/ic_launcher.png $EXT/res/mipmap-xhdpi/
cp /Users/kittiponkanda/Works/Personal/comic/src/th/kumomanga/res/mipmap-xxhdpi/ic_launcher.png $EXT/res/mipmap-xxhdpi/
cp /Users/kittiponkanda/Works/Personal/comic/src/th/kumomanga/res/mipmap-xxxhdpi/ic_launcher.png $EXT/res/mipmap-xxxhdpi/
```

### build.gradle — custom HttpSource
```groovy
ext {
    extName = 'SiteName'
    extClass = '.ClassName'
    extVersionCode = 1
    isNsfw = false
}
apply plugin: "kei.plugins.extension.legacy"
```

### build.gradle — Madara theme (minimal)
```groovy
ext {
    extName = 'SiteName'
    extClass = '.ClassName'
    themePkg = 'madara'
    baseUrl = 'https://SITE.com'
    overrideVersionCode = 1
    isNsfw = false
}
apply plugin: "kei.plugins.extension.legacy"
```

---

## Step 4 — Write the Extension

### Madara theme (minimal Kotlin, ~8 lines)
```kotlin
package eu.kanade.tachiyomi.extension.th.sitename

import eu.kanade.tachiyomi.multisrc.madara.Madara
import java.text.SimpleDateFormat
import java.util.Locale

class SiteName : Madara(
    "SiteName",
    "https://SITE.com",
    "th",
    dateFormat = SimpleDateFormat("dd/MM/yyyy", Locale.US),
)
```

### Custom HttpSource — reference files
- **Madara scraping (no theme):** `/Users/kittiponkanda/Works/Personal/comic/src/th/nanomanga/src/eu/kanade/tachiyomi/extension/th/nanomanga/NanoManga.kt`
- **WordPress REST API:** `/Users/kittiponkanda/Works/Personal/comic/src/th/kumomanga/src/eu/kanade/tachiyomi/extension/th/kumomanga/KumoManga.kt`

### Critical gotchas

**1. CDN blocks non-browser clients (kumomanga.net)**
`cmd3.weimanga.com` returns HTTP/2 PROTOCOL_ERROR for OkHttp/curl (TLS fingerprinting).
Fix: Use proxy URL found in page source:
```kotlin
url.replace("cmd3.weimanga.com", "bbb.webtoon168.com/file/cmd3.weimanga.com")
```

**2. Images inside `<noscript>` tags**
`doc.select("img")` returns nothing. Fix: use raw HTML + Regex:
```kotlin
val html = response.body.string()
val readerArea = html.substringAfter("id=\"readerarea\"").substringBefore("</div>")
val imgRegex = Regex("""src=["'](https?://[^"']+)["']""")
val pages = imgRegex.findAll(readerArea)
    .map { it.groupValues[1].replace("\\/", "/") }
    .toList()
```

**3. kotlinx.serialization silent failures**
Missing JSON fields throw inside `runCatching` silently. Always add defaults:
```kotlin
@Serializable data class WPPost(
    val id: Int,
    val title: WPTitle = WPTitle(""),   // ← must have default
    val content: WPContent? = null,
)
```

**4. iOS Tachimanga: avoid Observable.zip**
Use blocking calls instead:
```kotlin
// ❌ Bad for iOS
Observable.zip(req1, req2) { a, b -> ... }

// ✅ Good
override fun fetchMangaDetails(manga: SManga): Observable<SManga> {
    val result = client.newCall(request).execute().parseAs<...>()
    return Observable.just(result.toSManga())
}
```

**5. Thumbnail strategy for WordPress REST API sites**
- WP categories have no thumbnail in the API
- `fetchMangaDetails` only runs when user opens a manga — not during browse grid display
- For Latest Updates: thumbnails come free from post content (first `<img>` in post HTML)
- For Popular/Search: thumbnails populate after user opens each manga once (Mihon caches permanently)
- Making one API call per category to pre-fetch thumbnails causes timeout (~2.5s × 20 = 50s)

---

## Step 5 — Build the APK

```bash
# 1. Copy source files to build environment
cp -r /Users/kittiponkanda/Works/Personal/comic/src/LANG/EXTNAME \
      /tmp/extensions-source/src/LANG/

# Or for an existing extension update:
cp /Users/kittiponkanda/Works/Personal/comic/src/LANG/EXTNAME/src/eu/kanade/tachiyomi/extension/LANG/EXTNAME/ClassName.kt \
   /tmp/extensions-source/src/LANG/EXTNAME/src/eu/kanade/tachiyomi/extension/LANG/EXTNAME/
cp /Users/kittiponkanda/Works/Personal/comic/src/LANG/EXTNAME/build.gradle \
   /tmp/extensions-source/src/LANG/EXTNAME/

# 2. Build
cd /tmp/extensions-source
./gradlew :src:LANG:EXTNAME:assembleRelease

# 3. Find APK (version = 1.4.{extVersionCode})
find /tmp/extensions-source/src/LANG/EXTNAME/build/outputs/apk/release -name "*.apk"
```

---

## Step 6 — Calculate Source ID (for new extensions)

```python
def java_hash_code(s):
    h = 0
    for c in s:
        h = (31 * h + ord(c)) & 0xFFFFFFFF
    if h >= 0x80000000:
        h -= 0x100000000
    return h

lang = "th"
name = "SiteName"   # must exactly match `override val name` in Kotlin class
source_id = (1 << 32) | (java_hash_code(lang + name) & 0xFFFFFFFF)
print(source_id)
```

---

## Step 7 — Publish New Extension

```bash
cd /tmp/mihon-extensions

# 1. Copy APK
cp /tmp/extensions-source/src/LANG/EXTNAME/build/outputs/apk/release/*.apk apk/

# 2. Copy icon (xhdpi 96×96)
cp /Users/kittiponkanda/Works/Personal/comic/src/LANG/EXTNAME/res/mipmap-xhdpi/ic_launcher.png \
   icon/LANG.EXTNAME.png

# 3. Add to index.json
python3 - << 'EOF'
import json
with open("index.json") as f: d = json.load(f)
d.append({
    "name": "SiteName",
    "pkg": "eu.kanade.tachiyomi.extension.LANG.EXTNAME",
    "apk": "tachiyomi-LANG.EXTNAME-v1.4.1-release.apk",
    "lang": "LANG",
    "code": 1,
    "version": "1.4.1",
    "nsfw": 0,
    "hasReadme": 0,
    "hasChangelog": 0,
    "sources": [{"name": "SiteName", "lang": "LANG", "id": "SOURCE_ID", "baseUrl": "https://SITE.com", "versionId": 1}]
})
with open("index.json", "w") as f: json.dump(d, f, indent=2)
with open("index.min.json", "w") as f: json.dump(d, f, separators=(",", ":"))
EOF

# 4. Update README.md (extensions table + add new section with features/changelog)
python3 - << 'PYEOF'
# Just a reminder — edit README.md manually or via script
# Add row to extensions table and append a new ## ExtName section at the bottom
PYEOF

# 5. Commit and push
git add .
git commit -m "Add SiteName extension v1.4.1"
git push
```

---

## Step 8 — Update Existing Extension

```bash
# 1. Edit source file, then bump extVersionCode in build.gradle
#    e.g. change extVersionCode = 8  →  extVersionCode = 9

# 2. Sync to build env and build
cp .../ClassName.kt /tmp/extensions-source/src/LANG/EXTNAME/.../
cp .../build.gradle /tmp/extensions-source/src/LANG/EXTNAME/
cd /tmp/extensions-source && ./gradlew :src:LANG:EXTNAME:assembleRelease

# 3. Deploy
cd /tmp/mihon-extensions
OLD_APK="tachiyomi-LANG.EXTNAME-v1.4.OLD-release.apk"
NEW_APK="tachiyomi-LANG.EXTNAME-v1.4.NEW-release.apk"
rm apk/$OLD_APK
cp /tmp/extensions-source/src/LANG/EXTNAME/build/outputs/apk/release/$NEW_APK apk/

python3 - << 'EOF'
import json
with open("index.json") as f: d = json.load(f)
entry = next(e for e in d if e["name"] == "SiteName")
entry["apk"] = "NEW_APK_FILENAME"
entry["code"] = NEW_CODE_INT
entry["version"] = "1.4.NEW"
with open("index.json", "w") as f: json.dump(d, f, indent=2)
with open("index.min.json", "w") as f: json.dump(d, f, separators=(",", ":"))
EOF

# 4. Update README.md — bump version in extensions table + add changelog entry
#    e.g. change "1.4.8" → "1.4.9" in the table and add "| 1.4.9 | Fix: ..." row

git add . && git commit -m "Fix: description (vNEW)" && git push
```

---

## Install in Tachimanga (iOS)

1. Open Tachimanga → **More → Settings → Browse → Extension repos**
2. Add this URL:
   ```
   https://raw.githubusercontent.com/DemonX/mihon-extensions/main/index.json
   ```
3. Go to **Browse → Extensions** → install
4. To update an extension: tap **Update** next to it

---

## Version Number Reference

The APK version name is auto-generated as `1.4.{extVersionCode}`.
- `extVersionCode = 1` → `v1.4.1`
- `extVersionCode = 8` → `v1.4.8`

`code` in `index.json` must equal `extVersionCode`.

---

## Repo File Structure

```
mihon-extensions/
├── index.json          ← full extension index (used by Mihon to discover extensions)
├── index.min.json      ← minified copy (always keep in sync with index.json)
├── apk/                ← all APK files
│   ├── tachiyomi-th.kumomanga-v1.4.8-release.apk
│   └── tachiyomi-th.nanomanga-v1.4.1-release.apk
├── icon/               ← 96x96 PNG icons, named {lang}.{extname}.png
│   ├── th.kumomanga.png
│   └── th.nanomanga.png
├── README.md           ← user-facing install instructions
└── GUIDE.md            ← this file (AI + developer reference)
```
