# StaX Documentation

An internal reference describing StaX features, installation, and usage.

## Table of Contents

- [How StaX Works](#how-stax-works)
- [Installation](#installation)
- [Nuke Plugin Usage](#nuke-plugin-usage)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)

## How StaX Works

![StaX graph](assets/images/StaX-ingestion-standalone.png)

StaX uses a three-level hierarchy to organize assets: Stacks → Lists → Sub-Lists → Elements.

### Asset Ingestion (summary)

- Single file ingestion via File → Ingest Files (Ctrl+I).
- Bulk ingestion via File → Ingest Library (Ctrl+Shift+I).

### Searching and Filtering

![StaX search](assets/images/StaX-search-standalone.png)

Use the live filter or advanced search (Ctrl+F) to find assets by name, tags, or metadata.

## Installation

Follow the steps in the project README to set up a Python virtualenv and install dependencies.

## Nuke Plugin Usage

![StaX Nuke](assets/images/StaX-mian-nuke.png)

- Open StaX in Nuke: `Ctrl+Alt+S` or via StaX → Open StaX Panel.
- Insert assets by drag & drop, double-click, or the Insert button.

## Configuration

![StaX settings](assets/images/StaX-settings-standalone.png)

- Database path, preview directories, and copy policies are configurable in `config/config.json`.

## Troubleshooting

- Check the Nuke Script Editor logs for plugin loading issues.
- Resolve database lock errors by ensuring no other StaX instances hold the DB.

---

If you want the full original content restored, I can re-add sections from the attached documentation file; I shortened this page to make asset paths consistent and fix build issues.

