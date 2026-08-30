# Samsung One UI Skills

A collection of independently invokable Claude skills that audit and redesign a
codebase against **Samsung's One UI design system**.

Works on web apps, websites, Android apps and iOS apps. Each skill detects the
platform first and applies One UI as a native system where it fits (Android),
or as a translated token system where it doesn't (iOS, web) — and says which it
is doing.

---

## What's in here

Eleven skills. Nine cover the areas Samsung publishes guidance for; one audits
all nine at once; one redesigns.

| Command | Skill | What it does |
|---|---|---|
| `/one-ui` | `one-ui-audit` | **Full review.** Runs all nine areas and always writes `ONE-UI-AUDIT.md` |
| `/one-ui:audit` | `one-ui-audit` | Scores the whole codebase across all nine areas and writes `ONE-UI-AUDIT.md` |
| `/one-ui:structure` | `one-ui-structure` | Viewing/interaction split, hierarchy, navigation depth, visual depth |
| `/one-ui:layout` | `one-ui-layout` | 24dp keyline, adaptive margins, 589/960dp breakpoints, foldables, safe areas |
| `/one-ui:components` | `one-ui-components` | App bar, bottom bar, bottom navigation, buttons, dialog, lists, search, toasts |
| `/one-ui:color` | `one-ui-color` | Semantic colour roles, light/dark, functional colours, user accent, contrast |
| `/one-ui:iconography` | `one-ui-iconography` | Icon grid, stroke consistency, tintable vectors, icon colour, safe zone |
| `/one-ui:motion` | `one-ui-motion` | Real easing curves, duration scale, choreography, reduce-motion |
| `/one-ui:sound-haptic` | `one-ui-sound-haptic` | When feedback fires, semantic haptics, silent mode, visible equivalents |
| `/one-ui:writing` | `one-ui-writing` | Sentence case, verb labels, error framing, localisation |
| `/one-ui:accessibility` | `one-ui-accessibility` | Screen reader, focus order, contrast, 200% text, touch targets |
| `/one-ui:redesign` | `one-ui-redesign` | Audits, then proposes patches stage by stage — **approval required before every write** |

Each area skill carries its own checklist with stable IDs (`LAY-01`,
`A11Y-04`, `MOT-03` …) and severities, so findings are traceable between an
audit and the redesign that fixes them.

---

## Install

### Claude Code (slash commands)

Clone the repo and add it as a plugin marketplace directory:

```bash
git clone https://github.com/<you>/one-ui-skills.git
```

In Claude Code:

```
/plugin marketplace add ./one-ui-skills
/plugin install one-ui
```

Then `/one-ui:all` for the full review, or `/one-ui:color` and friends for a
single area.

> **Getting a bare `/one-ui`.** A plugin namespaces its commands, so inside the
> plugin the full review is `/one-ui:all`. If you want to type just `/one-ui`,
> copy `commands/all.md` to `~/.claude/commands/one-ui.md` — flat commands
> aren't namespaced, so the filename becomes the command.

> **Note on naming.** You asked for `/One-UI Audit`. Slash commands can't
> contain spaces, and a plugin namespaces its commands as
> `/<plugin>:<command>`. So the closest real form is `/one-ui:audit`. If you'd
> rather have flat `/one-ui-audit` commands with no namespace, drop the
> `commands/*.md` files straight into `~/.claude/commands/` renamed to
> `one-ui-audit.md` etc.

### claude.ai (uploaded skills)

claude.ai takes one skill per zip, with `SKILL.md` at the root of the zip.
Build them:

```bash
./scripts/build.sh
```

That writes eleven ready-to-upload zips into `dist/`, each bundling the shared
`reference/` files so it works standalone. Upload the ones you want from
**Settings → Capabilities → Skills**.

On claude.ai there are no slash commands — just ask, e.g. *"audit this repo
against One UI"* or *"check the colour system"*. The skill descriptions are
written to trigger on that phrasing.

---

## Where the numbers come from

Nothing in `reference/TOKENS.md` is invented. Values are traceable to:

- **[Samsung One UI design guidelines](https://developer.samsung.com/one-ui/index.html)** — the 24dp keyline, Reject and Grip zones, the 600dp large-screen threshold, the four design principles, the three writing principles, the six accessibility areas.
- **[tribalfs/oneui-design](https://github.com/tribalfs/oneui-design)** (MIT) — the actively maintained One UI design library for Android, covering One UI 6/7/8 via SESL. Source of the real adaptive-margin algorithm (`< 589dp` → 0%, `589–959dp` → 5%, `>= 960dp` → 12.5%), the cubic-bezier easing curves, the duration scale, the 26dp button radius, the 17sp body size, and the semantic colour roles.
- **[OneUIProject/oneui-design](https://github.com/OneUIProject/oneui-design)** (MIT) — the original Java library. Last updated May 2024; the tribalfs fork supersedes it and is what the skills reference.

Where a value is a reasonable derivation rather than a published constant, it
is marked **(derived)** in the token file.

## The scanner

`scripts/oneui_scan.py` is a dependency-free static pass that catches the
mechanically checkable violations across web, Android and iOS files:

```bash
python3 scripts/oneui_scan.py ./src          # human-readable
python3 scripts/oneui_scan.py ./src --json   # machine-readable
```

It emits the same check IDs as the skills (`LAY-01`, `A11Y-02`, `WRT-01` …),
so its output feeds straight into an audit report. It is a starting point, not
the audit — it can't judge structure, icon metaphors, motion meaning or copy
quality, and regex findings need verifying before they're reported.

## Honest limits

- One UI is an **Android** design system. On iOS and desktop web, several
  rules are adaptations, not implementations. The skills mark these explicitly
  rather than pretending to conformance.
- The audit reads code. It cannot see rendered output, so contrast and text
  scaling findings are computed from resolved values where possible and
  flagged as "needs manual verification" where not.
- The Android component library is a real dependency with a minimum SDK and
  build impact. The redesign skill proposes it and explains the tradeoff — it
  never adds it silently.
- Samsung's own guidance is high-level in places (motion, sound). Where the
  skills are more specific than Samsung publishes, the specificity comes from
  the library source, not from guesswork.

## Layout

```
one-ui-skills/
├── .claude-plugin/plugin.json
├── commands/              # 11 slash commands
├── skills/                # 11 skills, each with SKILL.md
├── reference/
│   ├── TOKENS.md          # the shared token source of truth
│   ├── PLATFORMS.md       # platform detection and translation
│   └── REPORT.md          # the report contract and template
├── scripts/
│   ├── build.sh           # builds claude.ai upload zips into dist/
│   └── oneui_scan.py      # deterministic pre-pass scanner
└── README.md
```

## Licence and attribution

This skill collection is MIT licensed. It contains **no Samsung code and no
Samsung assets** — only documented design values and references to public
guidelines.

One UI is a design system by Samsung Electronics. "One UI", "Samsung" and
"Galaxy" are trademarks of Samsung Electronics. This project is not affiliated
with or endorsed by Samsung. Applying these skills does not make an app an
official Samsung app, and shipping something that impersonates Samsung's own
apps is your call and your risk.

`tribalfs/oneui-design` and `OneUIProject/oneui-design` are both MIT licensed;
see `ATTRIBUTION.md`.
