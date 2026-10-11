# SpamDetectorBot

A lightweight, rule-based spam detector for incoming project bids. The application reads bid records from a CSV file, checks each bid against configurable text rules, and writes the analysis results to a JSON log.

> **Current scope:** This project uses simple rules, not a trained machine-learning model. Its output is a heuristic flag and should not be treated as a definitive spam verdict.

## Features

- **Repeated-phrase detection** — flags a bid when a sentence or phrase separated by `. `, `!`, or `?` appears more than once.
- **Excessive-link detection** — flags text containing more than three HTTP/HTTPS links.
- **Duplicate-bid detection** — flags text that exactly matches a bid processed earlier during the current run.
- **Configurable rules and polling interval** — settings are stored in `config.json`.
- **JSON output** — writes each scan's results to `logs/spam_log.json`.

## Project structure

```text
SpamDetectorBot/
├── config.json
├── incoming_bids.csv
├── requirements.txt
├── run.py
├── logs/
│   └── spam_log.json
└── spam_filter/
    ├── __init__.py
    ├── detector.py
    └── rules.py
```

## Requirements

- Python 3
- The packages listed in `requirements.txt`

The current implementation directly imports **pandas**. The requirements file also lists Streamlit, spaCy, and scikit-learn, although they are not currently used by the code in this repository.

## Setup

1. Clone the repository and enter its directory:

   ```bash
   git clone https://github.com/AMG-ir/SpamDetectorBot.git
   cd SpamDetectorBot
   ```

2. (Recommended) Create and activate a virtual environment:

   ```bash
   python -m venv .venv
   ```

   On Linux/macOS:

   ```bash
   source .venv/bin/activate
   ```

   On Windows PowerShell:

   ```powershell
   .venv\Scripts\Activate.ps1
   ```

3. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

4. Run the detector from the repository root:

   ```bash
   python run.py
   ```

The program reads `incoming_bids.csv`, analyzes its records, writes the results to `logs/spam_log.json`, and repeats after the configured polling interval.

## Input format

The input CSV must contain these columns:

| Column | Description |
| --- | --- |
| `id` | Identifier for the bid |
| `bid_text` | Text to analyze |
| `bidder_name` | Name or label associated with the bid |

A small sample dataset is included in `incoming_bids.csv`. Replace it with your own appropriately formatted records to test other cases.

## Configuration

Edit `config.json` to change the detector settings:

| Setting | Purpose | Default |
| --- | --- | --- |
| `rules.copy_paste` | Enable repeated-sentence detection | `true` |
| `rules.excessive_links` | Enable excessive-link detection | `true` |
| `rules.duplicate_text` | Enable exact duplicate-text detection | `true` |
| `poll_interval_seconds` | Delay between scans | `10` |
| `input_file` | CSV input path | `incoming_bids.csv` |
| `log_file` | JSON output path | `logs/spam_log.json` |
| `confidence_threshold` | Reserved/configured threshold; not currently applied by the detector | `0.7` |

## Output

Each processed bid produces a JSON object with fields such as:

- `id` — bid identifier
- `bidder_name` — bidder label
- `bid_text` — original text
- `spam` — whether any enabled rule was triggered
- `triggered_rules` — list of rules that matched
- `confidence` — a heuristic value calculated from the number of triggered rules

Example shape:

```json
{
  "id": 7,
  "bidder_name": "Example",
  "bid_text": "Example bid text",
  "spam": true,
  "triggered_rules": ["excessive_links"],
  "confidence": 0.8
}
```

The example illustrates the output format; the actual values depend on the input and rules that match.

## Limitations

- This is a **rule-based prototype**, not an ML classifier; it does not train a model.
- The current `confidence` value is a simple heuristic, not a calibrated probability. It starts at `0.7` even when no rule matches, so it should not be interpreted as the likelihood that a bid is spam.
- `confidence_threshold` is present in the configuration but is not currently used to make decisions.
- Duplicate detection is exact-match only and remembers processed text only for the current process. It is not a persistent history or fuzzy similarity check.
- The repeated-phrase rule uses basic punctuation splitting and may miss other forms of repetition.
- Each polling cycle reads the CSV and replaces the JSON log with the latest cycle's results; it does not maintain a historical log.
- The program expects each row to contain the required columns and valid text.

## Possible next steps

- Apply the configured confidence threshold or remove it until implemented.
- Improve input validation and error handling for malformed CSV rows.
- Add persistent duplicate tracking and timestamped scan history.
- Add automated tests for each detection rule.
- Evaluate false positives and false negatives against a labelled test dataset before relying on the detector.

## License

No license file is currently present in the repository. Add a `LICENSE` file if you want to specify how others may use, modify, and distribute this project.
