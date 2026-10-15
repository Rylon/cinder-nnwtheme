<!-- nnw-theme-stub: agents v2 -->
# Repository guidance

This repository is one NetNewsWire theme: the single root `*.nnwtheme` bundle,
its `fixtures/`, and `screenshots/`. The tooling is the `nnw-theme` npm package;
run it as `npx nnw-theme@2 <command>`. Never install it globally and never add
it to this repository. Read `npx nnw-theme@2 guide` before theme work.

Prefer CSS changes; change
`template.html` only when the requested structure or behavior requires it.
Finish theme work with `npx nnw-theme@2 check` and report the preview path.

Do not change `ThemeIdentifier` or the `.nnwtheme` bundle name after the first
release. Increase the plist `Version` (`npx nnw-theme@2 bump`) before every
release.

If the tooling misbehaves, do not work around it here: report it at
https://github.com/dave-atx/nnw-theme/issues with the `check` version line.

Local edits, renders, checks and packages are safe. Creating a GitHub release,
opening a pull request, changing repository topics, or otherwise mutating
GitHub requires the user's explicit intent. Theme and fixture HTML/JavaScript
are untrusted executable inputs; browser checks stay loopback-only.

## Cinder

Cinder is a **NetNewsWire article theme**, a restyle of
[Ember](https://github.com/dave-atx/ember-nnw-theme) by Dave Marquard. The shipped
product is the bundle in `Cinder.nnwtheme/`, three files:

| File | What it is |
|------|-----------|
| `Cinder.nnwtheme/template.html` | The article scaffold. NNW fills `[[macro]]` placeholders (title, byline, dates, avatar, body). It also contains the inline footnote-format adapter from Ember; themes cannot load a bundled `.js` file. |
| `Cinder.nnwtheme/stylesheet.css` | The whole theme. NNW loads its own `core.css` **first**, then this replaces the default stylesheet. |
| `Cinder.nnwtheme/Info.plist` | Theme name, identifier, author, version. |

Everything else is supporting material: `fixtures/` holds the test articles and
`screenshots/theme-preview.png` is the README image (`npx nnw-theme@2 screenshot
--fixture article --card --promote`). There is no project site; Ember's `docs/` was
removed.

The tooling is the `nnw-theme` npm package, run as `npx nnw-theme@2 <command>`.
The four workflows, the `creating-nnw-themes` skill, and the guidance above this
section are stubs from
[netnewswire-theme-template](https://github.com/dave-atx/netnewswire-theme-template).
**Fix tooling in [nnw-theme](https://github.com/dave-atx/nnw-theme), never here**, and
keep Cinder's own guidance in this section. The skill repeats this file, and
`pages.yml` publishes only the check preview gallery; both stay only because `check`
warns when a stub is missing.

### How NetNewsWire renders a theme (know this before touching CSS)

A rendered article = a platform **page skeleton** with three macros filled:
`[[title]]`, `[[style]]`, `[[body]]`.

- `[[style]]` = NNW's `core.css` **+** this theme's `stylesheet.css`. Your CSS
  is layered on top of core.css, not instead of it.
- `[[body]]` = `template.html` filled with the article's data.
- Platform differences matter: on **iOS/iPad** `[[font-size]]` is substituted and
  Dynamic Type drives sizing; on **macOS** `[[font-size]]` is left literal and a
  `text_size_class` is set instead. Dark mode is `@media (prefers-color-scheme: dark)`.

`npx nnw-theme@2 guide theme-format` has the details.

### Developing & testing

**Do not eyeball CSS changes.** `npx nnw-theme@2` reproduces NNW's pipeline from
pinned NetNewsWire files and checks every fixture in WebKit, the engine NNW uses.
After `render`, look at `build/preview/` in light and dark on each device and show
the user the change before moving on:

```sh
npx nnw-theme@2 render [fixture ...]   # write build/preview/ once
npx nnw-theme@2 check                  # release gate; finish every change with it
```

`check` renders the package's two built-in fixtures across Mac, iPhone, and iPad, and each
Cinder fixture on Mac and iPhone, all in light and dark. It fails on overflow, broken
images, page errors, and footnotes that don't resolve, stay legible, match each
other, or open their popover. `fixtures/footnote-providers.toml` pins the note every
footnote format must open through its `[expect.footnotes]` table, and every footnote
fixture sets `keep_with_word = true` so no marker wraps apart from its word. Real-article
fixtures are captured from NetNewsWire with `npx nnw-theme@2 capture`.

### Conventions

- Fixtures are TOML in `fixtures/` (keys mirror the template variables; HTML bodies
  use `'''triple-quoted'''` literals). Keep interesting edge cases: long byline,
  long title, pull-quotes, figures, footnotes, smart quotes.
- `build/`, `dist/`, and `*.nnwtheme.zip` are build artifacts; never commit them.
- Additional footnote popovers depend on NetNewsWire's **Article JavaScript**
  setting (enabled by default). Keep the adapter inline in `template.html`; `check`
  also renders with theme scripts removed to prove the article still reads well.
- Cinder is light only: there are no dark-mode rules and `color-scheme` is
  `light`, so dark renders must look identical to light. The ground is warm and
  the accent is NetNewsWire's default blue, `#086AEE`.
- Headings and quotations use `--ember-serif` (New York). There is no drop cap and
  no accent bar over section headings.
- The `--ember-*` custom properties, `ember-*` classes and `emberFootnotes` keep
  Ember's names so its changes merge cleanly. Don't rename them.
- Commit messages are Conventional Commits; `.github/cliff.toml` turns them into
  the release notes, so the type prefix decides which section a change lands in
  (and `chore(release):` / `ci:` are omitted entirely).

### Keeping in step with Ember

Cinder merges Ember's changes rather than porting them. A fresh clone only has
`origin`, so add Ember as `upstream` first:

```sh
git remote add upstream https://github.com/dave-atx/ember-nnw-theme.git
git config remote.upstream.tagOpt --no-tags
git fetch upstream '+refs/heads/*:refs/remotes/upstream/*' '+refs/tags/*:refs/upstream-tags/*'
```

Ember's tags live under `refs/upstream-tags/` so they never mix with Cinder's own
`v*` tags, which git-cliff uses to find the previous release. Merge with
`git merge --no-commit upstream/main` (or an Ember tag) and leave the commit to the
user. When resolving:

- **Bundle path:** Ember's bundle is still `Ember.nnwtheme/`. Git normally follows
  the rename; if a new upstream file lands in `Ember.nnwtheme/`, move it into
  `Cinder.nnwtheme/`.
- **`docs/`:** Cinder has no site. Keep every deletion (`git rm` the files).
- **`Info.plist`:** keep Cinder's identity and `Version`. Ember's version numbers
  don't apply here.
- **`README.md`:** keep Cinder's. If Ember adds a feature worth mentioning, add it
  under "Kept from Ember".
- **`stylesheet.css` and `template.html`:** take Ember's layout, image, caption and
  footnote fixes, but keep Cinder's look where they differ (see Conventions). Drop
  any new `prefers-color-scheme: dark` rules and anything that restores the drop cap
  or heading bars.

Finish with `npx nnw-theme@2 check`, and show the user any fixture whose look changed.

### Releasing

Two version numbers move together, and they are **not** the same number:

| Where | Form | Example |
|-------|------|---------|
| Git tag | `vMAJOR.MINOR` | `v1.1` |
| `Cinder.nnwtheme/Info.plist` → `Version` | a plain monotonic **integer**, +1 every release | `2` |

NetNewsWire compares that integer to decide whether an installed theme is
out of date, so it must increase on every release regardless of how the tag
moves. It is *not* derived from the tag. Cinder restarted at 1 under its new
identifier, so Ember's numbers mean nothing here.

```sh
npx nnw-theme@2 bump                 # Version +1
git commit -am 'chore(release): v1.1'
git push
```

Then run **Actions → Publish theme** on GitHub with the tag `v1.1`, leaving the notes
empty. Publishing is an external action: confirm with the user first. The workflow
releases the default-branch head after the full check, refuses a reused tag or a
Version that did not increase, attaches `Cinder.nnwtheme.zip`, and writes the notes
from the commits since the last `v*` tag with git-cliff. The README's download link
points at `releases/latest`, so nothing else needs editing.

The first release has no earlier `v*` tag, so git-cliff would list all of Ember's
history. Write those notes by hand in the workflow's notes field instead.
