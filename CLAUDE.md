# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single-file Korean-language shopping list web app. There is no build system, no package manager, and no test suite — the entire application is `index.html`.

## Development

Open `index.html` directly in a browser. No server, build step, or install is required.

There are no lint, test, or build commands.

## Architecture

All HTML, CSS, and JavaScript live together in `index.html`. The app connects to a hosted Supabase project via the CDN-loaded `@supabase/supabase-js@2` client.

### Supabase backend

- **URL / anon key** are hardcoded at the top of the `<script>` block. The anon key is a public key scoped to anonymous access, not a secret.
- **Table**: `shopping_items` with columns `id`, `name` (text), `checked` (boolean), `created_at` (timestamptz). Items are fetched ordered by `created_at` descending (newest first).

### State model

A module-level `items` array is the single source of truth in memory. Every async operation (add, toggle, delete, clearChecked) mutates this array and then calls `render()` to rebuild the DOM. There is no reactive framework — `render()` performs a full `innerHTML` replacement of the list on each call.

### Key functions

| Function | Purpose |
|---|---|
| `loadItems()` | Fetches all rows from Supabase on page load |
| `addItem()` | Inserts a new row, prepends to `items`, re-renders |
| `toggleItem(id)` | Flips `checked` on one row in Supabase and in `items` |
| `deleteItem(id)` | Deletes a single row by `id` |
| `clearChecked()` | Bulk-deletes all rows where `checked = true` |
| `render()` | Rebuilds the list DOM and stats from `items`; called after every mutation |
| `escapeHtml(str)` | Manual XSS sanitisation applied to item names before injecting into `innerHTML` |

### Error handling

Errors from Supabase are surfaced via `showError(msg)`, which displays a dismissing banner for 4 seconds. During add operations `setLoading(true/false)` disables the input and button to prevent duplicate submissions.
