# Run the AEB baseline in Google Colab

## Selected model

**Qwen/Qwen3-4B-Instruct-2507**, loaded with NF4 4-bit weights and FP16 compute. It is a 4B instruction-tuned model under Apache-2.0. This is a practical starting choice for the T4, not a claim that it is the best available model. Actual fit and speed must be verified in Colab.

## Files

- `AEB_Baseline_Colab.ipynb`: notebook to upload to Colab and add to GitHub.
- `aeb_pilot.jsonl`: the existing v0.2 dataset; upload it when the notebook asks.
- `AEB_Spec.md` and `Pilot_Test_Cases.md`: human review references. Their needed content is already carried in the JSONL input.

## Steps

1. Open https://colab.research.google.com/ and upload `AEB_Baseline_Colab.ipynb` using the notebook upload option.
2. Select a T4 GPU runtime.
3. Run installation, settings, dataset upload, prompt checks, and model loading in order. Use a fresh runtime if imports were already loaded at conflicting versions.
4. Run the single-case preview (TC-004). Check that output is produced and inspect the output-limit flag.
5. Run the full batch cell. Each case is checkpointed individually. The preview is reused, not regenerated.
6. Run the export cell to download a ZIP, even after a partial run. Optional Drive storage can be enabled before starting.
7. Review `comparison.md`, complete `manual_review.csv`, and retain `run_config.json` with the results.

## What this establishes

This is zero-shot generation with supplied requirements, specification context, and a scenario objective. No reference answers are sent to the model. There is no project-specific training yet. The 15 records are a development pilot and are not a held-out benchmark. Cases remain unexecuted controller-test drafts.

The exported files include raw predictions, references, exact prompts, token counts, generation time, memory measurements, truncation/format flags, error records, dataset hash/snapshot, resolved model revision, dependency versions, and a blank manual scoring sheet. Automated format diagnostics are not engineering-accuracy scores.

## Reproducibility and limitations

The notebook pins Transformers 4.57.1, Accelerate 1.10.1, and bitsandbytes 0.48.1 while retaining Colab's PyTorch. It records the runtime versions. The model's `main` revision is resolved to a commit SHA before downloading; reuse that recorded SHA for comparisons. Greedy decoding reduces sampling variation but does not guarantee identical outputs on different software/hardware.

The default input limit is 6,000 tokens, with 2,048 new output tokens and batch size 1. Inputs are never silently truncated. A generation that reaches the output limit without EOS is flagged for review. A larger output cap must be treated as a separate documented run. Downloads still transfer the original model files before quantization; 4-bit loading reduces GPU memory, not necessarily download size.

The notebook was checked for Python syntax and non-GPU data/prompt/export behavior. Model loading and inference were **not executed on a T4 here**. No accuracy, timing, or memory result is claimed in advance.

## Sources and acknowledgements

- https://huggingface.co/Qwen/Qwen3-4B-Instruct-2507
- https://huggingface.co/docs/transformers/v4.57.1/en/quantization/bitsandbytes
- https://pypi.org/project/bitsandbytes/0.48.1/
- https://pypi.org/project/accelerate/1.10.1/
- Qwen Team, *Qwen3 Technical Report* (2025): https://arxiv.org/abs/2505.09388
- Project specification CFTS-AEB-001 v0.1 and dataset v0.2.
- Prepared with ChatGPT assistance, 2026-10-15.
