
# jesscrape

[![Version](https://img.shields.io/badge/version-0.2.0-lightgrey.svg)](#)
[![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933.svg)](https://nodejs.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**jesscrape** is a modular web-scraping and data-extraction library for Node.js. It provides a unified interface for retrieving media, profile information, search results, translations, and lyrics from supported third-party providers.

> **Version:** `0.2.0`  
> **Status:** Early development  
> **Node.js:** `18+`

---

## Requirements

- Node.js 18 or newer
- npm
- Internet connection

### Dependencies

| Package | Purpose |
|---|---|
| axios | HTTP requests |
| cheerio | HTML parsing and data extraction |

## Install

```bash
npm install jesscrape
```

For local development:

```bash
npm install
```

## Basic Usage

```js
const {
  tiktok,
  instagram,
  spotify,
  youtube,
  tools,
  config,
  errors
} = require("jesscrape");

async function main() {
  const result = await tiktok.vt.no_wm(
    "https://www.tiktok.com/@username/video/123456789"
  );

  console.log(result.meta.downloadUrl);
}

main().catch(console.error);
```

All API methods are asynchronous and support `async/await`.

## Platform APIs

### TikTok

| Method | Description | Result |
|---|---|---|
| `tiktok.vt.no_wm(url)` | No-watermark video extraction | `meta.downloadUrl` |
| `tiktok.photo(url)` | Photo post extraction | `meta.images` |
| `tiktok.metadata(url)` | Post metadata extraction | `meta.clean` |
| `tiktok.profile(username)` | Public profile information | `meta.clean` |

**Video Downloader**

```js
const result = await tiktok.vt.no_wm(
  "https://www.tiktok.com/@username/video/123456789"
);

console.log(result.meta.downloadUrl);
```

**Photo Downloader**

```js
const result = await tiktok.photo(
  "https://www.tiktok.com/@username/photo/123456789"
);

console.log(result.meta.images);
```

**Metadata**

```js
const result = await tiktok.metadata(
  "https://www.tiktok.com/@username/video/123456789"
);

console.log(result.meta.clean);
```

**Profile Stalker**

```js
const result = await tiktok.profile("username");

console.log(result.meta.clean);
```

Profile information may include public statistics, biography, and avatar data, depending on provider availability.

### Instagram

| Method | Description | Result |
|---|---|---|
| `instagram.reel(url)` | Instagram Reel extraction | `meta.downloadUrl` |

```js
const result = await instagram.reel(
  "https://www.instagram.com/reel/EXAMPLE/"
);

console.log(result.meta.downloadUrl);
```

### Spotify

| Method | Description | Result |
|---|---|---|
| `spotify.search(query)` | Search tracks | Search results |
| `spotify.download(url)` | Audio download URL | `meta.downloadUrl` |
| `spotify.lyrics(title, artist)` | Lyrics lookup | `meta.plain`, `meta.synced` |

**Search**

```js
const result = await spotify.search("song title");

console.log(result.meta);
```

**Download**

```js
const result = await spotify.download(
  "https://open.spotify.com/track/EXAMPLE"
);

console.log(result.meta.downloadUrl);
```

**Lyrics**

```js
const result = await spotify.lyrics(
  "Song Title",
  "Artist Name"
);

console.log(result.meta.plain);
console.log(result.meta.synced);
```

### YouTube

| Method | Description |
|---|---|
| `youtube.search(query)` | Search YouTube content |
| `youtube.download(url, format)` | Extract media using the selected format |

Supported formats:

- `mp3`
- `mp4`
- `360`
- `720`
- `1080`

**Search**

```js
const result = await youtube.search("search query");

console.log(result.meta);
```

**Download**

```js
const result = await youtube.download(
  "https://www.youtube.com/watch?v=VIDEO_ID",
  "720"
);

console.log(result.meta);
```

Actual format availability depends on the video and provider.

### Tools

| Method | Description |
|---|---|
| `tools.translate(text, source, target)` | Text translation |
| `tools.telegramProfile(username)` | Public Telegram profile lookup |

**Translate**

```js
const result = await tools.translate(
  "Hello, world!",
  "en",
  "ms"
);

console.log(result.meta);
```

**Telegram Profile**

```js
const result = await tools.telegramProfile("username");

console.log(result.meta);
```

Telegram lookup is limited to publicly accessible profile information. It does not provide access to private accounts or messages.

## Response Envelope

All successful methods follow a consistent response structure:

```js
{
  ok: true,
  platform: "tiktok",
  sourceUrl: "https://...",
  fetchedAt: "2026-11-13T10:00:00.000Z",
  meta: {},
  raw: {}
}
```

### Response Fields

| Field | Type | Description |
|---|---|---|
| `ok` | Boolean | Indicates successful execution |
| `platform` | String | Platform name |
| `sourceUrl` | String | Original source URL, where applicable |
| `fetchedAt` | String | ISO-8601 timestamp |
| `meta` | Object | Normalized result data |
| `raw` | Object | Original provider response |

### Meta vs Raw

- `meta` contains convenient method-specific data.
- `raw` contains the original provider response.
- Both structures may vary depending on the platform and method.

## Configuration

jesscrape provides configuration management through the `config` object.

**Set Configuration**

```js
const { config } = require("jesscrape");

config.set({
  timeout: 20000
});
```

**Get Configuration**

```js
const currentConfig = config.get();

console.log(currentConfig);
```

**Reset Configuration**

```js
config.reset();
```

`config.reset()` restores the default configuration.

## Error Handling

jesscrape provides the following custom error classes:

| Error | Description |
|---|---|
| `ValidationError` | Invalid input |
| `NetworkError` | Network request failure |
| `TimeoutError` | Request timeout |
| `ProviderError` | Third-party provider failure |
| `NotFoundError` | Requested resource not found |
| `UnsupportedPlatformError` | Unsupported platform or operation |

### Try-Catch Example

```js
const { tiktok, errors } = require("jesscrape");

async function getVideo(url) {
  try {
    const result = await tiktok.vt.no_wm(url);

    console.log(result.meta.downloadUrl);

    return result;
  } catch (error) {
    if (error instanceof errors.ValidationError) {
      console.error("Invalid input:", error.message);
    } else if (error instanceof errors.NetworkError) {
      console.error("Network error:", error.message);
    } else if (error instanceof errors.TimeoutError) {
      console.error("Request timed out:", error.message);
    } else if (error instanceof errors.ProviderError) {
      console.error("Provider error:", error.message);
    } else if (error instanceof errors.NotFoundError) {
      console.error("Resource not found:", error.message);
    } else if (error instanceof errors.UnsupportedPlatformError) {
      console.error("Unsupported operation:", error.message);
    } else {
      console.error("Unexpected error:", error);
    }

    throw error;
  }
}

getVideo("https://www.tiktok.com/@username/video/123456789")
  .catch(() => {});
```

## Project Structure

```text
jesscrape/
├── index.js
├── package.json
├── README.md
├── LICENSE
├── src/
│   ├── index.js
│   ├── core/
│   │   ├── config.js
│   │   ├── errors.js
│   │   ├── request.js
│   │   ├── response.js
│   │   └── validator.js
│   └── platforms/
│       ├── tiktok/
│       ├── instagram/
│       ├── spotify/
│       ├── youtube/
│       ├── telegram/
│       └── tools/
├── examples/
└── test/
```

### Core Modules

| File | Responsibility |
|---|---|
| `config.js` | Configuration management |
| `errors.js` | Custom error classes |
| `request.js` | HTTP request handling |
| `response.js` | Response envelope generation |
| `validator.js` | Input validation |

Each platform directory contains its respective integration and implementation.

## Notes

- jesscrape `0.2.0` is an early-development release.
- API methods, response structures, and internal behavior may change in future versions.
- This library depends on third-party services and endpoints, including **TikWM, IgExport, SpotSaver, Siputzx, Savenow**, and others.
- Third-party endpoints, response formats, availability, rate limits, and usage policies may change at any time without notice.
- Provider outages or endpoint changes may cause certain methods to stop working.
- Download URLs may be temporary and can expire.
- Search results, profile information, and media availability depend on the respective providers.
- Respect platform terms of service, copyright, privacy rights, and applicable laws.
- Only retrieve or process content you are authorized to access and use.
- Avoid excessive requests to third-party services.

## Credits

This project would not be possible without the support, ideas, and resources from the following:

- **Allah** — For the strength, guidance, and opportunity to build this.
- **My Family** — For the endless support and patience.
- **JesterNovaZ** (Me) — The creator and maintainer of Jesscrape.
- **Array** — For the recommendations, suggestions, and architectural advice.
- **Vinz** — For providing and reverse-engineering the API endpoints used in this library.

## License

This project is licensed under the **MIT License**.

You are permitted to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the software, subject to the conditions of the MIT License.

See the [`LICENSE`](LICENSE) file for the complete license text.
