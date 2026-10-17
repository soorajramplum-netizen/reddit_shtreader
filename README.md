# Reddit Downloader

Download the full history of posts and comments from a list of subreddits into simple CSV files, grouped by cohort. Built for consumer research: finding what people actually worry about, in their own words.

It's free. It uses the public [Arctic Shift](https://arctic-shift.photon-reddit.com) Reddit archive, so you don't need a Reddit API key or account, and it needs no extra Python packages.

## What it does

- Reads a list of subreddits from `subreddits.csv`, each tagged with a cohort (beauty, food, women, city…) and a priority.
- Downloads every post, and optionally every comment, from a start date up to today, oldest first.
- Runs several subreddits at the same time (3 by default).
- Saves its place after every batch, so you can stop it any time and resume later without losing data.
- Re-running it later adds only what's new. Choosing an earlier start date fills in the older history.

## Quick start (Windows)

1. Install Python 3.9 or newer from [python.org](https://www.python.org/downloads/) and tick **"Add python.exe to PATH"**.
2. Double-click **`start.bat`**.
3. Enter a priority (1 to 4) and a start date (e.g. `2023-01-01`), or press Enter to accept the defaults.

To stop, double-click **`stop.bat`**. The downloader finishes the batch it's on, saves its place and exits. Run `start.bat` again to carry on.

## Command line (any OS)

```bash
python reddit_download.py                                   # priority 1 subreddits, since 2023-01-01
python reddit_download.py --priority 2                      # priorities 1 and 2
python reddit_download.py --subs IndianHaircare TwoXIndia   # only these subreddits
python reddit_download.py --since 2021-01-01 --no-comments  # posts only, from 2021
python reddit_download.py --workers 5                       # 5 subreddits at a time
```

| Option | Default | What it does |
| --- | --- | --- |
| `--priority N` | `1` | Downloads every subreddit whose priority is N or lower |
| `--subs A B …` | | Downloads only the named subreddits (overrides `--priority`) |
| `--since YYYY-MM-DD` | `2023-01-01` | Earliest date to download |
| `--no-comments` | off | Posts only, even for subreddits marked for comments |
| `--workers N` | `3` | How many subreddits to download at once |

To stop from the command line, create an empty file named `STOP` in the folder. `start.bat` deletes it automatically before the next run; from the command line, delete it yourself before resuming.

## The subreddit list: `subreddits.csv`

| Column | Meaning |
| --- | --- |
| `subreddit` | Name without `r/` |
| `lens` | `category` (a product area), `cohort` (a group of people) or `place` (a city or state) |
| `cohort`, `sub_cohort` | Your own grouping, e.g. `beauty` / `haircare`; written onto every row |
| `priority` | 1 = most important. `--priority 2` downloads everything with priority 1 or 2 |
| `comments` | `yes` to download comments too (comments are about 10× the volume of posts) |
| `activity_90d` | Notes on how active the subreddit is; not used by the script |

Add, remove or re-prioritise rows freely. The list that ships with the repo covers about 90 Indian subreddits across beauty, fashion, food, health, life stages, shopping, work and cities.

## Output

```
data/
  <subreddit>/posts.csv
  <subreddit>/comments.csv
  state.json        progress for each subreddit (what's covered, from and to)
```

**posts.csv:** `subreddit, cohort, sub_cohort, date, url, id, created_utc, author, title, selftext, score, num_comments, link_flair_text, over_18`

**comments.csv:** `subreddit, cohort, sub_cohort, date, url, id, created_utc, author, body, score, link_id, parent_id`

- `link_id` is the post a comment belongs to (`t3_<post id>`).
- `parent_id` is what it replies to: the post (`t3_…`) or another comment (`t1_…`).
- Deleted and removed items, and AutoModerator, are skipped.
- Files are UTF-8 and open in Excel or pandas.
- If a run is cut off at exactly the wrong moment, a batch can be saved twice. Remove duplicates by `id` when you analyse the data.

## How big it gets

A large subreddit can have tens of thousands of posts a year and roughly 10 times as many comments. Expect:
- about 2,000 posts a minute for each subreddit being downloaded,
- several hours for a large subreddit's comments,
- hundreds of MB to a few GB of CSV for a large list.

Plan big runs overnight.

## Notes

- **Be polite.** Arctic Shift is a free community service. Each worker pauses between requests and backs off when the server is busy. Running more than 3-5 workers tends to cause errors rather than speed.
- **"422" or timeout messages** mean the archive is busy. The script retries automatically. If a subreddit keeps failing, it's skipped, and running again picks it up.
- **Very recent posts** may show low scores and comment counts, because the archive captures them shortly after posting.
- **Reddit's audience is not the whole population.** Indian Reddit skews urban, young, male and English-speaking. Treat it as one voice among several.
- Reddit content belongs to its authors. Use it for research, and quote people anonymously.

## Files

```
reddit_download.py   the downloader
subreddits.csv       the subreddit list with cohorts and priorities
start.bat            Windows: double-click to run
stop.bat             Windows: double-click to stop cleanly
```
