# AGENTS.md

Instructions for AI coding agents working on this project.

## Project Overview

GitLess is a minimal VSCode extension for Git integration. It provides a dedicated Source Control panel section with views (Commits, Branches, Remotes, Stashes, Tags, Worktrees), an Inspect panel (File History, Line History, Search & Compare), and command palette commands for copying remote URLs and commit SHAs.

## Key Conventions

- Use pnpm for package management (not npm or yarn).
- Use Conventional Commits and Angular commit message conventions for all commit messages (e.g. `feat:`, `fix:`, `docs:`).
- Use Prettier for formatting with `"semi": false` (no semicolons).
- Use double quotes for strings.
- Do not create Git commits on `main`; always work on a feature branch.
- Do not Git push to `main`; always push to a feature branch.

## Architecture

- **Entry point**: `src/extension.ts` - `activate()` creates a `Container` instance.
- **DI container**: `src/container.ts` - Wires up the `GitService`, commands, views, and configuration.
- **Git layer**: `src/git/` - `models.ts` (interfaces), `shell.ts` (subprocess execution), `parsers.ts` (output parsing), `remoteUrls.ts` (URL generation), `gitService.ts` (high-level API).
- **Commands**: `src/commands/registerCommands.ts` - All command handlers registered in one function. Command IDs defined in `src/constants.ts`.
- **Views**: `src/views/` - `groupedView.ts` owns a single `TreeView` in the SCM panel and delegates to sub-view `TreeDataProvider`s (one per file). Tree node classes are in `src/views/nodes.ts`.
- **Configuration**: `src/config.ts` - Reads `gitless.*` settings. `src/constants.ts` has all command/view/context IDs.

## Adding Features

When adding a new command:

1. Add to `Commands` in `src/constants.ts`
2. Add handler in `src/commands/registerCommands.ts`
3. Add to `contributes.commands` in `package.json`
4. Add menu entries in `package.json` if needed

When adding a new view:

1. Add to `ViewIds` in `src/constants.ts`
2. Create view class in `src/views/`
3. Export from `src/views/index.ts`
4. Instantiate in `src/container.ts`
5. Declare in `package.json` under `contributes.views`

## Build and Test

```sh
pnpm install           # Install dependencies
pnpm run build         # Production build (esbuild → dist/extension.js)
pnpm run build:dev     # Dev build with sourcemaps
pnpm run lint          # Prettier check
pnpm run format        # Prettier write
pnpm run test          # Unit tests (Mocha in VS Code test host)
pnpm run test:e2e      # E2E tests (Playwright)
```

## Important Files

- `package.json` - Extension manifest. All commands, views, menus, and configuration are declared here.
- `src/constants.ts` - Single source of truth for command IDs, view IDs, and context value strings.
- `src/git/parsers.ts` - Git output parsing. Changes here should be accompanied by unit tests in `src/__tests__/parsers.test.ts`.
- `src/git/remoteUrls.ts` - Remote URL generation for GitHub, GitLab, Bitbucket, Azure DevOps, and Gitea. Tests in `src/__tests__/remoteUrls.test.ts`.

## Testing Guidelines

- Unit tests are in `src/__tests__/` using Mocha `tdd` UI (`suite`/`test`, not `describe`/`it`).
- Prefer testing pure functions (parsers, URL generation) over VSCode API integration.
- Run `pnpm run test` to execute unit tests; these run inside the VS Code test host.
- Run `pnpm run lint && pnpm run build` to verify before committing.
