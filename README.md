## Workflow

The framework uses four functional agents with distinct responsibilities, aiming to generate a concise summary that captures domain-relevant information while remaining faithful to the source and preserving the meaning of safety-sensitive content.

1. The Guidance Extraction Agent selects source-grounded, domain-aware sentences;
2. The Guided Summarization Agent uses these sentences as guidance to generate a task-adapted initial summary;
3. The Verification Agent then assesses the summary for faithfulness and safety consistency and identify actionable issues;
4. The Revision Agent proposes targeted edits, and decides whether to accept the revision after re-verification.

<img width="1524" height="540" alt="fig1" src="https://github.com/user-attachments/assets/d704ccdd-c397-46f1-9198-4ef87e621a69" />


## Prompts

Followings are detailed prompts used for guidance controller, guided summarizer, faithfulness checker, safety checker and revision editor. 

The runtime prompt definitions are available in [src/mentsum_agent/prompts.py](src/mentsum_agent/prompts.py).

<img width="1071" height="411" alt="prompt1" src="https://github.com/user-attachments/assets/c6f2a9cd-c4f5-4328-bf20-7d41525f8cc8" />
<img width="1082" height="319" alt="prompt2" src="https://github.com/user-attachments/assets/9c610261-3be3-48b0-909a-139bdaf2fa47" />
<img width="1064" height="383" alt="prompt3" src="https://github.com/user-attachments/assets/6b94b1f4-6381-43ff-8133-2c38d579e5eb" />
<img width="1070" height="522" alt="prompt4" src="https://github.com/user-attachments/assets/31d8993d-19d7-4f55-af87-9b0911803962" />
<img width="1084" height="423" alt="prompt5" src="https://github.com/user-attachments/assets/88f69f98-38bf-42d0-9c61-430f3512f8e6" />


## Requirements

- Python 3.10, 3.11, or 3.12
- An API key for an OpenAI-compatible endpoint when using a real model

## Quick start

Clone the repository and create an isolated environment:

```bash
git clone https://github.com/luqian721/mentsum-agentic-workflow/tree/main
cd mentsum-agentic-workflow
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -e .
cp .env.example .env
```

The default configuration uses a deterministic mock backend, allowing to run the workflow without network access or an API key. 

```bash
python -m mentsum_agent.run_one \
  --source-file examples/example.source \
  --index 0 \
  --include-source \
  --env-file .env.example \
  --output outputs/example.json
```

Run all examples as a batch:

```bash
python -m mentsum_agent.run_batch \
  --source-file examples/example.source \
  --include-source \
  --env-file .env.example \
  --id-prefix example \
  --output outputs/example.jsonl
```

Batch output is appended one record at a time. If the command is restarted, completed example IDs are skipped.

## Use a real model

Edit `.env` and set at least:

```dotenv
LLM_BACKEND=openai
LLM_API_KEY=your-api-key
LLM_MODEL=your-model-id
```

Set `LLM_BASE_URL` when using another OpenAI-compatible endpoint. Some models do not support JSON response mode; use `LLM_USE_JSON_MODE=false` for those models. 

Start with a small input file before running a large collection:

```bash
python -m mentsum_agent.run_batch \
  --source-file data/your_data.source \
  --limit 5 \
  --env-file .env \
  --output outputs/pilot.jsonl
```


## Model backends
The workflow supports two summarization backends:

- LLM: Configured through `SUMMARIZER_BACKEND=llm`, this is the default demonstration backend and can also be used for experiments that replace the fine-tuned GSum summarizer.

- GSum: Configured through `SUMMARIZER_BACKEND=external_gsum` and `GSUM_COMMAND`, this backend integrates an external GSum model fine-tuned on MentSum. The original implementation and training instructions are available in the [official GSum repository](https://github.com/neulab/guided_summarization).


## Resources

### Mental-health terminology

We construct a domain-specific terminology resource from the [2016 Kaiser Permanente Convergent Medical Terminology (CMT) Mental Health Problem List Subset](https://www.nlm.nih.gov/pubs/techbull/ma16/brief/ma16_snomed_ct_updated_subsets.html), which contains 1,162 SNOMED CT concepts. 

To enable QuickUMLS, set `TERM_MATCHER=quickumls` and configure the relevant `QUICKUMLS_*` paths in `.env`. A local QuickUMLS installation initialized from licensed UMLS data is required. The UMLS and SNOMED CT resources are subject to their respective [license terms](https://www.nlm.nih.gov/research/umls/Snomed/cmt.html).

For demonstration purposes, the repository also provides a lightweight lexicon matcher configured through `TERM_MATCHER=lexicon`, without requiring UMLS.

### Risk-cue lexicon

We construct a risk-cue lexicon by drawing surface forms from the [Construct-Tracker Suicide Risk Lexicon v1.0](https://github.com/danielmlow/construct-tracker) and organizing them into five domain-specific categories.
The resulting [risk-cue lexicon](resources/construct_tracker_safety_risk_cues_5cat.json) contains 344 surface forms.

The defalut demonstration configuration uses a smaller illustrative lexicon. To use the full resource, set `RISK_CUE_PATH=resources/construct_tracker_safety_risk_cues_5cat.json` in `.env`.


## Input and output

Input is a UTF-8 text file with one complete source post per line. See [`examples/example.source`](examples/example.source) for synthetic input.

`run_one` writes formatted JSON. `run_batch` writes JSON Lines, with one result per source line. Each successful result includes the source hash, extracted guidance, initial and final summaries, verification and revision histories, stopping reason, model ID, prompt versions, and public workflow configuration. See [`examples/example_output.jsonl`](examples/example_output.jsonl) for mock output.


## Privacy and limitations

- Treat source posts and generated outputs as potentially sensitive data.
- Review files before sharing them, even when they are de-identified.
- The included lexicons are demonstrations, not validated clinical resources.
- Model output can be incomplete, incorrect, or unsafe and requires independent evaluation.
- The synthetic examples are fictional and do not represent real people.

## License

This project is released under the [MIT License](LICENSE).
