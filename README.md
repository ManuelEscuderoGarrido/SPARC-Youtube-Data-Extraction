# YouTube Data Extraction Workshop

[English](README.md) | [Português (Portugal)](README.pt-PT.md)

This repository contains a Google Colab notebook for collecting video metadata, top-level comments, and user identifiers from a YouTube channel through the YouTube Data API v3. It is intended for teaching and small-scale research exercises. The default limits keep the extraction short enough for a workshop.

## What the notebook does

The notebook:

1. identifies the uploads playlist associated with a channel;
2. retrieves the channel's uploaded video identifiers;
3. selects videos by publication date and minimum duration;
4. collects top-level comments published within the selected period;
5. creates consolidated and per-video CSV files;
6. records extraction outcomes, including unavailable videos and disabled comments; and
7. packages the results in a ZIP archive.

Replies to top-level comments are not collected.

## Requirements

- A Google account
- A Google Cloud project with the YouTube Data API v3 enabled
- A YouTube Data API key
- Google Colab, recommended for the workshop

The notebook installs `google-api-python-client` and `isodate`. `pandas` is already available in Google Colab.

## Obtain an API key

1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a project or select an existing one.
3. Go to **APIs & Services → Library**.
4. Enable **YouTube Data API v3**.
5. Go to **APIs & Services → Credentials**.
6. Select **Create credentials → API key**.

The API key is not stored in the notebook. When the relevant cell runs, Colab requests it through a hidden input field. Never commit an API key to a public repository. For use beyond the workshop, restrict the key to the YouTube Data API v3 and review the available credential restrictions.

## Run the notebook

1. Download or clone this repository.
2. Open `youtube_data_extraction_workshop.ipynb` in Google Colab.
3. Edit the configuration cell.
4. Run the cells in order.
5. Enter the API key when prompted.
6. Review the extraction log and download the ZIP archive.

## Configuration

Only the configuration cell normally needs to be changed.

| Variable | Purpose | Workshop default |
| --- | --- | --- |
| `CHANNEL_ID` | YouTube channel ID beginning with `UC` | Alan Barroso channel ID |
| `CHANNEL_NAME` | Name used for the output folder | `Alan_Barroso` |
| `START_DATE` | First date included, interpreted in UTC | `2025-01-01` |
| `END_DATE` | Last date included, interpreted in UTC | `2025-12-31` |
| `MIN_VIDEO_DURATION_MINUTES` | Minimum video duration | `8` |
| `MAX_VIDEOS` | Maximum number of matching videos | `5` |
| `MAX_COMMENTS_PER_VIDEO` | Maximum top-level comments per video | `100` |
| `SAVE_TO_DRIVE` | Save results in Google Drive | `True` |
| `OUTPUT_ROOT` | Root output folder | `youtube_workshop_data` |

Set `MAX_VIDEOS` or `MAX_COMMENTS_PER_VIDEO` to `None` to remove the corresponding limit. Doing so may substantially increase runtime, storage use, and API-quota consumption.

## Output structure

When Google Drive storage is enabled, results are created under:

```text
MyDrive/
└── youtube_workshop_data/
    └── CHANNEL_NAME/
        └── START_DATE_END_DATE/
            ├── videos.csv
            ├── comments.csv
            ├── users.csv
            ├── extraction_log.csv
            ├── comments_by_video/
            │   └── VIDEO_ID_comments.csv
            └── users_by_video/
                └── VIDEO_ID_users.csv
```

The notebook also creates a ZIP archive of the period folder.

### Main files

- `videos.csv`: video ID, title, publication time, duration, view count, and the comment count reported by YouTube.
- `comments.csv`: one row per top-level comment, including the video, author information, text, dates, likes, and reply count.
- `users.csv`: distinct pairs of author channel ID and displayed author name found in the extracted comments.
- `extraction_log.csv`: one row per selected video with its extraction status and number of comments collected.

The displayed author name is not a stable identifier. When YouTube provides it, the notebook retains `author_channel_id`; this value may still be absent for some comments.

## Extraction behaviour

- Video details are requested in batches of up to 50 IDs.
- Comment pages contain up to 100 threads and are followed through `nextPageToken`.
- Comments are ordered by time so that extraction can stop after reaching the beginning of the selected period.
- Temporary server errors are retried with exponential backoff.
- Quota exhaustion stops the extraction explicitly.
- Disabled comments, unavailable videos, missing videos, empty periods, and classroom limits are recorded in the extraction log.

## Scope and limitations

API access does not make YouTube fully observable. Deleted, private, restricted, or otherwise unavailable content may be absent. Counts and availability can change after collection, so the extraction date and configuration should be documented with any analysis.

The sampling period applies both to video publication dates and to comment publication dates. The researcher remains responsible for defining the sample, duration threshold, exclusion rules, and unit of analysis.

## Responsible use

Public availability does not remove the need for data minimisation, secure storage, ethical review, and careful handling of user identifiers. Before collecting or sharing data, consider the applicable institutional requirements, legal framework, platform terms, and research purpose.

## Troubleshooting

| Message or status | Likely cause | Suggested action |
| --- | --- | --- |
| `No API key was provided` | The prompt was left empty | Run the API-key cell again and enter a valid key |
| `The channel was not found` | The channel ID is incorrect | Use the channel ID beginning with `UC`, not a handle or channel URL |
| `No videos matched...` | No video meets the date and duration filters | Review the dates and `MIN_VIDEO_DURATION_MINUTES` |
| `comments_disabled` | Comments are disabled for the video | No comment extraction is possible through this method |
| `quotaExceeded` or `dailyLimitExceeded` | The project's daily quota has been exhausted | Wait for the quota to reset or review the project's quota settings |

