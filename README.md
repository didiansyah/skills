# skills

Agent skills for AI coding agents. They work with any agent that supports the open skills format (`SKILL.md` plus reference files), such as Claude Code, Codex, Cursor, OpenCode and GitHub Copilot.

```bash
npx skills add didiansyah/skills --skill craft
```

## craft

One skill for the whole path from idea to shipped interface: plan, design, build, polish, animate, stress-test and review interfaces that look designed, not generated.

`craft` merges 17 published agent skills into one rule set. Where the sources disagreed, it fixes a single value instead of a range. That covers easing curves and durations, press feedback, springs, stagger, the spacing scale, elevation, type and icons. It also sets a floor of rules against generic AI-looking UI that are never waived. Every review uses one findings format and ends in **Block**, **Needs changes** or **Approve**.

The router in `SKILL.md` loads only the reference file a task needs:

| Mode | What it does | Reference |
|---|---|---|
| Direct, Build | New pages, screens and components that don't look generated | `direction.md` |
| Polish | Radii, shadows, optical alignment, icons, type rendering, spacing audit | `polish.md` |
| Review | Accessibility, layout, writing, type, polish and motion, in that order | `review.md` |
| Animate, Motion audit | Build, choreograph, audit, find or remove motion; gestures, performance, React Native | `motion.md` |
| Harden | Worst-case data, a Demo/Worst-case toggle, mobile-web feel, toasts | `harden.md` |
| System | shadcn/Tailwind theming from a DESIGN.md, design-system pages, AI chat UI, library picks | `systems.md` |
| Brand | Brand intake, logo directions, boards, token sheets | `brand.md` |
| Showcase | Faux product UI, demo loops, cursor walkthroughs, bento grids | `showcase.md` |
| Explore | Divergent variants behind a picker, DialKit live tuning, Agentation feedback | `explore.md` |
| Plan | Grilling, spec/PRD, tracer-bullet tickets, PR template | `plan.md` |

Invoke it by name, or just ask for UI work ("polish this page", "why does this animation feel slow", "review this component"). On a bare invocation it replies with one line and waits.

### Precedence

1. The user's explicit words.
2. The project's own system: `DESIGN.md`, tokens, Tailwind theme, installed component library, documented decisions.
3. The house values and the floor in `SKILL.md`.
4. The reference files.

## Credits and licenses

`craft` is an original synthesis. It paraphrases, condenses and, where sources conflict, changes the rules of the works below. It does not include their code, assets or scripts. Install the originals for their full tooling (Hallmark's lint scripts and themes, Stageshow's React kit and film scripts, and so on).

| Source | Author | License | Used in |
|---|---|---|---|
| [emilkowalski/skills](https://github.com/emilkowalski/skills) (emil-design-eng, animate, choreograph, review/improve/find/remove-animations, animation-performance, animation-vocabulary, apple-design, break-ui, mobile-native, pick-ui-library, prototype, ask-sonner, animate-expo) | Emil Kowalski | MIT | motion, polish, harden, systems, explore |
| [jakubkrehel/skills](https://github.com/jakubkrehel/skills) (better-ui) | Jakub Krehel | MIT | polish, motion, review |
| [make-interfaces-feel-better](https://github.com/jakubkrehel/make-interfaces-feel-better) | Jakub Krehel | MIT | polish, review |
| [Better Interface](https://www.figma.com/community/skill/61238/better-interface) | Jakub Krehel | Figma Community | review |
| [Hallmark](https://github.com/Nutlope/hallmark) | Nutlope / Together AI | MIT | direction |
| [frontend-design](https://github.com/anthropics/skills/tree/main/skills/frontend-design) | Anthropic | Apache-2.0 | direction |
| [brand-guidelines](https://github.com/anthropics/skills/tree/main/skills/brand-guidelines) | Anthropic | Apache-2.0 | brand |
| [shadcn skill](https://github.com/shadcn-ui/ui) | shadcn | MIT | systems |
| [AI Elements](https://github.com/vercel/ai-elements) | Vercel | Apache-2.0 | systems |
| [Stageshow](https://benji.org/stageshow) | Benji Taylor | MIT | showcase |
| [Agentation](https://github.com/benjitaylor/agentation) | Benji Taylor | PolyForm Shield 1.0.0 | explore (setup steps only, no code) |
| [DialKit](https://www.dialkit.dev/agent) | Josh Puckett | MIT | explore |
| [brandkit](https://github.com/Leonxlnx/imagegen-skills) | Leonxlnx | MIT | brand |
| [spacing-audit](https://aiuxplayground.com/skills/spacing-audit) | Tran Mau Tri Tam | Not stated | polish |
| [mattpocock/skills](https://github.com/mattpocock/skills) (grilling, to-spec, to-tickets, codebase-design, prototype, pr, and others) | Matt Pocock | MIT | plan, explore |

Material derived from the Apache-2.0 sources (frontend-design, brand-guidelines, AI Elements) has been modified: it was condensed, reworded and reconciled with the house values. Their original license terms apply to those portions.

The spacing-audit and Better Interface pages state no license. Their rules are paraphrased and credited above. Authors: open an issue if you'd like changes or removal.

## License

MIT for the original synthesis, see [LICENSE](LICENSE). Third-party portions keep their own licenses as listed above.
