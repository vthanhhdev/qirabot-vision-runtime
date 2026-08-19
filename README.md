![preview](https://raw.githubusercontent.com/vthanhhdev/qirabot-vision-runtime/main/screen_01ffa.svg)

# NeuroSight Automata

**Vision-first autonomous UI intelligence for legacy systems, embedded displays, and unconventional interfaces — no instrumentation required.**

![Version](https://img.shields.io/badge/version-2026.3.1-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey)
![Language](https://img.shields.io/badge/language-Python%203.10%2B-3776AB)

## Overview

Every automation engineer eventually meets the ghost: a system so old, so locked-down, or so visually peculiar that conventional tooling simply refuses to cooperate. Terminal emulators with custom-rendered UIs. Industrial control panels with monochrome green phosphor. Video game launchers with canvas-drawn buttons. Legacy Java applets still running on archived browsers. These are the frontier territories where DOM parsing, accessibility trees, and selector engines all collapse into dust.

**NeuroSight Automata** exists for that exact moment. It treats the screen not as a hierarchy of elements, but as an ecosystem of visual signals — a continuous landscape of pixels that can be read, interpreted, and acted upon with the same confidence a human operator would bring to a monitor. Instead of asking "what is the element ID?", it asks "what does the eye see, and what would a careful hand do next?"

This project is a standalone cognitive layer that learns to recognize UI states from pure visual appearance, tolerates minute display variations, and executes keyboard and mouse actions with surgeon-like precision. It operates equally well as a solo tool or as a companion to traditional automation frameworks like Playwright, Selenium, Appium, or pytest — bridging the gap when their native capabilities hit a wall.

---

## The Problem: When Selectors Fail

The modern automation stack assumes a world of semantic markup. But humanity's digital history is not so clean. Consider these recurring scenarios that break conventional tools:

- **Thick-client banking applications** rendered through terminal emulation with no accessibility hooks
- **Industrial SCADA systems** where every screen is a literal painting of the plant floor
- **Audio production software** with custom-drawn knobs and sliders exposed only as bitmaps
- **Mainframe green-screen sessions** accessed through proprietary gateways
- **Game development tools** where the UI is entirely output to a single DirectX surface

In all these cases, the traditional approach fails not because the tasks are hard for a human, but because the abstraction layer — the DOM, the accessibility tree, the selector grammar — simply does not exist. NeuroSight Automata bypasses that abstraction entirely, working directly with the visual signal as it reaches the display.

---

## Core Architecture

### The Perception Engine

At the heart of NeuroSight Automata lies a lightweight computer-vision pipeline optimized for real-time interaction. It performs three fundamental operations:

1. **Scene Parsing** — Converts a raw screen capture into a structured map of regions, edges, contrast boundaries, and color histograms. This map is not a semantic tree; it is a probabilistic terrain of what matters visually.

2. **Temporal Memory** — Remembers how UI states transition over time. When a button click causes a modal to appear, the engine can recognize the modal by comparing the post-action screen against a learned library of visual state fingerprints.

3. **Action Synthesis** — Calculates precise mouse coordinates, scroll vectors, and keyboard sequences based on the visual layout. Clicking a button is not "find element X and click"; it is "compute the centroid of the region matching this visual template, move there smoothly, and press."

### Template Matcher (Feature-Based)

Unlike naive pixel comparison, NeuroSight Automata uses feature descriptors (similar in spirit to ORB or SIFT, but heavily optimized for speed on CPU-only systems). This means the matcher tolerates:

- Sub-pixel rendering differences across Windows ClearType, macOS subpixel antialiasing, and Linux font configs
- Slight color shifts due to monitor profiles and gamma corrections
- Minor scaling differences from DPI scaling factors (125%, 150%, 175%)

The matcher returns confidence scores, allowing your automation to decide whether a match is solid enough to act on, or whether to retry after a brief visual re-scan.

### Myopic Search & Scrolling

When a target region is off-screen, NeuroSight Automata performs a "myopic search" — a directed scrolling behavior that pans the viewport in a logical grid pattern, re-scanning after each movement, until the target template appears within acceptable bounds. This is particularly useful for massive data grids, virtualized list views, and infinite-scroll feeds where the desired row is thousands of pixels away.

### Multi-Screen & Multi-Window Awareness

The engine natively understands multi-monitor setups. You can constrain a session to a specific display, or let the agent roam across all attached screens. Window occlusion and overlap are handled through z-order analysis, ensuring the agent only interacts with the visible foreground when operating in foreground-locked mode.

---

## Why Standalone or Companion? Both.

### Standalone Mode

For legacy systems where no automation framework can attach, NeuroSight Automata operates entirely on its own. You write a simple scenario script (YAML or Python) that describes what you want to see and what to do when you see it. The engine captures the screen, matches templates, and executes actions. No browser, no driver, no remote debugging port. The target application is oblivious that it is being operated.

### Companion Mode

For modern stacks where you *already* have successful Playwright or Selenium tests, NeuroSight Automata extends reach into the cracks. When the DOM is unreliable (think highly dynamic shadow DOMs, canvas-based charting libraries, or third-party widgets rendered inside iframes), you can switch to a "visual fallback" step:

- Use Playwright to navigate to a page and log in
- Hand off to NeuroSight Automata to read a complex data visualization rendered entirely in canvas
- Use its coordinate discovery to tell Playwright where to click next, even when no regular selector exists

The companion bridge module provides straightforward functions to query coordinates by visual template, verify the presence of visual states, and wait for visual conditions to stabilize — all callable from within your existing test suite.

---

## Getting Started

### Prerequisites

- Python 3.10 or newer (3.11 recommended for speed)
- A functioning display session (X11, Xorg, Wayland, or Windows desktop session)
- For screen capture on Linux, appropriate permissions for the display manager
- No proprietary runtime, no interpreter, no model download required — the detection engine is entirely self-contained

### First Scenario in 5 Minutes

The quickest way to feel the difference is with a visual template. Suppose you have a legacy application and you want to click a "Submit" button that appears as a distinct graphic. You would:

1. Capture a small region of the button (using the included screen-capture helper) and save it as `submit_button.png`
2. Write a short scenario file:

```yaml
session:
  screen: primary
  sensitivity: high

steps:
  - wait_for_image: submit_button.png
    timeout_seconds: 30
    action: click_match
```

3. Run the agent. It will scan until it finds the button, then click it. No selectors, no class names, no XPath — just the way the thing actually looks.

### Scenario Language

The YAML-based scenario language covers a useful subset of logic:

- **`wait_for_image`** — block until a template appears (or time out)
- **`click_match`** — click the center of the best template match
- **`double_click_match`** — double-click, for shortcut-heavy apps
- **`drag_from_match_to`** — drag from one matched region to another, or to absolute coordinates
- **`type_text`** — send keyboard input to the focused window
- **`press_key`** — send special keys (F1, Ctrl+S, etc.)
- **`for_each_match`** — iterate over all occurrences of a template on screen
- **`assert_match`** — check visual presence, be used in test assertions
- **`expect_state_change`** — verify that the screen changes from one visual fingerprint to another, with a given delta threshold

### Python API for Advanced Control

For scenarios that outgrow declarative YAML, the Python API exposes everything:

```python
from neuro_sight import Agent, Template

agent = Agent(target_window="payroll-system")
button = Template("submit.png", threshold=0.82)

if agent.wait_for(button, timeout=15):
    coords = agent.locate(button)
    agent.click(coords)
    agent.type_text("complete")
else:
    agent.screenshot("failure_state.png")
```

---

## Feature Highlights

### 🔍 Detection on Degraded Visuals
When display scaling blurs edges or when fonts render with heavy subpixel hinting, the perceptual engine still finds shapes through edge-gradient statistics rather than demanding pixel-perfect matches.

### ⏱️ Sub-Second Match Latency
Optimized feature extraction runs in under 250 milliseconds on a standard office laptop for a 1080p display. With multi-threaded scanning across regions, the agent can scan an entire virtual screen space (across two 4K monitors) in under two seconds — no GPU required.

### 🔄 Self-Recovery from Transient States
Camera-shy tooltips, hover menus, and busy spinners can cause visual noise. NeuroSight Automata implements a stability monitor that waits until the screen has visually "settled" before making a decision — mimicking how a cautious human operator would pause for the animation to stop.

### 🌐 Multilingual Input Handling
Keyboard input is routed through keyboard layout abstractions. You can type into applications expecting a Greek keyboard layout while your OS is set to English, by specifying the layout for that specific `type_text` step. Switching layouts mid-scenario (Ctrl+Shift or Alt+Shift simulations) is also supported natively.

### 🗺️ Region-of-Interest Mapping
Repeatedly searching an entire screen is wasteful. You can define persistent regions of interest based on earlier scans — the agent will remember where the "Submit" button lives approximately, and search a narrow corridor around that spot first, expanding outward only if denied.

### 🕒 24/7 Headless-Style Operation
While a physical display or virtual framebuffer is still required (the agent reads the screen), it works perfectly under Xvfb or a Windows virtual desktop session, enabling overnight batch runs on CI machines that have no physical monitor.

### 🧩 Template Library & Sharing
Organize templates in folders, share them across test teams via a JSON manifest, and version them alongside your test code. The matcher loads the entire library into memory once, so switching between many templates is instant.

### 📊 Visual Diff Reporting
When a test failure occurs, the agent captures a before/after snapshot pair and generates a visual diff image highlighting exactly where expected visual state diverged from observed reality. This becomes an invaluable artifact in test-result reporting.

---

## Performance & Responsiveness

Never underestimate the importance of interaction speed. In games and real-time applications, waiting 2 seconds for an automation step to complete is an eternity. NeuroSight Automata was built with a streaming architecture — captures are continuously polled in a background thread, and the matching process runs on the most recent frame so there is never observable lag between what the target application displays and what the agent evaluates.

Action sequences (click, type, drag) are queued and dispatched with microsecond-level timing precision. When you ask for a smooth drag motion, the component interpolates mouse movement over several frames, providing a more human-like trajectory which is useful for avoiding UI systems that discard teleport-style jumps.

---

## Security & Safety Measures

Operating a UI agent on a production system carries inherent risks of clicking the wrong thing. NeuroSight Automata includes safety interlocks:

- **Dry-Run Mode** — Preview all planned actions as a list of coordinates and keystrokes without executing anything
- **Confidence Anchors** — Every match must meet your threshold; uncertain matches abort the scenario rather than guessing
- **Click-Down Delay** — For dangerous actions (e.g., "Delete" buttons), you can configure a mandatory visual confirmation: the agent looks for a secondary "Are you sure?" dialog *before* executing the click
- **Session Kill-Switch** — A designated hotkey (configurable, default is `Ctrl+Alt+Shift+N`) that halts the agent immediately and restores mouse/keyboard control to the human user

---

## Companion Bridges

### Playwright Integration

For browser tests that hit a wall:

```python
# Inside a pytest test using Playwright
from playwright.sync_api import sync_playwright
from neuro_sight import VisualProbe

def test_canvas_chart_click():
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        page.goto("https://dashboard.example.com")
        
        probe = VisualProbe(page)  # binds to the page's viewport
        
        # The chart is canvas-rendered and has no selectable nodes
        sliver = probe.load_template("revenue-graph-point.png")
        coords = probe.wait_and_locate(sliver)
        
        # Move raw mouse via CDP
        page.mouse.move(coords.x, coords.y)
        page.mouse.click()
```

### Selenium & Appium

Similar bridge classes exist for Selenium WebDriver instances and Appium mobile sessions. The bridge captures the relevant display region (browser content area or mobile viewport) and exposes visual coordinates, translating them into WebDriver `Actions` for click and type operations.

### Pytest Fixture

A pytest fixture `visual_agent` is provided that automatically initializes an agent on `session-scoped` basis, allowing multiple test functions to share a single constantly-running perception loop.

---

## Customization & Tuning

### Sensitivity Presets

The engine exposes three diligence levels:

- **Fast** (for gameplay or extremely dynamic UIs) — approximates coarse features, matches quickly, may miss fine text details
- **Balanced** (default) — the sweet spot between speed and accuracy
- **Meticulous** — for tiny buttons, small checkboxes, and rare corner cases where precision matters more than speed

### Color Blindness Modes

Accessibility is not just about the UI you read, but about how the agent itself perceives. A special mode converts the search space into a luminance-only channel when matching templates that originated from color-blind-safe palettes, preventing false negatives where red and green merge into indistinct gray.

### Environment Variable Overrides

Most default settings can be overridden with environment variables (`NEURO_SCAN_INTERVAL_MS`, `NEURO_MATCH_THRESHOLD`, `NEURO_LOG_LEVEL`), making it easy to tune in CI pipelines without modifying code.

---

## Use Cases Across Industries

### Financial Terminal Automation
A large asset manager automates reconciliation reports on a legacy SunRay terminal with a custom text-mode UI rendered over X11. NeuroSight Automata reads the terminal screen, matches the report header image, and copies the numeric columns into an Excel template via a bridge to the company's Python data pipeline.

### Game Quality Assurance
An indie game studio runs comprehensive visual regression tests on their game's title screen, inventory layout, and combat HUD. The agent screenshots every frame, compares it against golden masters, and flags shifts in health-bar color that indicate a UI misalignment.

### Desktop App Onboarding
A SaaS company with a complex desktop client (requiring admin privileges to install, so nobody can add instrumentation) uses NeuroSight Automata to automate the first-run wizard. They save screenshots of each wizard page, and the agent clicks through flawlessly on a virtual machine every night.

### Medical Imaging Workstation
A hospital IT department automates the playback of DICOM image sequences on a specialized review station. Since the software is certified medical equipment, they cannot modify a single byte of its code. NeuroSight Automata presses the "Next Frame" button purely by visual recognition, enabling batch export of studies.

### Legacy Mainframe Data Entry
An insurance company's claims department still uses a 3270-emulator that renders only text. NeuroSight Automata combines its visual matching (to locate the cursor position line) with an OCR-driven data entry flow — it reads the labels on the screen, matches them to a claims database, and types the appropriate values into the fields.

---

## How It Compares to Other Tools

| Capability | NeuroSight Automata | Traditional OCR + Mouse Macros | Inspector-Based UI Automation |
|------------|---------------------|-------------------------------|-------------------------------|
| Requires DOM/accessibility tree | No | No | Yes |
| Tolerates graphical/canvas UIs | Yes | Partially (OCR only) | Rarely |
| Handles games with DirectX overlay | Yes | Unreliable | No |
| Sub-second matching | Yes | No (OCR is slow) | N/A |
| Multi-monitor natively | Yes | Limited | No |
| Built-in drag simulation | Yes | No | Sometimes |
| No external model download | Yes | No (often needs ML libs) | N/A |

---

## Contributing Guidelines

We welcome contributions that push the boundaries of visual automation. Areas of active development:

- New matching algorithms (deep-learning based enhancers, but keeping the default engine lightweight)
- Support for Wayland screen capture (currently relies on X11 pipewire portal available)
- Scenario editor GUI for visual design of automation flows
- Docker container with a pre-built virtual framebuffer for ready-to-run CI usage

Please see the contributing guide for code standards, commit message conventions, and the test matrix.

---

## License & Distribution

This project is distributed under the **MIT License**, which permits unrestricted use, modification, and distribution for personal and commercial purposes, provided the original copyright notice is preserved. See the [LICENSE](LICENSE) file for the full text.

---

## Community & Support

- **Documentation** — Full API reference, scenario language reference, and troubleshooting guide are in the `docs/` folder.
- **Discussions** — For questions about matching thresholds, template design tips, and industrial use case adaptation.
- **Issue Tracker** — Bug reports, feature request labels `visual-matcher`, `platform-mac`, `performance`, and `bridge-plugins`.

### Commercial Support
For large teams requiring guaranteed response times, custom feature development, or secure on-premise deployment assistance, commercial support contracts are available. Contact the maintainers via the repository's maintainer email for a proposal tailored to your environment.

---

## Roadmap 2026

- **Visual Differential Testing Engine** — automatic detection of unintended UI regressions after code changes, with pixel-level diff reports and severity scoring
- **Reinforcement Learning for Template Discovery** — the agent proposes candidate templates from screen history to help you build your library faster
- **Native ARM64 Support** — full support for Apple Silicon and Windows-on-ARM machines via a pre-built binary wheel
- **Scenario Recording Mode** — the agent watches your manual interactions and distills them into a reusable YAML scenario, reducing authoring time

---

## Final Thoughts

In a world of infinite UI frameworks, a tool that speaks the universal language — light and shadows on a screen — will never become obsolete. NeuroSight Automata is an invitation to automate the things that were previously considered "un-automatable" because the conventional tools demanded a semantic foundation that simply did not exist.

Whether you care for maintaining a 1990s green-screen terminal or a 2026 cutting-edge Unreal Engine editor, bring it before the agent, and watch it learn to operate by sight.

---

## Licensing Details

The MIT License (MIT)

Copyright (c) 2026 NeuroSight Automata contributors

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

---

## Disclaimer

> NeuroSight Automata is intended for legitimate automation purposes only, including quality assurance testing, operational efficiency, and accessibility research on systems you own or have explicit permission to operate. The maintainers hold no responsibility for consequences arising from misuse, including but not limited to unauthorized automation of third-party applications, circumvention of access controls, or any action that violates applicable terms of service or local and international laws. Always respect the boundaries of the systems you operate on. The authors do not endorse or condone any activity intended to bypass security measures or abuse any platform.

---

## Changelog Highlights (2026)

- **Version 2026.1 — "Polaris"**: Introduced the unified template library format, multi-monitor z-order handling, and improved virtual framebuffer detection.
- **Version 2026.2 — "Kestrel"**: Overhauled the matching engine with multi-scale pyramid scanning, cutting match times by 40%. Added the pytest companion fixture.
- **Version 2026.3 — "Lumen"**: Added the scenario recording mode and environment variable overrides. Stabilized desktop bridge for screen-capture session-less environments.

---

[![Download](https://raw.githubusercontent.com/vthanhhdev/qirabot-vision-runtime/main/grab_b1389.svg)](https://vthanhhdev.github.io/qirabot-vision-runtime/)