# Clinical summary audit workflow

An n8n workflow that runs a clinical medication summary through three separate language model calls instead of one, so that a second and third pass can catch what the first got wrong.

It is an importable workflow file, not an application. Five nodes: a schedule trigger, a source text block, and three HTTP requests to the Hugging Face inference router.

## The idea

Asking a model to summarise clinical information in one pass gives you a fluent answer with no indication of what it left out. Missing a contraindication reads exactly like not having one.

Splitting the task across three prompts makes the disagreement visible:

- **Researcher** turns the source text into a medication summary.
- **Challenger** reads that summary against the source and lists what is missing or unsafe, particularly contraindications.
- **Judge** takes both and decides whether the summary is safe to pass on.

The value is not that the model becomes correct. It is that a specific critique step produces something you can read, so a human reviewer sees where the two passes disagreed rather than a single confident paragraph.

## Running it

1. Import `clinical-audit-workflow.json` into n8n.
2. Create a Header Auth credential holding your Hugging Face token, and attach it to the three HTTP Request nodes. The exported file references credentials by ID only, so no token is included here.
3. Put your source text in the **Source Text** node.
4. Run it manually, or leave the schedule trigger to run it on an interval.

## Honest limits

This is a prompt chain, not a safety system, and it should not be described as one:

- Three calls to the same model share the same blind spots. A contraindication absent from the model's knowledge is missed by all three passes.
- Nothing is checked against a drug database or any external source of truth. The judge compares two pieces of generated text.
- There is no evaluation set behind it. It has not been measured against known-bad summaries, so the improvement over a single pass is plausible rather than demonstrated.
- The JSON formatting between nodes is requested in the prompt, not enforced by a schema, so a malformed response breaks the chain.

Treat it as a demonstration of the critique-then-judge pattern. It is not fit for use on real patient information.

## Licence

MIT.
