

## Quick start

```bash
pip install -r requirements.txt

# Reproduces Section VI-A (the gate check). No API key required.
python -m evaluation.run_gate_eval

# Re-score the cached model runs. No API key required.
python -m evaluation.run_labeler_eval
python -m evaluation.run_variance_eval --no-run
python -m evaluation.run_flow_judge_eval
```

The three model experiments resume from the caches in `reports/labeler/` and
`reports/flow_judge/`, which are complete for all seven configurations. They make
API calls only for work that is not already cached. To re-run them from scratch,
copy `.env.example` to `.env` and supply keys.

---

## Where each result in the paper comes from

| Paper | Claim | Code | Artifact |
|---|---|---|---|
| §V-A, App. B | Corpus: 105 documents, 21 types, 24 agents, 2,520 flows | `pipeline/doc_specs.py`, `pipeline/generate_dataset.py` | `data/dataset.json` |
| §IV-A, Tab. III–IV | The CI label schema | `core/Agent.py` | — |
| §IV-B, App. A | The gate and its decision procedure | `core/ci_policy.py` | — |
| §IV-B, Tab. V | Tiers and which require an assignment | `core/ci_policy.py` | — |
| §V-A | Agents and the assignment relation | `world/clinic.py`, `world/agents_pool.py` | — |
| §V-B, Tab. VI | Flow taxonomy F1–F6 | `evaluation/eval_flows.py` | — |
| **§VI-A** | **Gate reproduces its specification: 259 permit, 2,261 deny, 0 FP, 0 FN** | `evaluation/run_gate_eval.py` | `reports/gate_accuracy_report.json` |
| **§VI-B, Tab. VII** | **Label conformance: tier 78–86%, ACL 23–80%** | `evaluation/llm_labeler.py`, `evaluation/run_labeler_eval.py` | `reports/labeler_eval_report.json`, `reports/labeler/*.jsonl` |
| §VI-B, Fig. 2 | Direction of role-grant error (O:U ratio) | `evaluation/make_figures.py` | `reports/figures/x2_direction.png` |
| **§VI-C, Tab. VIII–IX** | **Self-consistency over five draws; 13–124 decisions change** | `evaluation/run_variance_eval.py` | `reports/variance_report.json` |
| §VI-C, Fig. 3 | Unstable decisions and the subset the policy denies | `evaluation/make_cross_figs.py` | `reports/figures/x3_stability.png`, `x5_instability.png` |
| §VI-D | Controls: generation cue, inert field, ensemble | `evaluation/run_labeler_eval.py`, `evaluation/run_variance_eval.py` | same reports |
| **§VI-E** | **PrivaCI-Bench HIPAA split: 214 flows, best 87.8%** | `evaluation/llm_flow_judge.py`, `evaluation/run_flow_judge_eval.py`, `evaluation/hipaa_flows.py` | `data/hipaa_flows.json`, `reports/flow_judge_report.json`, `reports/flow_judge/*.jsonl` |
| App. C | Labeler and judge prompts, verbatim | `evaluation/llm_labeler.py`, `evaluation/llm_flow_judge.py` | — |
| — | Corpus validation (coverage, consistency, content) | `pipeline/validate_dataset.py` | `reports/validation_report.json` |

Model plumbing common to the three model experiments is in
`evaluation/llm_client.py` and `evaluation/providers.py`. The seven
configurations are Opus 4.8, Opus 4.8 with thinking disabled, Sonnet 5,
Haiku 4.5, GPT-5.4, GPT-5.4-mini and GPT-5.4-nano.

---

## Layout

```
core/         CI label schema and the deterministic gate
world/        the 24 agents, their roles, and the assignment relation
pipeline/     corpus specification, generation, normalization, validation
evaluation/   flow construction and the four experiments
data/         dataset.json (the corpus) and hipaa_flows.json (PrivaCI-Bench)
reports/      every result in the paper, plus per-model raw output caches
paper/        LaTeX source (IEEEtran), figures, bibliography
```

### A note on `data/dataset.json`

The corpus is committed deliberately. It is 105 documents whose **labels were
hand-authored first** and whose content was then generated to satisfy them
(§V-A); the labels are the policy under test, so the repository is not
reproducible without them. Document content and extracted attribute values were
generated with Sonnet 4.6, as the paper's acknowledgment records.

### A note on `reports/labeler/` and `reports/flow_judge/`

These JSONL files are the raw labels and judgments returned by each model, one
file per configuration, keyed so runs resume without re-billing. They are what
makes the scoring independently checkable: anyone can recompute Tables VII–IX
from them without an API key and without trusting our aggregation.

---

## Reproducibility notes

- `evaluation/run_gate_eval.py` is fully deterministic and needs no network.
- The model experiments are **not** deterministic, and that is the finding of
  §VI-C rather than a defect: identical requests produce different labels. The
  cached outputs are the specific draws the paper reports.
- Sampling differs by provider. Opus 4.8 and Sonnet 5 reject sampling
  parameters; the OpenAI models were run at temperature 0 where the endpoint
  permits it. §V-D states the consequences for cross-provider comparison.
- Figures were produced by `make_figures.py` and `make_cross_figs.py` from the
  committed reports.

## Limitations

The paper's limitations are in §VII-A and are not restated here. The two that
most affect how this artifact should be read: the corpus is synthetic,
single-domain, and labeled by a single annotator, so conformance results cannot
separate model error from human disagreement; and the PrivaCI-Bench experiment
is judge-only and single-draw, evaluating decision quality rather than the
architecture.

## Citing

<!-- TODO: add the citation once the venue and year are fixed. -->

## License

<!-- TODO: choose a license. Common choices for a research artifact are MIT or
     Apache-2.0 for the code and CC BY 4.0 for the corpus and paper. Note that
     data/hipaa_flows.json is derived from PrivaCI-Bench; check that project's
     license before redistributing it, and consider replacing the file with a
     download script if its terms require attribution-only use. -->
