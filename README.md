# Bias in Debate

A Python data pipeline developed for AP Research to investigate judge tendencies and racial bias in high-school debate. It collects Tabroom tournament records and judge paradigms, links judges and debaters, and builds structured datasets for statistical analysis.

## What the pipeline does

| Component | Purpose |
| --- | --- |
| Judge and debater collection | Gather judging histories, written judging preferences, and competitor records |
| SQLite storage | Maintain structured records across collection and processing stages |
| Round compilation and matching | Build high-school round datasets and connect debaters to judge records |
| Coverage summaries | Report tournament and event coverage through CSV outputs |
| Paradigm classification | Use a local Ollama model to classify written judging preferences as lay or technical, with validated JSON output |
| Identity preparation | Filter relevant debate events, enrich available names, and optionally estimate demographic probability vectors |

The demographic estimates are name-based model outputs, not self-reported or verified identities. Any downstream study needs to account for their measurement error, data coverage, and possible confounding.

## Engineering details

- Progress tracking, retries, and resumable collection for long-running jobs.
- SQLite-backed storage with separate judge, debater, and round-processing stages.
- Graceful-stop support through signals and stop files.
- Configurable request pacing and bounded parallel classification.
- Environment-based Tabroom credentials; the optional classifier uses a local Ollama endpoint.

## Outputs

Runtime data is written under `output/`, including SQLite databases, progress/failure records, and research summaries. The [tournament summary module](src/tournament_stats.py) produces:

- `results/tournaments_by_hs_round_count.csv`
- `results/events_by_tournament_count_hs.csv`

These report the coverage of the collected data. The tournament CSV applies a minimum high-school round-count threshold defined in the module; its rows should not be treated as a count of every tournament collected.

## Setup and use

Install the Python dependencies in a virtual environment:

```bash
python -m pip install -r requirements.txt
```

Review the stage flags and paths in [main.py](main.py) before running. The checked-in configuration enables later enrichment stages and assumes earlier datasets already exist; a fresh run requires selecting the collection stages first.

- Supply `DEBATE_EMAIL_USER` and `DEBATE_EMAIL_PASS` as environment variables for stages that authenticate with Tabroom.
- If enabling judge-paradigm classification, run a local Ollama service with the model configured in `JUDGE_PARADIGM_MODEL_NAME` (currently `qwen2.5:7b`).
- Select the relevant `RUN_*` flags, input IDs, and output paths, then run from the repository root:

```bash
python main.py
```

## Code guide

- [judge_scraper.py](src/judge_scraper.py) and [debater_scraper.py](src/debater_scraper.py): data collection.
- [storage.py](src/storage.py): database operations.
- [rounds_compiler.py](src/rounds_compiler.py) and [judge_debater_matcher.py](src/judge_debater_matcher.py): round preparation and matching.
- [judge_paradigm_classifier.py](src/judge_paradigm_classifier.py): local-model classification.
- [tournament_stats.py](src/tournament_stats.py): coverage summaries.

## Status

Ongoing research tooling. Collection, matching, and model-generated classifications need validation before being used to draw conclusions about judge behavior.
