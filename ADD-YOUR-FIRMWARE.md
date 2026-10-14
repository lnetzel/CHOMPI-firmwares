# Adding a firmware to the CHOMPI setup page (instructions for an AI agent)

You are helping the author of a CHOMPI firmware get it listed on the setup
page at https://ugrossek.github.io/CHOMPI/setup/. The result is **one pull
request** to https://github.com/ugrossek/CHOMPI-firmwares that adds one file,
`firmwares/<id>.json`. After it is merged, every GitHub release the author
publishes shows up on the page by itself. The author does this once.

Work in the author's firmware repository. Ask the author whenever something
below says so, and before anything that publishes (a release, a fork, a pull
request). Don't change their code, releases or tags unless they ask you to.

## 1. Look at what is there

- `gh auth status` must work; if not, ask the author to run `gh auth login`.
- Find the GitHub repo (`gh repo view --json nameWithOwner`) and its releases:
  `gh release list` and `gh release view <tag> --json assets,isPrerelease`.
- The page needs a release with the firmware `.bin` attached, or a `.zip`
  with the `.bin` inside. If there is none, tell the author and offer to make
  one from their build output (`gh release create`); test builds should be
  marked pre-release. Don't publish it without their OK.
- One repository may release several firmwares. Then each one gets its own
  entry, told apart by tag or asset name.

## 2. Write the entry

Fill in `firmwares/<id>.json`. `<id>` is lower case, digits and `-`, e.g.
`sing`.

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

- `title`: the firmware's name as users know it, up to 24 characters.
- `card`: the file name on the SD card (`05_SING.bin`), 1-16 of
  `A-Z 0-9 _ -`. Usually the title in capitals. It must not change later:
  the page recognises a firmware on a card by it.
- `author`: the name the author goes by in the CHOMPI community. Ask.
- `summary`: one sentence, 10-200 characters, saying what it does for a
  musician, not how it's built. Draft it from the README and let the author
  confirm it.
- `folder`: the folder on the SD card the firmware reads or writes, if any.
  Find it in the source (look for paths passed to `f_open`, `f_opendir`,
  `f_mkdir` and similar). Leave the field out if it uses none.
- `color`: optional, `#RRGGBB`, the colour of the firmware's key. Leave it
  out unless the author asks for one; the list's maintainer picks colours
  that are easy to tell apart on the LEDs.
- `maintainers`: the author's GitHub login, plus anyone else who may change
  the entry.
- `source.repo`: `owner/name`.
- `source.asset`: a regular expression matching the release file. It must
  match the file in **future** releases too, so allow for a version in the
  name if the author puts one there (`^chompi-x0x-.*\\.bin$`).
- `source.tag`: optional regular expression for the release tag. Needed if
  the repo releases other things too.
- The version shown on the page is the `(?P<v>...)` group from `asset` or
  `tag`, or else the whole tag. Check it gives sensible versions.
- `source.zip`: only for zip assets, a regular expression for the `.bin`
  inside, e.g. `/POLY\\.bin$`.
- `prefer`: leave it out normally. Then the page offers the newest normal
  release, and pre-releases only while there is no normal release. Set
  `"newest"` if the author marks every release as pre-release but wants the
  newest one offered anyway.

Make sure the patterns match the existing releases the way you expect, and
that they won't pick up releases of something else in the same repo.

## 3. Check it

```bash
gh repo fork ugrossek/CHOMPI-firmwares --clone=true    # ask first
cd CHOMPI-firmwares
# put the entry in firmwares/<id>.json, then:
python3 tools/firmwares.py check --changed <id>
```

`check` downloads the releases the entry would take, checks that each file is
a CHOMPI firmware (the same test the launcher does) and lists them with their
versions. Fix the entry until the list is right and nothing is marked `:x:`.
Python 3 and `gh` are all it needs.

## 4. Open the pull request

Show the author the entry and the output of `check`. With their OK, commit on
a branch and open a pull request against `ugrossek/CHOMPI-firmwares` `main`:

- Title: `Add <title>`
- Body: one or two sentences on what the firmware is, the output of `check`,
  and a note that the entry was prepared with an AI agent.

Tell the author that from now on a GitHub release is all it takes: new
releases appear on the page within about an hour, pre-releases only as
described under `prefer`. If they rename the release file or change their tag
style, the entry's patterns need updating (another small pull request).
