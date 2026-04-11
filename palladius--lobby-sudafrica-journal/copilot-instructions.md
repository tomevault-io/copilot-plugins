## lobby-sudafrica-journal

> This GH repo was created by Openclaw while Riccardo was dictating stuf via Telegram on a trip to Southafrica, with zero access to the machine!

# Lobby Journal Technical Guide

This GH repo was created by Openclaw while Riccardo was dictating stuf via Telegram on a trip to Southafrica, with zero access to the machine!
The files are quite messy and unorganized.

- `git log` should highlight a huge commit for each day, where Openclaw created the file and added all the content. Some files were mistakenly overwritten insted of edited, so good info has been delted, like in 10feb example. `git diff` on GITROOT/2026-10-DD.md should show an interesting diff, and many DELETED files should show the bug in the editing tool that Lobby had at the time.

## 🛠️ Environment & Git

- **Repo:** `https://github.com/palladius/lobby-sudafrica-journal/`
- **Branch:** `master` only. **DO NOT use `main`**.
- **Timezone:** Always SAST (GMT+2).

## 🖼️ Image Management & Recovery

- **Types:**
  - **Pixar:** AI-generated (`images/pixar/`).
  - **Real:** Camera photos (`images/real/`).
- **Mapping:** All AI/Real pairings **MUST** be tracked in `~/.openclaw/workspace/banana_mapping.csv`. Update this file if moving/renaming images.
- **Recovery:**
  - **Pixar:** Search `~/.openclaw/workspace/nano_banana/` by date/name.
  - **Real:** Search session history for `telegram-USERID-MSGID.jpg` and ask Riccardo to re-upload.

Whenever you add images and you know there's original X and pixar version Y, add both to the post **and** update `images_mappings.csv`!

### 📊 Image Mapping

`images_mappings.csv` contains a very simple schema:

1. Timestamp of the image creation/additio1. Permalink to original image
2. Permalink to pixar image
3. Description

## 🧪 Post-Trip Harmonization

1. Ensure all dates/times are correct.
2. Verify all image links in `.md` files. **NEW RULE**: Use the interactive toggle template: `{% include captionizer.html original="..." pixar="..." caption="..." %}` (do as in jekyll-site/\_posts/2026-02-21-colori-panorami-e-una-lezione-di-vita.md page which is perfect).
3. Confirm all mappings exist in `banana_mapping.csv`.
4. Use `git mv` to move files from `images/XXX` to `jekyll-site/assets/images/YYYY-MM-DD/`.

More under `CLEANUP_ORGANIZATION.md`

## Move to jekyll, CloudFlare pages and wrangler

1. `git mv 2026-02-XX.md` into `jekyll-site/_posts/YYYY-MM-DD-title-of-the-day.md`. This is important, to preserve the order of the posts and the history of the files. Lobby did A LOT of small commits, and might have removed some files.
2. Check git log of that file since inception. Shouldnt be more than 10 changes each. Look if there was any change that removed more than 3 lines at once. If so, this is a sign of a bug in Edit tool (edit becomes a substitution) -> in this case dump this as 2026-02-XX-commit-YYYYYY.md so a human can review how to integrate the lost data in current one.
3. Take notes into `tmp/2026-02-XX.log`.
4. run `just test` to catch for automated errors. Any broken images, ensure they get fixed, possibly by moving images from ./images/ or ./assets/ into jekyll-site/assets/images/ . Remember the DATE and pixar/original dichotomy!
5. After you're done, use bin/find_openclaw_images.sh to find any missing images and possibly add them to the post (confirm with user! Link them as artifacts as user observes the localost:4000 page to see a visual diff).
6. If this is a BIG thing, andor if user is not listening, open a <gh> bug on the repo like "Riccardo review day XX" and link the page and the added pictures so user can review which pictures should be put there. For every additional picture, add a line in the bug report with a possible name/description. If this is very LLM intensive, feel free to code a "bin/find_caption_for_image.sh <IMAGE>". Use a nice prompt which can be used in a safari or Family trip to South Africa. Also it should say if its Picture or Generated.

## Skills

- If you are on pupurabbux or `mini-lobby`, you can use the "Lobby on Pupurabbux" skill to find missing images.
- use `bin/pixerizza.py` to pixerize original pictures moved from Google Photos.

## Tests

- Before commit, launch tests: `just test`
- You can also test a single day like this: `TEST_SUDAFRICA_DAY=20260210 just test`

## Git

- **IMPORTANT** use `git mv` whenever possible!!! Dont `mv` as it looks like git rm + git add and we lose history!!!
- do NOT commit files in private/ or tmp/. I need to keep tmp/ out of gitignore to let AI manage files in there! ;)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/palladius)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/palladius)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
