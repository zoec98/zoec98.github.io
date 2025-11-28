# Adding New Books

These steps assume you already have Hugo installed locally and are running `hugo serve -D -E -F` while editing.

1. **Create a folder** under `content/books/` that matches your book's slug (example: `content/books/space-odyssey/`).
2. **Add `_index.md`** inside that folder with front matter similar to:
   ```md
   ---
   title: "Space Odyssey"
   description: "Optional short elevator pitch."
   weight: 2
   bookCollapseSection: true
   ---

   Opening paragraph or blurb for the landing page.
   ```
   - `title` shows in the sidebar.
   - `weight` controls book ordering among other books.
   - Setting `bookCollapseSection` to `true` keeps long chapter lists tidy.
3. **Create one markdown file per chapter** in the same folder. Use numbers in filenames only if it helps you; chapter order is controlled via the `weight` field:
   ```md
   ---
   title: "Chapter Title"
   weight: 1
   draft: false
   ---

   Chapter text goes here.
   ```
4. **Use front matter wisely**:
   - `draft: true` hides a chapter/book from production but still shows locally when you pass `-D`.
   - `weight` must increase with each chapter to keep them sorted.
   - Dates are optional for books, but you can add `date:` if you want them to appear in RSS feeds.
5. **Link to or from the blog** (optional): reference supporting posts via regular Markdown links such as `/blog/hello-update/` to connect lore drops with timeline updates.
6. **Preview search**: new content becomes searchable when you run `hugo serve`. No extra steps required.
7. **Commit your changes** (including the new folder) and open a pull request for review.

If you need a template, copy the existing `content/books/hello-world/` folder and update the metadata/text before committing.
