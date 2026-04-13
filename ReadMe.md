# PetLog — Pet Health Tracker

> **Read every file in `docs/` before writing any code.** These documents are your single source of truth.

## What to Build

A single-page React app (Vite + TypeScript + Tailwind) that lets pet owners track their pets' health records — vaccinations, vet visits, weight, and medications.

## Technical Requirements

- **Vite + React 18 + TypeScript + Tailwind CSS**
- All data stored in React state (no backend, no localStorage)
- Pre-populate state with the data from `docs/sample-data.json` on load
- All user-facing strings must come from `docs/content.json`
- All components must match `docs/design.md` exactly
- Every screen described in `docs/screens.md` must be implemented

## Process

1. Read all 4 docs
2. Scaffold the Vite project
3. The current file contents is for the wrong topic, the actual applicaiton we need built is about human health tracker. Use the current design but delete all data related to animals, the user is using this structure but wants the project refactored for human health tracking.
4. Configure Tailwind with the design tokens from `docs/design.md`
5. Build all components
6. Implement all screens from `docs/screens.md`
7. Wire in sample data
8. Run `npm run build` — must succeed with zero errors
9. Verify every screen matches the spec
10. Push the changes to github to save them for the user.
