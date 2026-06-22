# Extension Build & Publish Guide

Step-by-step guide for creating a new Mihon/Tachiyomi extension from a website URL.

---

## Prerequisites

- macOS with Homebrew
- JDK 17+ installed (`brew install openjdk@17`)
- Android SDK with build-tools
- keiyoushi/extensions-source cloned at `/tmp/extensions-source/`

```bash
git clone https://github.com/keiyoushi/extensions-source /tmp/extensions-source
```

---

## Step 1 — Investigate the Website

Before writing any code, understand what kind of site it is.

### Check for Madara (WordPress manga theme)
```bash
curl -s "https://SITE.com/wp-admin/admin-ajax.php" -d "action=madara_load_more" -I
# If 200 or 400 → Madara theme ✅
```

Look for these signs in the HTML:
- `div.page-item-detail` in manga listings
- `div.post-title`, `div.summary_image` on manga pages
- `li.wp-manga-chapter` in chapter lists
- `div.reading-content img.wp-manga-chapter-img` in reader

### Check for plain WordPress (REST API)
```bash
curl -s "https://SITE.com/wp-json/wp/v2/categories?per_page=5" | python3 -m json.tool
# If returns JSON array → plain WordPress with categories/posts structure
```

### Check how chapter images are served
```bash
# Open a chapter page, find the image CDN domain
# Test if images are accessible directly:
curl -I "https://IMAGE-CDN.com/path/to/image.jpg"
# If PROTOCOL_ERROR or empty reply → CDN blocks non-browser clients
# Look for proxy alternatives in the page source
```

---

## Step 2 — Choose Extension Pattern

| Site Type | Approach |
|-----------|----------|
| Madara theme | Use `themePkg = 'madara'` in build.gradle (minimal code needed) |
| Plain WordPress (categories=manga, posts=chapters) | Custom `HttpSource` with WP REST API |
| Custom site | Custom `HttpSource` with HTML scraping via Jsoup |

---

## Step 3 — Set Up Extension Files

Create the directory structure:

```
/Users/kittiponkanda/Works/Personal/comic/src/{lang}/{extname}/
├── build.gradle
├── res/
│   ├── mipmap-hdpi/ic_launcher.png     (72x72)
│   ├── mipmap-mdpi/ic_launcher.png     (48x48)
│   ├── mipmap-xhdpi/ic_launcher.png    (96x96)
│   ├── mipmap-xxhdpi/ic_launcher.png   (144x144)
│   └── mipmap-xxxhdpi/ic_launcher.png  (192x192)
└── src/eu/kanade/tachiyomi/extension/{lang}/{extname}/
    └── {ClassName}.kt
```

**Tip:** Copy icons from an existing extension and replace with the site's favicon:
```bash
cp -r /tmp/extensions-source/src/th/kumomanga/res /path/to/new-ext/res
# Replace ic_launcher.png files with site favicon at correct sizes
```

### build.gradle (custom HttpSource)
```groovy
ext {
    extName = 'SiteName'        // Display name
    extClass = '.ClassName'     // Kotlin class name with dot prefix
    extVersionCode = 1          // Integer, increment on each release
    isNsfw = false              // true if site has adult content
}

apply plugin: "kei.plugins.extension.legacy"
```

### build.gradle (Madara theme — even simpler)
```groovy
ext {
    extName = 'SiteName'
    extClass = '.ClassName'
    themePkg = 'madara'         // Uses madara multisrc library
    baseUrl = 'https://SITE.com'
    overrideVersionCode = 1
    isNsfw = false
}

apply plugin: "kei.plugins.extension.legacy"
```

---

## Step 4 — Write the Extension

### Option A: Madara theme (minimal)
If the site uses Madara, create a tiny Kotlin file:

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

### Option B: Custom HttpSource (full control)
See `/Users/kittiponkanda/Works/Personal/comic/src/th/nanomanga/NanoManga.kt` for Madara scraping.
See `/Users/kittiponkanda/Works/Personal/comic/src/th/kumomanga/KumoManga.kt` for WordPress REST API.

**Key methods to implement:**
- `popularMangaRequest` / `popularMangaParse`
- `latestUpdatesRequest` / `latestUpdatesParse`
- `searchMangaRequest` / `searchMangaParse`
- `mangaDetailsRequest` / `mangaDetailsParse`
- `chapterListRequest` / `chapterListParse`
- `pageListRequest` / `pageListParse`

**Common gotchas:**
- Images inside `<noscript>` tags → use raw `response.body.string()` + Regex instead of Jsoup `.select("img")`
- CDN blocking non-browser clients → find proxy URL in page source
- kotlinx.serialization: all JSON fields need default values if they may be absent
- iOS Tachimanga: avoid `Observable.zip`; use blocking `.execute()` calls

---

## Step 5 — Build the APK

```bash
# 1. Copy your extension into the build environment
cp -r /Users/kittiponkanda/Works/Personal/comic/src/{lang}/{extname} \
      /tmp/extensions-source/src/{lang}/

# 2. Build
cd /tmp/extensions-source
./gradlew :src:{lang}:{extname}:assembleRelease

# 3. Find the APK
find /tmp/extensions-source/src/{lang}/{extname}/build/outputs/apk/release -name "*.apk"
# Output: tachiyomi-{lang}.{extname}-v1.4.1-release.apk
```

The version name is auto-generated as `1.4.{extVersionCode}`.

---

## Step 6 — Calculate Source ID

Required for `index.json`. The source ID is deterministic from lang + name:

```python
def java_hash_code(s):
    h = 0
    for c in s:
        h = (31 * h + ord(c)) & 0xFFFFFFFF
    if h >= 0x80000000:
        h -= 0x100000000
    return h

lang = "th"
name = "SiteName"   # must match `override val name` in your Kotlin class
source_id = (1 << 32) | (java_hash_code(lang + name) & 0xFFFFFFFF)
print(source_id)
```

---

## Step 7 — Publish to GitHub Repo

```bash
cd /tmp/mihon-extensions

# 1. Copy APK
cp /tmp/extensions-source/src/{lang}/{extname}/build/outputs/apk/release/*.apk apk/

# 2. Copy icon (xhdpi = 96x96, used in index)
cp /Users/kittiponkanda/Works/Personal/comic/src/{lang}/{extname}/res/mipmap-xhdpi/ic_launcher.png \
   icon/{lang}.{extname}.png

# 3. Add entry to index.json
python3 - << 'EOF'
import json

with open("index.json") as f:
    d = json.load(f)

d.append({
    "name": "SiteName",
    "pkg": "eu.kanade.tachiyomi.extension.{lang}.{extname}",
    "apk": "tachiyomi-{lang}.{extname}-v1.4.1-release.apk",
    "lang": "{lang}",
    "code": 1,
    "version": "1.4.1",
    "nsfw": 0,
    "hasReadme": 0,
    "hasChangelog": 0,
    "sources": [{
        "name": "SiteName",
        "lang": "{lang}",
        "id": "SOURCE_ID_HERE",
        "baseUrl": "https://SITE.com",
        "versionId": 1
    }]
})

with open("index.json", "w") as f:
    json.dump(d, f, indent=2)
with open("index.min.json", "w") as f:
    json.dump(d, f, separators=(",", ":"))
EOF

# 4. Commit and push
git add .
git commit -m "Add SiteName extension v1.4.1"
git push
```

---

## Step 8 — Update Existing Extension

When fixing bugs:

```bash
# 1. Edit KT source at:
#    /Users/kittiponkanda/Works/Personal/comic/src/{lang}/{extname}/.../{ClassName}.kt

# 2. Bump extVersionCode in build.gradle (e.g. 8 → 9)
sed -i '' 's/extVersionCode = 8/extVersionCode = 9/' \
    /Users/kittiponkanda/Works/Personal/comic/src/{lang}/{extname}/build.gradle

# 3. Sync to build env & build
cp .../KT_FILE /tmp/extensions-source/src/{lang}/{extname}/.../
cp .../build.gradle /tmp/extensions-source/src/{lang}/{extname}/
cd /tmp/extensions-source && ./gradlew :src:{lang}:{extname}:assembleRelease

# 4. Deploy
cd /tmp/mihon-extensions
rm apk/tachiyomi-{lang}.{extname}-v1.4.{OLD}-release.apk
cp .../new.apk apk/
# Update index.json: apk filename, code, version
python3 -c "
import json
with open('index.json') as f: d=json.load(f)
entry = next(e for e in d if e['name']=='SiteName')
entry['apk'] = 'tachiyomi-{lang}.{extname}-v1.4.{NEW}-release.apk'
entry['code'] = NEW_CODE
entry['version'] = '1.4.{NEW}'
with open('index.json','w') as f: json.dump(d,f,indent=2)
with open('index.min.json','w') as f: json.dump(d,f,separators=(',',':'))
"
git add . && git commit -m "Fix: ... (vNEW)" && git push
```

---

## Install in Tachimanga (iOS)

1. Open Tachimanga → **More → Settings → Browse → Extension repos**
2. Add:
   ```
   https://raw.githubusercontent.com/DemonX/mihon-extensions/main/index.json
   ```
3. Go to **Browse → Extensions** → install the extension
4. To update: tap **Update** next to the extension name

---

## Repo Structure Reference

```
mihon-extensions/
├── index.json          — full extension index (used by Mihon)
├── index.min.json      — minified copy
├── apk/                — all APK files
│   └── tachiyomi-{lang}.{name}-v{version}-release.apk
├── icon/               — 96x96 PNG icons
│   └── {lang}.{name}.png
├── README.md           — user-facing install instructions
└── GUIDE.md            — this file
```

