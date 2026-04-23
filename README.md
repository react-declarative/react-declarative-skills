<img src="https://github.com/react-declarative/react-declarative/raw/master/assets/logo.png" height="55px" align="right">

# ⚛️ react-declarative-skills

> **Claude Code skill for the `react-declarative` TypeScript/React library** — AI-assisted form generation, data grid setup, routing, architecture guidance, and full API reference in one place.

## 🚀 Installation

The skill is installed under `~/.claude/skills/react-declarative/` and is automatically picked up by Claude Code.

```bash
npx skills add https://github.com/react-declarative/react-declarative-skills
```

## 🤖 Claude Code Skill

The `skills/react-declarative/` folder is a Claude Code agent skill that provides:

- **Form generation** — complete `TypedField[]` schemas for any form layout, validation, conditional logic
- **Data grid setup** — `ListTyped` with columns, filters, pagination, row actions, chips
- **Architecture guidance** — full project structure with DI, services, routing, modals
- **API reference** — all field types, layout containers, hooks, routing, reactive subjects, slot factory
- **Real-world examples** — 17 form schemas covering common UI patterns

### Skill structure

```
skills/react-declarative/
├── SKILL.md                          # Core concepts, quick decision guide, key rules
├── evals/
│   └── evals.json                    # 15 test prompts with expected outputs
└── references/
    ├── installation.md               # Install, peer deps, tsconfig, first form/grid
    ├── one-form.md                   # <One /> props, TypedField schema, apiRef
    ├── state-management.md           # handler, onChange, payload, isInvalid, dirty
    ├── list-grid.md                  # <List /> props, IColumn, ColumnType, actions, chips
    ├── field-types.md                # All FieldType values with examples
    ├── layout-types.md               # All layout containers (Group, Paper, Tabs, Condition…)
    ├── components.md                 # Scaffold2, KanbanView, WizardView
    ├── hooks.md                      # Async hooks, navigation hooks, useCollection
    ├── reactive.md                   # Subject, BehaviorSubject, Source pipelines
    ├── routing.md                    # Switch router, ISwitchItem, guards, redirects
    ├── dependency-injection.md       # provide/inject, TYPES symbols, scoped containers
    ├── slot-factory.md               # OneSlotFactory, ListSlotFactory, slot interfaces
    ├── advanced.md                   # AI-assisted form generation, playground
    ├── guide-async-data.md           # Choosing between handler / useSinglerunAction / useAsyncValue
    ├── guide-conditional-fields.md   # isVisible vs Condition vs hidden, RBAC patterns
    ├── guide-custom-fields.md        # FieldType.Component write-back, OneSlotFactory
    ├── guide-form-validation.md      # isInvalid, cross-field rules, async validation
    ├── guide-routing.md              # Switch setup, guards, prefetch/unload, OutletView
    ├── guide-architecture.md         # Full project architecture, DI wiring, service layers
    ├── login_form_example.md
    ├── account_info_example.md
    ├── profile_card_form_example.md
    ├── settings_page_form_example.md
    ├── adaptive_form_example.md
    ├── variant_form_example.md
    ├── order_info_form_example.md
    ├── product_shape_form_example.md
    ├── rate_card_form_example.md
    ├── crypto_form_example.md
    ├── machine_learning_form_example.md
    ├── dashboard_form_example.md
    ├── kpi_review_example.md
    ├── google_forms_like_example.md
    ├── custom_form_example.md
    ├── gallery_of_controls_example.md
    └── typography_example.md
```



## 📦 Library Installation

```bash
npm install --save react-declarative tss-react @mui/material @emotion/react @emotion/styled
```

Requires **MUI v5** (`@mui/material ^5.5.0`). Not compatible with MUI v4 or v6.

Lite variant (forms only, smaller footprint):

```bash
npm install --save react-declarative-lite
```



## 🧩 What the skill helps you build

| Component | Description |
|--|-|
| [`<One />`](https://github.com/react-declarative/react-declarative) | Schema-driven forms — no `useState`/`useEffect` wiring |
| [`<List />`](https://github.com/react-declarative/react-declarative) | Paginated data grids with filters, chips, row actions |
| [`<Scaffold2 />`](https://github.com/react-declarative/react-declarative) | App shell with sidebar navigation and top bar |
| [`<KanbanView />`](https://github.com/react-declarative/react-declarative) | Drag-and-drop kanban board with real-time support |
| [`<WizardView />`](https://github.com/react-declarative/react-declarative) | Multi-step wizard with MUI Stepper |
| [`<Switch />`](https://github.com/react-declarative/react-declarative) | Client-side router with guards, prefetch, redirects |

<sub>MIT © <a href="https://github.com/tripolskypetr">tripolskypetr</a></sub>
