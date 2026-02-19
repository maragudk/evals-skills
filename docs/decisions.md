# Project Decisions

This document records significant architectural and design decisions made throughout the project's development.

## 2026-02-19: Use FastHTML + TailwindCSS + HTMX for trace-annotation-tool

The trace-annotation-tool skill generates a custom Python web app for open coding of LLM traces. We chose FastHTML as the web framework, with TailwindCSS (via CDN) for styling and HTMX for interactivity.

Alternatives considered:
- **Self-contained HTML/JS file**: Good portability, but clunky for file I/O and persistence. The tool needs to read trace data from disk and auto-save annotations, which is awkward without a server.
- **marimo notebook**: Reactive and already in the toolchain, but not flexible enough for a custom annotation UI with keyboard shortcuts, collapsible trace sections, and tailored rendering.
- **Streamlit**: Purpose-built for data apps, but similarly constrained in UI flexibility.
- **Flask + Jinja + HTMX**: Proven and boring, but more boilerplate. Requires wiring up Jinja templates, static files, and HTMX manually.
- **FastAPI**: Modern, async, but same boilerplate issue as Flask for this use case.

Tradeoffs: FastHTML is younger (2024, Answer.ai) and has a smaller community than Flask/FastAPI. However, it was purpose-built for HTMX-first server-rendered apps, which is exactly what the annotation tool is. Its Python component model for HTML generation is conceptually similar to gomponents. The generated code is simpler and more idiomatic for the HTMX pattern compared to Flask+Jinja. Since this is a local dev tool (not production infrastructure), the risk of a younger framework is acceptable.

Decision: Use FastHTML for the generated annotation tool. TailwindCSS via CDN keeps styling dependency-free. Vanilla JS only for keyboard shortcut bindings.
