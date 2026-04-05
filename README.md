# 📸 Instagram Handle Finder – Dynamic Sheets

> An **n8n automation workflow** that automatically discovers, validates, and writes verified Instagram handles for competitor brands — directly into your Google Sheets.

![Workflow Overview]

---

## 🚀 What It Does

This workflow takes a list of competitor brand names from a Google Sheet and — without any manual intervention — finds their official Instagram handles, verifies them live, and writes the results (including follower count and bio) back into your sheet. Failed lookups are logged too, with a reason code so you always know what happened.

---

## 🔁 Workflow Steps

| Step | Node | Description |
|------|------|-------------|
| 1 | `⚙️ Config – Sheet IDs` | Set your input & output Google Sheet IDs in one place |
| 2 | `Read Competitors Sheet` | Reads all brand rows from your Google Sheet |
| 3 | `Loop One by One` | Processes each brand sequentially |
| 4 | `Search Instagram Handle` | Searches Google (via Serper API) for the brand's Instagram page |
| 5 | `Extract Instagram Link` | Parses the top search result to pull out the Instagram URL |
| 6 | `Validate Handle Format` | Checks the handle against rules (length, system pages, etc.) |
| 7 | `Handle Valid?` | Branches: valid handles go to live verification, invalid ones go to failure path |
| 8 | `Verify Live Instagram Profile` | Hits the Apify Instagram scraper to confirm the profile actually exists |
| 9 | `Extract Profile Fields` | Pulls `username`, `followersCount`, `biography` from Apify's response |
| 10 | `Update row in sheet` | Writes handle, follower count, status, and timestamp back to the sheet |
| 11 | `Prepare Failure Record` | Builds a structured failure object with a reason code |
| 12 | `Write Failure to Sheet` | Logs the failure reason back to the sheet |
| 13 | `Back to Loop (Fail)` | Returns to the loop to continue with the next brand |

---

## 🛠️ Tech Stack

- **[n8n](https://n8n.io/)** – Workflow automation platform
- **[Google Sheets API](https://developers.google.com/sheets/api)** – Input/output data store
- **[Serper API](https://serper.dev/)** – Google Search API for finding Instagram URLs
- **[Apify Instagram Profile Scraper](https://apify.com/apify/instagram-profile-scraper)** – Live profile verification and data extraction

---

## 📋 Prerequisites

Before importing this workflow, make sure you have:

- [ ] An **n8n** instance (self-hosted or cloud)
- [ ] A **Google Sheets OAuth2** credential configured in n8n
- [ ] A **Serper API key** (free tier available at [serper.dev](https://serper.dev))
- [ ] An **Apify API token** (free tier available at [apify.com](https://apify.com))

---

## ⚙️ Setup & Configuration

### 1. Import the Workflow
Download `Instagram_Handle_Finder_-_Dynamic_Sheets.json` and import it into your n8n instance via **Workflows → Import from file**.

### 2. Configure Sheet IDs
Open the `⚙️ Config – Sheet IDs` node and replace the placeholder values:

```
input_sheet_id   → Your Google Sheet ID containing brand names
input_sheet_tab  → Tab name (default: Sheet1)
output_sheet_id  → Your output Google Sheet ID
output_sheet_tab → Tab name (default: Competitor_Results)
```

> Your Sheet ID is the long string in the URL: `https://docs.google.com/spreadsheets/d/`**`THIS_PART`**`/edit`

### 3. Set Up Credentials
- In `Search Instagram Handle`: Replace the `api_key` query parameter with your **Serper API key**
- In `Verify Live Instagram Profile`: Replace the `token` query parameter with your **Apify API token**
- In all Google Sheets nodes: Connect your **Google Sheets OAuth2** credential

### 4. Prepare Your Input Sheet
Your input Google Sheet should have these columns:

| Column | Description |
|--------|-------------|
| `brand_name` | The brand/company name to search for |
| `competitor_name` | Your internal label for this competitor |
| `brand_category` | Category (e.g., Fashion, Food, Tech) |

### 5. Output Sheet Columns
The workflow writes the following columns to your output sheet:

| Column | Description |
|--------|-------------|
| `brand_name` | Brand name (used as match key) |
| `candidate_handle` | Discovered Instagram handle |
| `status` | `found` or `failed` |
| `Follower` | Follower count at time of discovery |
| `discovered_at` | ISO timestamp of when the handle was found |
| `failure_reason` | Reason code if lookup failed (see below) |

---

## ❌ Failure Reason Codes

| Code | Meaning |
|------|---------|
| `NO_HANDLE_EXTRACTED` | No Instagram URL found in search results |
| `SYSTEM_PAGE_URL` | URL pointed to an Instagram system page (e.g., `/explore`, `/reel`) |
| `INVALID_HANDLE_LENGTH` | Handle was too short or too long |
| `PROFILE_NOT_FOUND` | Apify couldn't find the profile |
| `USERNAME_MISMATCH` | Apify returned a different username than expected |
| `INSUFFICIENT_FOLLOWERS` | Profile had fewer than 100 followers |
| `NO_POSTS` | Profile exists but has zero posts |
| `UNKNOWN_FAILURE` | Catch-all for unexpected errors |

---

## 📌 Notes

- The Serper search query is geo-targeted to **India** (`gl=in`) and searches for the brand's official India page. Update the `q` parameter in `Search Instagram Handle` if you want a different region.
- The workflow processes brands **one at a time** (sequential loop) to avoid rate limiting.
- Apify's sync endpoint has a **120-second timeout** per profile lookup.

---

## 📄 License

MIT License – feel free to use, modify, and share.

---

## 🙋 Author

Built by **Himanshu** — automating competitor research so you don't have to do it manually.

> Got questions or improvements? Open an issue or connect on [LinkedIn](https://www.linkedin.com/in/himanshu-yadav-83b963209/).
