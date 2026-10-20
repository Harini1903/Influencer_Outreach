# Automated Micro-Influencer Outreach System

EDXSO AI Engineer Intern, Assignment 1.
Niche: **Fashion & Beauty**. Platform: **YouTube**.

A pipeline that discovers micro-influencers, filters and classifies them, enriches their profiles, writes personalized outreach with an LLM, and simulates sending with duplicate protection and a log.

```
Discovery -> Collection -> Filtering -> Enrichment -> AI Personalization -> Review -> Sending -> Tracking
```

## Important disclosures

- **Glowly is a fictional sample brand** created for this assignment. The messages say "I represent Glowly", so they must never be sent to real creators.
- **No real outreach was sent.** The sending layer ran in `DRY_RUN` mode: emails are logged as `SIMULATED`. If dry-run is switched off, the code redirects every email to a test inbox you control, never to the creator.
- **No data is fabricated.** Missing values are marked `Not Found` (email, Instagram, website) or `Not Available` (audience age, gender, geography). Nothing is guessed.
- **Privacy:** emails in the dataset were published by creators in their public channel descriptions. Keep this repository private, or remove the `email` column before making it public.

## Results of the test run

| Stage | Result |
|---|---|
| Channels discovered | 400 |
| In the 5k-100k follower range | 52 |
| Passed automated filters (shortlist) | 27 |
| Emails found in shortlist | 10 of 27 |
| Instagram links found | 2 of 27 |
| Websites found | 3 of 27 |
| Messages generated and validated | 27 of 27 |
| Rejected in review | 3 |
| Held for manual check | 18 |
| Approved | 6 |
| Emails simulated | 5 |
| Instagram DMs queued for manual sending | 6 (2 with a handle found) |

## Technology stack

- Python 3, run in Google Colab (notebook) or locally
- **YouTube Data API v3** (`google-api-python-client`) for discovery and metrics
- **Gemini API** (`google-genai`) for message generation
- `pandas` for data processing, **SQLite** for the outreach log, `smtplib` for the optional email path

## Data source and discovery methodology

1. Eight search queries (`skincare routine`, `makeup tutorial`, `fashion haul`, `outfit ideas`, `get ready with me`, `beauty product review`, `styling tips`, `hair care routine`) are run through YouTube `search.list` with `type=channel`, 50 results each.
2. Channel IDs are de-duplicated, giving 400 unique channels.
3. `channels.list` returns subscribers, country, description and the uploads playlist for each channel, in batches of 50.

Known weakness: searching with `type=channel` matches channel *names*, so many results are tiny or empty channels named after the query. The filters remove them, but a video-based search would give a better candidate pool.

## Engagement rate

For each channel in the follower range, the last 10 uploads are fetched:

```
engagement rate = (sum of likes + sum of comments) / sum of views x 100
```

Videos with hidden likes or zero views are skipped. If fewer than 5 videos are usable, engagement is treated as unavailable and the channel fails, rather than being estimated.

## Filtering and classification logic

Every one of the 400 channels gets `PASS` or `FAIL` and a stated reason.

| Rule | Threshold |
|---|---|
| Followers | 5,000 to 100,000 |
| Engagement rate | at least 1.0% (above 12% is flagged for manual verification, not rejected) |
| Average views | at least 500 |
| Videos used for engagement | at least 5 |
| Activity | last upload within 90 days |
| Niche | keyword match on channel name, description and recent titles (Beauty or Fashion) |
| Brand accounts | rejected if name or description matches brand patterns (`official`, `pvt`, `ltd`, etc.) |

All thresholds are named variables at the top of the filtering cell.

## Profile enrichment

From the public channel description only:

- **Email**: regex match. If none, `Not Found`.
- **Instagram handle** and **website**: URL parsing. If none, `Not Found`.
- **Content themes**: keyword matching on channel name, description and recent video titles.
- **Audience age, gender, geography**: `Not Available`. The public API does not expose them.

## AI model and personalization

- **Model:** `gemini-flash-latest`, with automatic fallback to `gemini-3.1-flash-lite` and `gemini-2.5-flash-lite` when a model is rate-limited. The model that wrote each message is stored in the `model_used` column.
- **Collaboration angle (rule-based, not chosen by the LLM):** 60,000+ followers: paid sponsorship. 20,000+ followers or 4%+ engagement: affiliate campaign. Otherwise: product barter with UGC content.
- **Prompt:** the model receives only the facts collected (channel name, niche, three recent video titles, collaboration angle) and is told to use nothing else. It returns JSON with an email subject, a 60-90 word email, and a 15-30 word Instagram DM.
- **Rules in the prompt:** greet with "Hi there," (real first names are never guessed), never claim to have followed, watched regularly or enjoyed the creator's content, no unsupported flattery, reference one specific recent video.
- **Validation in code:** word counts are checked, and the call is retried with a correction if they are out of range. Statuses are `OK`, `FAILED_VALIDATION`, `API_UNAVAILABLE`, `NOT_RUN`. A separate check scans for banned phrases and the greeting rule.

## Review stage

Automatic flags mark channels that look like a seller (several shop-style words in titles), have mostly non-Latin-script titles, show no fashion/beauty content in recent titles, or have no contact route. Sellers are rejected; the rest are `NEEDS_MANUAL_CHECK`. Messages were then read by a human against each channel's recent titles, and only approved channels move to sending. Nothing is approved automatically.

## Sending mechanism

- Only `APPROVED` records are processed.
- Only addresses that pass an email-format check are used (`Not Found` never does).
- **Duplicate prevention:** a check before every send, plus a `UNIQUE` constraint on email in the SQLite `outreach_log` table, so the database itself refuses a second row for the same address.
- Each attempt is logged with influencer, email, sent flag, timestamp, status and detail.
- **Instagram DMs are never sent automatically.** They are written to `dm_queue` as `MANUAL_SEND_REQUIRED` (handle found) or `NO_INSTAGRAM_HANDLE_FOUND`, to be sent by a person.
- The SMTP path (Gmail app password) exists in the code but was not exercised in the test run.

## Output files

| File | Contents |
|---|---|
| `influencers_all_400.csv` | all discovered channels with pass/fail and reason |
| `influencers_shortlisted.csv` | the 27 shortlisted, enriched profiles |
| `influencers_with_messages.csv` | shortlist with generated emails, DMs and review status |
| `outreach_tracker.csv` | influencer, email, message generated, sent, date, status |
| `instagram_dm_queue.csv` | DMs for manual sending |

## Limitations

- Low email yield (10 of 27): the API cannot read YouTube's hidden "business email" button, only addresses written in descriptions.
- No audience demographics; engagement is computed from the last 10 videos only.
- Niche, content-theme and seller detection are keyword heuristics. They miss cases (for example, a channel named "Style Tips" that posts religious vlogs passed the automated filter and was caught only in manual review) and can produce false positives.
- Channel-name search produces a noisy candidate pool.
- Some shortlisted channels have non-English content or look like small businesses; language and audience fit are not checked automatically.
- The sample brand sells skincare, so fit is loose for pure fashion creators.
- The free LLM tier has daily limits; generation is resumable and skips completed records, but a full run can span more than one day.
- Instagram data collection and DMs are out of scope because no compliant automated route was used.

## Setup

1. Create a Google Cloud project, enable **YouTube Data API v3**, and create an API key.
2. Create a Gemini API key in Google AI Studio.
3. **Colab:** add the keys as Secrets named `YOUTUBE_API_KEY` and `GEMINI_API_KEY` with notebook access on, then run the notebook top to bottom.
   **Local:** copy `.env.example` to `.env`, fill in the keys, run `pip install -r requirements.txt`.
4. Keep `DRY_RUN = True` unless you are sending to your own test inbox (set `TEST_RECIPIENT`, and add `GMAIL_ADDRESS` and `GMAIL_APP_PASSWORD`).

## Extending from 50 to 500+ influencers

- Add video-based and hashtag-style queries; paginate with `nextPageToken`.
- Cache raw API responses so reruns do not spend quota.
- Batch calls (already 50 channels per request) and track quota per day.
- Move the LLM step to a queue with the same resume-and-fallback logic.
- Add new data sources (for example a CSV importer) behind the same record format.
