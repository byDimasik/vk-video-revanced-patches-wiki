# Артемий Лебедев (VK Video) — Chapter-Name Pattern Research

Channel: [Артемий Лебедев](https://vkvideo.ru/@temalebedev) (`https://vkvideo.ru/@temalebedev`), owner id `-211437014`.
Purpose: feed a future SponsorBlock-style segment skipper with per-channel chapter-name rules.

---

## 1. Method — where chapters live and how to extract them

- Chapters are **not** a separate player API field: they live in each video's **description** inside a plain-text `«Таймкоды:»` block (`Таймкоды:` followed by `MM:SS Название` lines). The same parsed block also powers the player's on-screen chapter timeline — no hidden metadata involved.
- Extraction path used:
  1. Render the video page (`https://vkvideo.ru/video-211437014_<id>`) to confirm the visible description.
  2. Call `https://api.vkvideo.ru/method/video.get` with the **anonymous token** the web player obtains (`access_token` from the player bootstrap request) to get machine-readable metadata including the full description text.
  3. Parse the `Таймкоды:` block: regex per line `^(\d{1,2}:\d{2}(?::\d{2})?)\s+(.+)$`.
- Consequence for a skipper: the data source is a **description таймкоды block parse**, not an API chapters object — robust and version-independent of the APK.

## 2. Analyzed videos

| Video | id | Date | Duration | Chapters (total) |
|---|---|---|---|---|
| Выпуск №772 | `video-211437014_456248817` | 2026-11-06 | 36:57 | ~85 |
| Выпуск №771 | `video-211437014_456248810` | 2026-11-03 | 42:52 | ~90 |
| Ознобихин (podcast) | `video-211437014_456248789` | 2026-10-30 | 1:18:52 | 18 |

## 3. Recurring ad / self-promo chapter patterns

Timestamps per video (video id shorthand: 772 = `_456248817`, 771 = `_456248810`, 770 = previous episode referenced for stability, podcast = `_456248789`):

| Pattern | Occurrences | Notes |
|---|---|---|
| **«Контент везде»** | 772 @ 02:40 · 771 @ 07:05 · 770 @ 02:28 | The show's ad block. **Most stable marker** across episodes — exact-name match is enough. |
| **«Важнейшая новость»** | 772 @ 00:20 · 771 @ 00:00 | Opening news segment. **NOT an ad** — but positionally distinctive (always first chapter), making it a candidate for a separate `intro` SponsorBlock-style category (skippable "recap/opening" semantics, not sponsorship). |
| **Sponsor @ 00:00** | 772: «Индилайт» @ 00:00 · podcast: «Электромобиль Атом» @ 00:00 | Pre-roll sponsor read; the **name varies per episode** (brand name), only the position is stable. Position+duration-based heuristic (chapter at 00:00, short, before «Важнейшая новость»/«Контент везде»). It is a **visible regular chapter**, not hidden metadata — heuristics must operate on the parsed таймкоды list. |
| **Self-promo design slots** | 772: «Новинки дизайна» @ 21:41, «Инструмент путешественника» @ 22:48 · 770: «Новинки дизайна» @ 25:40, «Инструмент путешественника» @ 28:33 · 771: «Самый хороший дизайн» @ 03:56, «Медиа про дизайн» @ 18:31, «Новинка магазинуса» @ 19:28 | Fixed recurring self-promo segment names; exact-name matching works, but the set is a small per-channel dictionary rather than a single title. |
| **«Нативная интеграция: …» prefix** | 771 @ 22:22 («Нативная интеграция: Levenhuk») | **Paid integration**; name is a prefix + brand. Prefix matcher required (or a `Нативная интеграция:` family rule). |

## 4. Unique-chapter contrast

- **Episodes №771/№772** (news show): ~70–85 **unique content chapters** per episode — nearly one chapter per news headline. Volume is high, but recurring ad/self-promo names are few and stable.
- **Podcast (Ознобихин)**: 18 chapters total, **17 unique content chapters** — a long-form conversation with only the pre-roll sponsor pattern in common with the news show.
- Practical takeaway: content chapters are noise; only the recurring vocabulary (ad block, sponsor reads, self-promo slots, «Нативная интеграция:») needs rules.

## 5. Design implications for a future SponsorBlock-style skipper

1. **Per-channel rules**, keyed by channel/owner id (`-211437014`), not global — patterns here are show-specific vocabulary.
2. **Matcher types** needed:
   - *exact name* — «Контент везде», «Новинки дизайна», «Инструмент путешественника», «Самый хороший дизайн», «Медиа про дизайн», «Новинка магазинуса»;
   - *name prefix* — «Нативная интеграция: …» (brand suffix varies);
   - *position + duration pattern* — pre-roll sponsor at 00:00 (brand name varies per episode).
3. **Categories**:
   - `sponsor` — «Контент везде», «Нативная интеграция: …», pre-roll 00:00 sponsor reads;
   - `promo` / `self-promo` — «Новинки дизайна», «Инструмент путешественника», «Самый хороший дизайн», «Медиа про дизайн», «Новинка магазинуса» (may be separate from paid sponsor, user-toggleable);
   - `intro` — «Важнейшая новость» opening segment (candidate; NOT an ad).
4. **Data source**: parse the `Таймкоды:` block from the video description (rendered page or `api.vkvideo.ru/method/video.get` with the player's anonymous token). Segments derive directly from chapter start times (and next-chapter start as the implicit end) — no extra API surface required.
