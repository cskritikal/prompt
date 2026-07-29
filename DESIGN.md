---
name: SOC Agent Steering Console
description: Dark-mode high-contrast security operations center (SOC) interactive documentation & prompt studio
colors:
  primary: "#06b6d4"
  success: "#10b981"
  warning: "#f59e0b"
  danger: "#f43f5e"
  neutral-bg: "#0b0f17"
  surface-bg: "#111827"
  card-bg: "#161e2e"
  border-subtle: "#1f293d"
  text-primary: "#f3f4f6"
  text-muted: "#9ca3af"
typography:
  display:
    fontFamily: "Fira Code, JetBrains Mono, monospace"
    fontSize: "clamp(1.5rem, 3vw, 2.25rem)"
    fontWeight: 700
    lineHeight: "1.2"
  body:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: "14px"
    fontWeight: 400
    lineHeight: "1.5"
  code:
    fontFamily: "Fira Code, JetBrains Mono, monospace"
    fontSize: "13px"
    lineHeight: "1.6"
rounded:
  sm: "4px"
  md: "6px"
  lg: "10px"
spacing:
  xs: "4px"
  sm: "8px"
  md: "16px"
  lg: "24px"
  xl: "32px"
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "#ffffff"
    rounded: "{rounded.sm}"
    padding: "8px 16px"
  button-secondary:
    backgroundColor: "{colors.card-bg}"
    textColor: "{colors.text-primary}"
    rounded: "{rounded.sm}"
    padding: "8px 16px"
---

# Design System: SOC Agent Steering Console

## Overview

**Creative North Star: "The Tactical Command Bridge"**

A high-density, mission-critical security interface modeled after modern Security Operations Centers (SOC) and cyber-threat monitoring consoles. Built specifically for triage efficiency, zero eye strain in dark control rooms, and ultra-fast snippet copy/injection.

Key Characteristics:
- Obsidian-black and slate surfaces with high-contrast indicator LED badges
- Monospace font dominance for code, log snippets, and regex rules
- Split-pane layout with category sidebar, snippet inspector, and live interactive playground
- Instant filter, search, defang toggle, and JSON/YAML export controls

## Colors

The color palette uses deep dark tones with distinct security indicator accents.

### Primary & Status Accent
- **Active Steer Cyan** (`#06b6d4`): Used for primary interactive actions, selected tabs, and active steering parameters.
- **Suppressed Emerald** (`#10b981`): Status indicator for Route 2 suppressions, clean checks, and verified benign rules.
- **Miscalibrated Amber** (`#f59e0b`): Warning indicators for priority inflation and analyst review required.
- **Critical Escalation Rose** (`#f43f5e`): High/Critical escalation markers, defanged threat indicators, and rule violation warnings.

### Neutral
- **Deep Obsidian** (`#0b0f17`): Base background.
- **Dark Slate Surface** (`#111827`): Sidebar and panel headers.
- **Elevated Card BG** (`#161e2e`): Code blocks, snippet cards, and playground input panels.
- **Tactile Edge Border** (`#1f293d`): Subtle grid lines and card borders.

### Named Rules
**The Indicator Rarity Rule.** Neon accent colors (cyan/emerald/rose) are strictly reserved for operational status badges, active buttons, and copy triggers (<10% visual surface).

## Typography

**Display & Code Font:** `Fira Code, JetBrains Mono, SF Mono, monospace`
**Body Font:** `Inter, system-ui, sans-serif`

### Hierarchy
- **Display** (700, clamp(1.5rem, 3vw, 2.25rem), 1.2): Section titles and application header.
- **Headline** (600, 1.125rem, 1.3): Snippet card headings and category labels.
- **Body** (400, 14px, 1.5): Snippet descriptions, rule rationale, and user guidance.
- **Code & Snippet** (400, 13px, 1.6): Raw markdown prompt snippets, JSON schemas, and log telemetry.

## Layout

- **Split-Pane Grid**: 260px fixed left category sidebar + flexible central documentation reader + 400px collateral interactive playground drawer.
- **Compact Density**: Tight vertical padding (8px - 16px) prioritizing information density and scannability.

## Elevation & Depth

Flat surfaces with subtle 1px crisp borders (`#1f293d`). Hover states utilize a subtle cyan outline glow (`0 0 12px rgba(6, 182, 212, 0.25)`).

## Shapes

- **Radius**: Tight, modern 4px - 6px corners for buttons, inputs, and code containers.

## Components

### Buttons
- **Primary Action**: Cyan background, white monospace text, 4px radius.
- **Secondary Ghost**: Slate card background, subtle border, hover cyan glow.

### Code Blocks & Prompt Cards
- Monospace font, `#0b0f17` background with 1px `#1f293d` border, equipped with top-right quick-copy and variable-tuning controls.

## Do's and Don'ts

### Do:
- **Do** maintain high contrast between text (`#f3f4f6`) and deep dark background (`#0b0f17`).
- **Do** format all steering prompt snippets in styled monospace blocks with line numbers and one-click copy.
- **Do** defang all IOC examples (e.g. `hxxp[:]//`, `192[.]168[.]1[.]1`).

### Don't:
- **Don't** use light mode or generic white cards.
- **Don't** use rounded pill buttons or overly soft pastel colors.
