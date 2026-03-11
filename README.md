# ADSP — AI Development State Protocol

![protocol](https://img.shields.io/badge/protocol-ADSP-blue)
![version](https://img.shields.io/badge/version-v0.1-green)
![status](https://img.shields.io/badge/status-experimental-orange)
![AI-Native](https://img.shields.io/badge/AI--Native-Development-purple)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

> A lightweight protocol for coordinating **AI and human developers working in parallel inside the same repository**.

---

# Why This Exists

AI can write code.

But **multiple AI agents modifying the same repository simultaneously** creates chaos:

- migrations collide
- APIs change unexpectedly
- changes overwrite each other
- CI pipelines fail
- agents lose track of project state

Traditional workflows (branches, PRs, tickets) were designed for **humans coordinating socially**.

AI agents need something different:

**Machine-readable coordination.**

ADSP introduces a simple concept:

> The repository stores its own **development state** so AI agents can behave coherently.

---

# The Core Idea

Instead of only storing code…
