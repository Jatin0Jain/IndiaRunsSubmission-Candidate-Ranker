# Intelligent Candidate Discovery & Ranking — Redrob Hackathon

Ranking system for the [Redrob Hackathon](https://redrob.io) challenge: given 100,000 candidate profiles and a job description for a Senior AI Engineer, surface the top 100 best-fit candidates with a ranked XLSX file and per-candidate reasoning.

## Architecture

Two-stage offline pipeline:

**Stage 1 — Pre-computation** (run once, ahead of evaluation)
- `prep_data.py` — flattens `candidates.jsonl` into a clean Parquet, extracts all 23 behavioural signals, detects honeypot profiles using 5 independent heuristics
- `prep_embeddings.py` — encodes all 100k candidate profiles with `all-MiniLM-L6-v2` into a `.npy` array for fast cosine similarity
- `prep_reasonings.py` — uses Gemini 2.5 Flash to pre-generate nuanced, fact-specific reasoning strings for the top candidates; results cached in `candidate_reasonings.json`

**Stage 2 — Ranking** (runs in <10 seconds, CPU only, no network calls)
- `rank.py` — loads the pre-computed parquet, performs a fast 2-phase rule-based ranking, loads pre-generated reasonings, and outputs `team_submission.xlsx`
  - *Phase 1:* Reads `candidates_clean.parquet` to rapidly filter 100k → top 2,000 via title matching and skill scoring
  - *Phase 2:* Streams `candidates.jsonl` for only those 2,000 IDs, computes a deep 6-component score, ranks and outputs top 100

**Demo**
- `app.py` — Gradio app for interactive demo of the full ranking pipeline on the 100k dataset (hosted on HuggingFace Spaces)

## Quick Start

> **The final output is already included.** After cloning, open `team_submission.xlsx` — it contains the ranked top 100 candidates with scores and reasoning. No setup needed to view results.

```bash
git clone https://github.com/Jatin0Jain/IndiaRunsSubmission-Candidate-Ranker.git
cd IndiaRunsSubmission-Candidate-Ranker
# team_submission.xlsx is ready to review
python validate_submission.py team_submission.xlsx   # validates the output
```

## Full Reproduction (from scratch)

To regenerate `team_submission.xlsx` from the raw 100k dataset, you need the challenge data bundle (`candidates.jsonl`) provided by Redrob. Place it in a `data/` directory.

```bash
# 0. Install dependencies
pip install -r requirements.txt

# 1. Pre-compute (one-time, ~15 min on CPU)
#    Requires: data/candidates.jsonl (from challenge bundle)
python prep_data.py --data-dir ./data          # → data/candidates_clean.parquet
python prep_embeddings.py --data-dir ./data     # → data/candidate_embeddings.npy
python prep_reasonings.py --data-dir ./data     # → candidate_reasonings.json (needs GEMINI_API_KEY in .env)

# 2. Generate submission (fully offline, ~10 seconds, no API calls)
python rank.py --candidates ./data/candidates.jsonl --out ./team_submission.xlsx

# 3. Validate output
python validate_submission.py team_submission.xlsx
```

### Prerequisites for full reproduction
| Requirement | Details |
|---|---|
| Python | 3.10+ |
| Challenge data | `candidates.jsonl` (100k profiles, from Redrob) → place in `./data/` |
| Gemini API key | Only for `prep_reasonings.py` (set `GEMINI_API_KEY` in `.env`) |
| GPU | Not required |
| Network | Not required during ranking (Step 2) |

## Repository Structure

```
├── rank.py                    # Core ranker: 2-phase scoring → team_submission.xlsx
├── constants.py               # Scoring weights, skill relevance maps, firm lists
├── prep_data.py               # Flattens JSONL → Parquet with honeypot detection
├── prep_embeddings.py         # Generates candidate embeddings (all-MiniLM-L6-v2)
├── prep_reasonings.py         # Pre-generates reasoning via Gemini 2.5 Flash
├── validate_submission.py     # Validates output (supports .xlsx and .csv)
├── app.py                     # Gradio demo UI (HuggingFace Spaces)
├── candidate_reasonings.json  # Pre-computed reasoning strings (used by rank.py)
├── team_submission.xlsx       # ✅ FINAL OUTPUT — 100 ranked candidates
├── submission_metadata.yaml   # Team info, methodology, compute details
├── requirements.txt           # Python dependencies
└── README.md
```

## Ranking Formula

The scorer (`rank.py`) uses a **6-component rule-based score** out of 100:

| Component | Max Points | Description |
|---|---|---|
| Title / Career Match | 30 | Strong ML/AI title = 30 pts; career description evidence for soft titles |
| Skills Quality | 25 | Per-skill: relevance × proficiency × duration × endorsements |
| Experience Years | 15 | Sweet spot: 5–9 years = full 15 pts |
| Company Type | 15 | Product company > consulting; pure consulting = 0 |
| Location | 10 | Noida/Pune > Tier-1 India cities > willing to relocate |
| Behavioural Multiplier | ×0.40–1.15 | Activity, response rate, notice period, GitHub score |

All scoring constants (skill weights, firm lists, normalisation) are in `constants.py`.

## Hackathon Evaluation Metrics

| Metric | Weight |
|---|---|
| NDCG@10 | 50% |
| NDCG@50 | 30% |
| MAP | 15% |
| P@10 | 5% |

## Compute Constraints Met

- **Runtime:** ~10 seconds end-to-end (well under the 5-minute limit)
- **Memory:** <4 GB RAM peak
- **No GPU** required
- **No network calls** during ranking (fully offline)
- **Honeypot protection:** 5-heuristic detection; zero honeypots in final top 100

