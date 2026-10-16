# TEST — deck jetable, séparé du deck du quotidien

Copie locale de `anki-deck-wset-3` pour développer un add-on sans toucher au deck
utilisé tous les jours. Quatre cartes seulement. Anki voit un autre paquet :

- decks : `TEST WSET 3 Wine`, `TEST WSET 3 VIN`, `TEST WSET 3`
- types de notes : `TEST WSET 3 MCQ`, `TEST WSET 3 Cloze`, …
- GUID : `guid_for("wset3-test", id, lang)`
- paquets : `wset3-test-en.apkg`, `wset3-test-fr.apkg`

Importer ce dépôt ne met pas à jour `WSET 3 Wine` / `WSET 3 VIN`.

# Wine Theory Anki

An open, version-controlled and bilingual **Anki deck for advanced wine study**, built
around the knowledge expected for the WSET Level 3 Award in Wines.

The cards are written independently in English and French, stored as YAML, reviewed in
Git, and compiled into standard `.apkg` packages. You can download a ready-made deck,
build it locally, or import and synchronize it with AnkiWeb in one command.

> **Projet francophone ?** Le paquet français, les noms de paquets et l’interface des
> cartes sont localisés. Le code et la documentation du dépôt restent principalement en
> anglais pour faciliter les contributions internationales.

> [!IMPORTANT]
> This is an independent study aid. It is **not affiliated with, endorsed by, or connected
> to the Wine & Spirit Education Trust (WSET)**. WSET is a registered trademark of the
> Wine & Spirit Education Trust. No official textbooks, workbooks or exam questions are
> reproduced here.

## What the project provides

- English and French Anki packages generated from the same card IDs.
- MCQ, basic and cloze cards with localized deck names and interfaces.
- Shuffled MCQ choices on every review.
- Stable note identifiers, so importing a newer release updates existing notes without
  creating duplicates or resetting their scheduling.
- Schema validation, tests and automated release builds.
- A review workflow for factual accuracy, credible distractors and natural French.

The project is a work in progress. English coverage is currently broader than French
coverage. Draft cards are excluded from normal builds, and cards without a `fr:` block
are omitted from the French package.

See the [project roadmap and completion checklist](ROADMAP.md) for the current migration,
translation and quality-review status.

## Use the ready-made deck

The easiest option does not require Python or any development tools.

1. Install [Deck Manager](../anki-deckdock) in Anki, then paste the download URL of one
   package from a versioned release:

   ```text
   https://github.com/{owner}/{repo}/releases/download/v0.1.0/wset3-test-fr.apkg
   ```

   `wset3-test-en.apkg` is the English package. The two files are two separate follows.
2. Confirm Anki’s import dialog. Deck Manager records the version only after that dialog
   succeeds.

Choose the English or French package according to the language in which you want to
study. Importing both is supported, but creates separate English and French notes and
deck trees.

### Updating an existing installation

Deck Manager reads GitHub’s stable latest release (`GET /repos/{owner}/{repo}/releases/latest`).
A new `vX.Y.Z` tag with the same file name is an update. The same tag is treated as
current, even if the file bytes changed. Replacing the assets of a published version
does not notify anyone.

Released card IDs are permanent, and the generated Anki GUIDs are deterministic. Anki
can therefore update the existing notes while retaining review history and scheduling.

As with any Anki collection change, keeping a recent backup is sensible. Anki creates
automatic backups, which can be managed from the profile screen.

## Build from source

### Requirements

- [Python 3.12 or later](https://www.python.org/downloads/)
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- `make` for the short commands below; every command also has a direct `uv` equivalent

From the repository root:

```bash
uv sync --extra dev
make check
make build
```

The generated packages are written to `dist/`:

```text
dist/wset3-test-en.apkg
dist/wset3-test-fr.apkg
```

To build only one language:

```bash
uv run wset3-anki build --lang en --out dist
uv run wset3-anki build --lang fr --out dist
```

Normal builds skip `status: draft` cards. For local editorial previews only, you can add
`--include-drafts`.

### Useful commands

| Command | Purpose |
| --- | --- |
| `make check` | Run linting, formatting checks, type checks, schema validation and tests |
| `make build` | Build both English and French `.apkg` packages |
| `make push` | Build French, import it into Anki Desktop and sync AnkiWeb |
| `make push ANKI_LANG=en` | Build, import and sync the English package |
| `make push ANKI_LANG=all` | Build, import and sync both language packages |
| `uv run wset3-anki validate` | Validate only the card YAML files |
| `uv run wset3-anki schema` | Regenerate `schema/cards.schema.json` after a schema change |

If `make` is unavailable, use the equivalent commands directly:

```bash
uv run wset3-anki check
uv run wset3-anki build --lang all --out dist
uv run wset3-anki push --lang fr --out dist
```

## Build, import and sync with AnkiConnect

[`make push`](#useful-commands) provides the shortest local publishing workflow. It
builds the selected package, starts Anki Desktop when possible, imports the `.apkg`, and
asks Anki to synchronize the active profile with AnkiWeb.

### One-time setup

1. Install [Anki Desktop](https://apps.ankiweb.net/) and connect the intended profile to
   your AnkiWeb account.
2. In Anki, open **Tools → Add-ons → Get Add-ons**.
3. Enter the AnkiConnect add-on code **`2055492159`**. See the
   [AnkiWeb add-on page](https://ankiweb.net/shared/info/2055492159) or the
   [AnkiConnect repository](https://github.com/FooSoft/anki-connect) for details.
4. Restart Anki Desktop once so the add-on is loaded.

You can then publish the French package with:

```bash
make push
```

Or select another language:

```bash
make push ANKI_LANG=en
make push ANKI_LANG=all
```

The command prints the number of imported notes and confirms the AnkiWeb sync when it
finishes.

> [!WARNING]
> The final sync concerns the **entire active Anki profile**, not only this deck. Before
> running `make push`, make sure Anki Desktop is using the intended profile and AnkiWeb
> account. Resolve any pending one-way sync prompt in Anki itself first.

By default the tool contacts AnkiConnect only on `http://127.0.0.1:8765`. Do not expose
an unauthenticated AnkiConnect endpoint to the public internet. Advanced installations
can set a custom endpoint or API key:

```bash
ANKI_CONNECT_URL=http://127.0.0.1:8765 \
ANKI_CONNECT_KEY=your-key \
make push
```

To require Anki to be opened manually instead of allowing the command to launch it:

```bash
uv run wset3-anki push --lang fr --out dist --no-launch
```

### AnkiConnect troubleshooting

- **Cannot reach AnkiConnect:** confirm that add-on `2055492159` is installed, restart
  Anki, and leave Anki Desktop open while retrying.
- **Anki cannot be launched automatically:** open it manually and run the command again,
  or use `--no-launch`.
- **Synchronization fails:** check that the active Anki profile is connected to AnkiWeb
  and complete any sync or conflict prompt directly in Anki.
- **Custom AnkiConnect configuration:** set `ANKI_CONNECT_URL` and, if configured in the
  add-on, `ANKI_CONNECT_KEY`.

## Contributing

Contributions are welcome: factual corrections, better distractors, clearer explanations,
French translations, new cards, templates, tests and tooling all help.

If you have found a problem but do not want to edit YAML, use the structured
[issue templates](../../issues/new/choose) to report a card or a build problem.

Before editing cards, read:

- [CONTRIBUTING.md](CONTRIBUTING.md) for the development and pull-request workflow;
- [CONTENT_GUIDELINES.md](CONTENT_GUIDELINES.md) for authorship and card-writing rules;
- [import/STUDY_CARD_QUALITY.md](import/STUDY_CARD_QUALITY.md) for the chapter review rubric.

The short version is:

1. Fork the repository and create a focused branch.
2. Edit the existing chapter file in `cards/`; do not create one file per card.
3. Use original wording and public, checkable sources. Never copy WSET course materials
   or exam questions.
4. Give new facts `status: draft`. Never rename or reuse a released card `id`.
5. If a card is bilingual, keep the same choice count and correct-choice index in `en:`
   and `fr:`.
6. Run `make check`.
7. Open a focused pull request explaining what changed, why, and which sources support
   factual corrections.

Small corrections can go directly to a pull request. For a large new chapter, a schema
change or a substantial workflow change, open an issue first so the approach can be
agreed before a large amount of work is written.

You do **not** need Anki or AnkiConnect to contribute card content. They are only needed
to preview or automatically import a built package.

## Repository layout

```text
cards/                  YAML source of truth, one file per source chapter
import/                 Migration ledger and editorial quality-review material
schema/                 Generated JSON Schema for card YAML
templates/              Anki HTML, CSS, JavaScript and localized UI/deck names
src/wset3_anki/         Validator, builder and AnkiConnect integration
tests/                  Unit and integration tests
.github/                CI, release workflow and pull-request template
dist/                   Locally generated packages; not source
```

The stable `deck:` values in YAML remain in English, for example
`France::Burgundy`. Display names are localized at build time, producing paths such as
`TEST WSET 3 Wine::France::Burgundy` and `TEST WSET 3 VIN::France::Bourgogne`.

Editors with YAML language-server support can use `schema/cards.schema.json` for inline
validation. If `src/wset3_anki/schema.py` changes, regenerate and commit the schema:

```bash
uv run wset3-anki schema
```

## Releases

Every successful push to `main` publishes the next patch release: `v0.1.0`, then
`v0.1.1`, and so on. Deck Manager treats a new `vX.Y.Z` as an update and ignores a
file that was replaced under the same number, so the workflow never reuses a tag.

The asset names stay the same:

```text
wset3-test-en.apkg
wset3-test-fr.apkg
```

Pull requests build the packages and do not publish. The follow URL uses the published
number, for example `…/releases/download/v0.1.0/wset3-test-fr.apkg`.

Because card IDs, model IDs and deck IDs are stable, a newer package updates existing
notes. Contributors must therefore never rename a released card ID merely to improve
its wording.

## License and attribution

- Code in `src/`, `templates/`, tests and automation is licensed under the
  [MIT License](LICENSE).
- Card content in `cards/` is licensed under
  [CC BY-SA 4.0](LICENSE-CONTENT).

If you redistribute or adapt the cards, provide appropriate attribution and distribute
your card-content changes under the same CC BY-SA 4.0 license.

WSET is a registered trademark of the Wine & Spirit Education Trust. This independent
project is not affiliated with, endorsed by, or connected to WSET.
