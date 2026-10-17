# Fairground — LLM evaluation dashboard

A personal full stack prototype for exploring quality and safety evaluation of local language models. The interface is React + Vite; the API is plain Node.js and connects to Ollama's local HTTP API.

## Run it

Requirements: Node.js 18+ and Ollama with a model pulled (for example `ollama pull llama3.1:8b`). Install the dependencies from this folder with `npm install`.

In one terminal, start the API:

```sh
npm run server
```

In another, start the React app:

```sh
npm run dev
```

Open the Vite URL shown in the terminal. Set `OLLAMA_URL` to change the Ollama endpoint and `PORT` to change the API port. The Vite dev server proxies `/api` to `127.0.0.1:8787`.

## Evaluation flow

The API generates answers for built-in or imported datasets, then returns prompt, reference, optional context, metadata, generated answer, heuristic screens, and faithfulness judge output. Bias, toxicity, relevance, and the legacy hallucination screen are simple heuristics for demo/triage only.

When a row has `context`, the configured Ollama judge extracts factual claims and marks them supported, contradicted, or not verifiable against that context. Contradicted and unverifiable claims enter the human review queue in **Datasets** / **Insights**. Rows without context are explicitly marked not evaluated for faithfulness. Judge results are fallible; human reviewers can confirm an issue or mark no issue.

Select a live run in **Recent evaluations** (or click **Breakdown**) to inspect each metric and the generated examples. The overall score is the simple, equally weighted arithmetic mean of all available sub-scores. Faithfulness is omitted if no rows had context; failed judge rows are also excluded from its mean and are surfaced for review. The report explains how each score was produced.

This prototype does **not** call DeepEval or another evaluation framework. The faithfulness judge is an Ollama model selected in the run form (default `llama3.1:8b`). Hallucination, bias, toxicity, and relevance are basic hand-written heuristics. A future DeepEval integration would require a Python service or a compatible Node implementation and explicit framework metric configuration.

## JSONL datasets

In **Datasets**, import either a `.json` array of examples (or an object containing a records array) or a `.jsonl` file with one object per line. An import mapping step detects source keys and lets you map them to `prompt`, `reference`, `context`, and `metadata`, with a preview before saving. Unmapped fields are preserved under `metadata.sourceFields` by default. The app currently accepts up to 5,000 examples and 2 MB per file. Evaluation processes 20 examples per API batch, shows progress, and the **Stop evaluation** button cancels the current Ollama request and prevents later batches from starting. Completed batches remain available as a stopped partial run. Imported sets live in this browser's local storage. In Datasets, choose a sample size (100 by default) and create a random subset without replacement from an imported set; the sample becomes selectable for a new evaluation, and its metadata records the source and sampling method. Export as JSON or JSONL. Examples: [`samples/golden-example.json`](samples/golden-example.json) and [`samples/golden-example.jsonl`](samples/golden-example.jsonl).

For a small local run, import [`samples/quick-smoke-test.json`](samples/quick-smoke-test.json). It contains 8 short, project-authored examples covering groundedness and fairness; it is illustrative, not a benchmark.

## Benchmark catalog

The benchmark cards document upstream provenance, approximate sample size, repository license, source link, and known limitations for BBQ, TruthfulQA, and HaluEval. The app does not bundle benchmark data. Download the official release yourself, review upstream terms (including any underlying dataset terms), and convert examples to the JSONL fields before importing. Keep a record of the exact release/version and any conversion or filtering when comparing model results.

## Evaluation cautions

The benchmark catalog is a reference, not a claim that the dashboard reproduces each benchmark's official protocol. For example, BBQ expects a multiple-choice setup and benchmark-specific bias scoring; TruthfulQA has updated multiple-choice instructions; HaluEval separates recognition tasks from generation. Use official scoring code and protocol for comparable published results. Avoid treating one aggregate score as proof of fairness or factuality. The frontend's seeded overview is illustrative; a live run requires Ollama.

Run history and human review labels are currently stored in browser memory/local storage only; they are not synced between devices or backed up by the Node server.

## Problems I ran into
The original model I used was too large, which made my laptop heat up and for a small example dataset (8 examples) to take over a minute to compute.
