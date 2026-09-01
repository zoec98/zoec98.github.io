---
title: "Building Books from Markdown"
date: 2026-09-01T00:00:00Z
description: "How our book files become EPUB, DOCX, HTML, and ODT documents"
tags: [ "Writing", "Publishing" ]
draft: false
---

Our books are written as ordinary Markdown files. A small build system turns those files into formats that are easier to read, share, or upload: EPUB for e-readers, DOCX for Word-compatible editors, HTML for a web browser, and ODT for LibreOffice.

You do not need to understand the internal machinery to use it. Usually, you write your chapters, open a terminal in the book folder, and type one short command:

```sh
gmake
```

The finished files are placed in an `out` folder. The most important result is one EPUB containing the whole book, plus one DOCX file for each chapter.

[Download the ready-to-build example book](building-books-example.zip). Unzip it, open the resulting folder in a terminal, and run `gmake`. It includes a completed `book.yaml`, the build files, and a short Lorem Ipsum chapter that is also a Markdown primer.

## The three supporting files

Each book folder has three important build files alongside its chapter Markdown files.

`book.yaml` contains the shared information about the book: its title, author, language, identifier, and sometimes cover image information. This means that you write these details once instead of repeating them in every chapter. The build process adds this information to every exported file.

The `Makefile` is the list of build commands. Think of it as a menu of repeatable jobs. It knows which chapter files belong to the book, which program to use, and where to put the results. Chapter files are named with a two-digit number at the beginning, such as `01-opening.md` or `12-the-journey.md`. This keeps them in the correct order when the complete EPUB is made.

The Lua file, `filters/docx-double-paragraphs.lua`, is only used for DOCX exports. It adds a blank paragraph after normal text paragraphs. Word and other document editors keep this spacing, which makes each chapter more comfortable to copy and paste into DeviantArt Literature deviations. It leaves image and note paragraphs alone, so they are not given an unwanted extra gap.

## The files

Put the following files in your book folder. The Lua file goes in a `filters` subfolder.

### `book.yaml`

This is the template, shown exactly as it is stored:

```yaml
---
title: {{TITLE}}
author: {{AUTHOR}}
lang: {{LANG}}
identifier: {{IDENTIFIER}}
{{COVER_BLOCK}}...
```

Before building, replace `{{TITLE}}`, `{{AUTHOR}}`, and `{{LANG}}` with your book's details.

Use a short language code such as `en`, `de`, or `fr`.

Replace `{{IDENTIFIER}}` with a unique identifier; a UUID written as `urn:uuid:...` is a good choice. On a Mac, run the command `uuidgen` and then use the resulting string as the uuid value: `urn:uuid:4088A941-1A97-4F7F-A9AE-1FAC41A40625` or similar. This is used by EPUB readers to determine book identity, so that a book will not show up twice in the catalog.

If your EPUB has a title image, replace `{{COVER_BLOCK}}` with a line such as `cover-image: assets/title.jpeg`; create a directory `assets` and put `title.jpeg` in that directory. If no title image is being used, remove that line entirely.

The trailing `...` ends the YAML document and should remain.

Always put title, author, lang, identifier, or the title file name in double quotes, so `author: "Zo� Cordelier"` including the quites. Leaving them off might work until there is puncuation, and then the problem will be hard to find.

## Images and covers

Keep book images in an `assets/` folder next to the Markdown chapters. A common choice for the cover is `assets/title.jpeg`. To use it as the EPUB cover, add this line to `book.yaml` before the final `...`:

```yaml
cover-image: assets/title.jpeg
```

Use images in a chapter with normal Markdown syntax, for example:

```md
![](assets/theimage.jpg)
```

Pandoc follows those image references when it builds the EPUB and includes the image files in the finished book. Keep the paths relative to the chapter file, and make sure the images are present before running `gmake`.

### `Makefile`

```makefile
PANDOC ?= pandoc
YQ ?= yq
OPEN ?= open

BOOK_META_FILE ?= book.yaml
OUT_DIR ?= out

READEST_APP ?= Readest

DOCX_FILTER_FILE := filters/docx-double-paragraphs.lua

CHAPTERS := $(sort $(wildcard [0-9][0-9]-*.md))
TITLE_SLUG := $(shell $(YQ) -r '.title // "book"' $(BOOK_META_FILE) | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$$//')

HTML_FILES := $(patsubst %.md,$(OUT_DIR)/%.html,$(CHAPTERS))
EPUB_FILES := $(patsubst %.md,$(OUT_DIR)/%.epub,$(CHAPTERS))
ODT_FILES := $(patsubst %.md,$(OUT_DIR)/%.odt,$(CHAPTERS))
DOCX_FILES := $(patsubst %.md,$(OUT_DIR)/%.docx,$(CHAPTERS))
ALL_EPUB := $(OUT_DIR)/$(TITLE_SLUG).epub

PANDOC_METADATA := --metadata-file=$(BOOK_META_FILE)
EPUB_OPTIONS := $(PANDOC_METADATA) --toc --toc-depth=3
DOCX_OPTIONS := --lua-filter=$(DOCX_FILTER_FILE) $(PANDOC_METADATA)

.PHONY: all html epub odt docx readest clean check-tools date-start date-end

all: check-tools date-start docx $(ALL_EPUB) date-end

html: check-tools $(HTML_FILES)
epub: check-tools $(EPUB_FILES)
odt: check-tools $(ODT_FILES)
docx: check-tools $(DOCX_FILES)

readest: check-tools $(ALL_EPUB)
	$(OPEN) -a "$(READEST_APP)" "$<"

check-tools:
	@command -v $(PANDOC) >/dev/null 2>&1 || { echo "Missing required tool: $(PANDOC). Install pandoc before building."; exit 1; }
	@command -v $(YQ) >/dev/null 2>&1 || { echo "Missing required tool: $(YQ). Install yq before building."; exit 1; }

date-start date-end:
	@echo "Date: $(shell date)"

$(OUT_DIR):
	mkdir -p $@

$(OUT_DIR)/%.html: %.md $(BOOK_META_FILE) | $(OUT_DIR)
	$(PANDOC) $< $(PANDOC_METADATA) -o $@

$(OUT_DIR)/%.epub: %.md $(BOOK_META_FILE) | $(OUT_DIR)
	$(PANDOC) $< $(EPUB_OPTIONS) -o $@

$(ALL_EPUB): $(CHAPTERS) $(BOOK_META_FILE) | $(OUT_DIR)
	build_time="$$(date -u '+%Y-%m-%dT%H:%M:%SZ')"; \
	$(PANDOC) $(CHAPTERS) $(EPUB_OPTIONS) --metadata date="$$build_time" --metadata modified="$$build_time" -o $@

$(OUT_DIR)/%.odt: %.md $(BOOK_META_FILE) | $(OUT_DIR)
	$(PANDOC) $< $(PANDOC_METADATA) -o $@

$(OUT_DIR)/%.docx: %.md $(BOOK_META_FILE) $(DOCX_FILTER_FILE) | $(OUT_DIR)
	$(PANDOC) $< $(DOCX_OPTIONS) -o $@

clean:
	rm -rf $(OUT_DIR)
```

A Makefile is a list of rules. Its basic shape is `target: sources`, followed by a new line beginning with a **tab** and then the action to perform. A target is the thing you ask for, such as `docx`; its sources are the files it needs. Do not replace the tabs before action lines with spaces. Make requires real tabs, and a text editor can silently change them.

### `filters/docx-double-paragraphs.lua`

```lua
local function para_is_empty(para)
  return #para.content == 0
end

local function para_contains_structural_inline(para)
  for _, inline in ipairs(para.content) do
    if inline.t == "Image" or inline.t == "Note" then
      return true
    end
  end
  return false
end

local function should_double_space_para(para)
  return not para_is_empty(para) and not para_contains_structural_inline(para)
end

local function spacer_para()
  -- Pandoc's DOCX writer drops truly empty paragraphs, so use a single
  -- non-breaking space to force Word to keep a visually blank paragraph.
  return pandoc.Para({ pandoc.Str("\u{00A0}") })
end

local function expand_blocks(blocks)
  local expanded = {}

  for _, block in ipairs(blocks) do
    if block.t == "Div" then
      block.content = expand_blocks(block.content)
      table.insert(expanded, block)
    else
      table.insert(expanded, block)

      if block.t == "Para" and should_double_space_para(block) then
        table.insert(expanded, spacer_para())
      end
    end
  end

  return expanded
end

function Pandoc(doc)
  doc.blocks = expand_blocks(doc.blocks)
  return doc
end
```

You normally do not need to edit this file. Pandoc runs it only for DOCX exports, where it inserts the visible blank paragraphs needed for comfortable DeviantArt pasting.

## What you need installed

The build uses two programs:

- **Pandoc**, which converts Markdown into EPUB, DOCX, HTML, and ODT.
- **yq**, which reads the title from `book.yaml` so the complete EPUB receives a sensible filename.

Before it starts building, the Makefile checks for both programs and gives a clear message if one is missing. You only need to install them once on a computer.

On a Mac with [Homebrew](https://brew.sh/), install Pandoc, yq, and GNU Make with:

```sh
brew install pandoc yq gmake
```

Use `gmake` rather than the older `make` supplied with macOS. It is the version this build is designed to use.

On Linux, the equivalent command is usually `make` (`gmake` is the Linux `make` on a Mac, renamed so that it doesn't clash with the older Mac `make`.

## The useful commands

Run these commands from the folder that contains the book's `Makefile`.

- `gmake` builds the usual publishing package: DOCX files for all chapters and one complete EPUB for the book.
- `gmake docx` makes only the chapter DOCX files. Use this when you want to publish or update a chapter on DeviantArt.
- `gmake epub` makes one EPUB for each individual chapter.
- `gmake html` makes an HTML file for each chapter, useful for a quick browser preview.
- `gmake odt` makes OpenDocument Text files for each chapter, useful with LibreOffice.
- `gmake readest` opens the complete-book EPUB in the Readest reader app, if it is installed.
- `gmake clean` removes the generated `out` folder. It does not remove your Markdown chapters or `book.yaml`.

For a fast, completely fresh build on a Mac, use:

```sh
time ( gmake clean; gmake -j )
```

`gmake clean` removes the old generated files. The semicolon means “then run the next command.” `gmake -j` builds independent work at the same time, which can be faster than building one file after another. The parentheses keep both steps together, and `time` reports how long the whole clean rebuild took.

For example, after editing chapter 07, you can run `gmake docx`, then open `out/07-your-chapter-title.docx`. Copy its contents and paste them into a DeviantArt Literature deviation. The extra blank lines are intentional: they preserve the readable paragraph spacing there.

## How the complete EPUB is made

The complete-book EPUB uses all numbered chapter files, in filename order, and includes a table of contents up to three heading levels deep. Its filename comes from the book title in `book.yaml`, converted into a simple web-friendly form. For example, a title of *The Moon Garden* becomes `out/the-moon-garden.epub`.

The build also records the current UTC date and time in the complete EPUB. This helps e-readers and libraries recognise a newly built edition after a change.

## A simple routine

1. Write or edit the Markdown chapters.
2. Check that the title and other shared details in `book.yaml` are still correct.
3. Run `gmake`.
4. Open the EPUB to check the book as a reader will see it.
5. Use the chapter DOCX files when preparing DeviantArt Literature deviations.

Because all exports come from the same Markdown source, a correction only needs to be made once. Rebuilding then gives you matching versions for readers, editors, and web previews.
