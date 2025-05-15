# Clinical Note Review Tool

This is a lightweight Python script designed to manually review clinical notes related to a medical condition, using simple keyboard inputs.

It intakes rows of a CSV, shows note by note text with highlighting of key words and allows you to record positive/negative/maybe status quickly for each patient encounter.

---

##  Purpose

The tool is part of a digital phenotyping pipeline to identify patients presenting with a particular condition. It enables a clinician or reviewer to manually classify patient encounters based on review of their notes.

## Features

- **Interactive Review**: Presents notes one-by-one for manual classification (`1`, `0`, `?`).
- **Skips Reviewed Cases**: Keeps a running log in `manual_review.csv` so work is not lost between sessions.
- **Filters Low-Value Notes**: Discharge summaries (`IP_NOTE_TYPE_C == 5`) are only shown after other note types.
- **Keyword Highlighting**: Automatically highlights relevant terms chosen by the clinician in yellow.
- **Prioritised Workflow**: Reviews CSNs with the highest `TOTAL_FLAGS` first.
- **Robust Exit and Resume**: Can quit and resume at any time without repeating reviewed cases.

---

## File Structure

- `notes review - 1 row per note.csv`: Input file with clinical notes (one per row) - you input your own.
- `manual_review.csv`: Output file automatically generated to track review status.
- `review_script.ipynb`: Jupyter Notebook containing the Python logic.

---

## Manual Classification Logic

| Key Pressed | Meaning                        | Logic                                                             |
|-------------|--------------------------------|-------------------------------------------------------------------|
| `1`         | Confirmed positive             | CSN marked positive; no further notes from that CSN shown        |
| `0`         | Negative                       | Continue to next note within CSN                                 |
| `?`         | Unsure                         | Continue to next note, final CSN label is `?` if no `1` seen     |

- If a CSN has only `0`s → it's marked as `0`.
- If it has any `?` and no `1` → it's marked as `?`.
- If it has a `1` → marked `1`, review ends for that CSN immediately.

---

## Development Process

This tool was developed iteratively with the assistance of ChatGPT (OpenAI) through a conversational process. Requirements were refined across multiple improvements, including:

- Avoiding discharge summaries unless necessary
- Highlighting relevant keywords for visual triage
- Sorting logic correction (`groupby().sort_values()` → incorrect pattern)
- Ensuring resumability and fault tolerance
- Human-centric design for quick manual triage

- chat is attached [Slightly censored chat](CensoredChat.md) with a few lines change just to remove some specifics of what I was working on

