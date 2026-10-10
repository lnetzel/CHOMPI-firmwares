# CHOMPI firmwares

The list of firmwares the [CHOMPI setup page](https://ugrossek.github.io/CHOMPI/setup/)
offers. Users pick firmwares for the 15 keys there and write them all to
CHOMPI over USB.

## Getting your firmware on the page

Using an AI coding agent? Tell it: *Follow https://github.com/ugrossek/CHOMPI-firmwares/blob/main/ADD-YOUR-FIRMWARE.md for this firmware.* It prepares the entry, checks it and opens the pull request with you.

By hand: you do this **once**. After that, every release you publish on GitHub shows
up on the page by itself, usually within an hour.

1. Publish your firmware as a GitHub release with the `.bin` attached (or a
   `.zip` with the `.bin` in it). Mark test builds as **pre-release**.
2. Open a pull request that adds `firmwares/<name>.json`:

```json
{
  "title": "SING",
  "card": "SING",
  "author": "ugrossek",
  "summary": "An instrument led by your voice: press keys and sing, CHOMPI plays the keys' notes with your voice.",
  "maintainers": ["ugrossek"],
  "source": {
    "repo": "ugrossek/CHOMPI",
    "tag": "^sing-(?P<v>beta.*)$",
    "asset": "^CHOMPI_SING_.*\\.bin$"
  }
}
```

A check runs on the pull request and lists the releases it found and whether
each one is a CHOMPI firmware. Once the entry is merged, it's on the page.

### The fields

| Field | |
|---|---|
| `title` | Shown on the page, up to 24 characters. |
| `card` | The file name on the SD card: `05_SING.bin`. 1–16 of `A-Z 0-9 _ -`, and unique. Keep it the same across versions, so the page recognises older ones on a card. |
| `author` | Your name as people know it. |
| `summary` | One sentence: what it does. |
| `folder` | Optional: the folder on the card your firmware keeps its files in. The page warns when two firmwares share one. |
| `maintainers` | GitHub logins who may change this entry. |
| `source.repo` | Where your releases are. |
| `source.tag` | Optional: a regular expression the release tag must match. Useful when one repo releases several firmwares. |
| `source.asset` | A regular expression for the file in the release. |
| `source.zip` | If the asset is a zip: a regular expression for the `.bin` inside it. |
| `prefer` | `stable` (default): the page offers your newest release, and pre-releases only while there is no release. `newest`: always the newest, pre-release or not. |
| `skip` | Optional: release tags to leave out, e.g. a broken one. |

The version shown is the `(?P<v>…)` group from `asset` or `tag`, or else the
tag itself.

### What is checked

Every file is downloaded and checked the way the launcher checks it (size,
vector table, entry point), and it must not be a launcher. Its size and
SHA-256 are recorded; the page refuses a download that doesn't match. If you
replace the file in an existing release, the new one is checked and taken.

Nobody tests every release on hardware. The page names the author of each
firmware and links to its release notes.

## How it works

`tools/firmwares.py poll` runs every hour (`.github/workflows/publish.yml`),
records new releases in `releases/` and publishes the site: `catalog.json` and
copies of the `.bin` files, which the browser can't fetch from GitHub
directly. New releases are posted to the issue labelled `release-log`.

To take a release down: add its tag to the entry's `skip`, or revert the
commit that recorded it.

Run it locally:

```bash
python3 tools/firmwares.py check          # validate the entries
python3 tools/firmwares.py poll           # needs GITHUB_TOKEN or `gh auth login`
python3 tools/firmwares.py build --out _site
```

`launcher/` holds the launcher the page offers as an update
(`launcher.json` names its version; the file must carry the tag
`CHOMPI-LAUNCHER <version>`).
