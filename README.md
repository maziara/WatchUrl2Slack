# WatchUrl2Slack

Monitor a web page for a specific string in its links and get notified on Slack when it appears or disappears.

## What It Does

This tool scrapes a target URL, searches all hyperlinks (`<a>` tags) for a case-insensitive substring match in their `href` attribute, and sends a Slack message when:

- **String found**: The first time the string is detected, it sends a notification with all matching links.
- **String removed**: If the string was previously found but is no longer present, it sends a removal notice and resets — so it will notify again on the next detection.

A local flag file (`notifiedFlag`) prevents duplicate notifications between runs.

## How It Works

```
[Run script] → [Fetch URL] → [Parse all <a> links] → [Search hrefs for string]
                                                            │
                                              ┌─────────────┴─────────────┐
                                          Found                       Not Found
                                              │                           │
                                     Flag exists?                  Flag exists?
                                      No → Notify                  Yes → Notify
                                           + set flag                   + remove flag
```

## Prerequisites

- Python 3.10+
- A [Slack Bot Token](https://api.slack.com/tutorials/tracks/getting-a-token) with `chat:write` scope
- A Slack channel the bot has been invited to
- Docker & Docker Compose (optional, for containerized runs)

## Setup

### 1. Clone the repository

```bash
git clone git@github.com:maziara/WatchUrl2Slack.git
cd WatchUrl2Slack
```

### 2. Configure environment variables

Copy and edit the `.env` file:

```bash
cp .env .env.local  # or edit .env directly
```

| Variable | Description | Example |
|---|---|---|
| `URL_TO_WATCH` | The web page URL to monitor | `https://example.com/events` |
| `STRING_TO_LOOK_FOR` | Substring to search for in link hrefs (case-insensitive) | `registration` |
| `SLACK_BOT_TOKEN` | Slack Bot OAuth token (`xoxb-...`) | `xoxb-1234-5678-abcdef` |
| `SLACK_CHANNEL` | Slack channel name or ID to post to | `#alerts` |

### 3. Install dependencies

**With Pipenv:**

```bash
pip install pipenv
pipenv install
```

**With pip:**

```bash
pip install beautifulsoup4 slack-sdk python-dotenv
```

## Usage

### Run directly

```bash
pipenv run python Look4StringInUrl.py
```

### Run with Docker Compose

```bash
docker-compose up --build
```

This builds the container, mounts the project directory, and runs the script once.

### Scheduled monitoring

The script runs once per execution. To monitor continuously, schedule it with cron:

```bash
# Check every 15 minutes
*/15 * * * * cd /path/to/WatchUrl2Slack && pipenv run python Look4StringInUrl.py
```

Or with Docker:

```bash
*/15 * * * * cd /path/to/WatchUrl2Slack && docker-compose up --build
```

## Project Structure

```
WatchUrl2Slack/
├── Look4StringInUrl.py   # Main script — fetches URL, parses links, sends Slack messages
├── .env                  # Environment variables (not committed in production)
├── Pipfile               # Python dependencies (Pipenv)
├── Pipfile.lock          # Locked dependency versions
├── dockerfile            # Docker image definition (Python 3.10-slim + Pipenv)
├── docker-compose.yml    # Docker Compose service definition
└── .gitignore            # Git ignore rules
```

## Code Overview

`Look4StringInUrl.py` contains:

| Function | Purpose |
|---|---|
| `sendSlackMessage(msg)` | Posts a message to the configured Slack channel using the Slack SDK |
| `writeFlag()` | Creates the `notifiedFlag` file to mark that a notification was sent |
| `removeFlag()` | Deletes the `notifiedFlag` file to reset notification state |
| `flagged()` | Returns `True` if the flag file exists |

**Main flow** (module-level):
1. Loads environment variables from `.env`
2. Fetches the target URL with `urllib.request`
3. Parses HTML with BeautifulSoup
4. Collects all `<a>` tags whose `href` contains the search string (case-insensitive)
5. If matches found and not previously flagged → sends Slack messages and sets flag
6. If no matches found and previously flagged → sends removal notice and clears flag

## Dependencies

| Package | Version | Purpose |
|---|---|---|
| `beautifulsoup4` | 4.12.3 | HTML parsing |
| `slack-sdk` | 3.33.1 | Slack API client |
| `python-dotenv` | 1.0.1 | Load `.env` file into environment |

## Troubleshooting

**No notifications received**
- Verify `SLACK_BOT_TOKEN` and `SLACK_CHANNEL` are set correctly in `.env`
- Ensure the Slack bot has been invited to the target channel
- Check that the bot token has `chat:write` scope

**Duplicate notifications**
- The `notifiedFlag` file tracks notification state. If deleted manually, the next run will re-notify.

**String not detected even though it's on the page**
- The script only searches `href` attributes of `<a>` tags, not page text content.
- The match is case-insensitive on the href value.

**Docker issues**
- The container mounts the current directory as `/home`, so the `.env` and `notifiedFlag` files are shared between host and container.

## License

No license specified.
