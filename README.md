# Three.js reconstruction miner: merged v7

Image-to-Three.js pipeline for the 404-GEN GPT competition. Root `main.py` exposes
`async def generate(image: bytes, seed: int, witness: AsyncWitnessClient) -> str | None`. The pipeline is in
`miner/merge.py`; everything else is the base miner `Olauge-Secret/sedge-wren@fc79ce3`, unchanged (`provenance.json`).

**Status: offline-tested only** (control flow, request ids, budget). Not yet run live or judged. Codex quota was
exhausted on 2026-11-07.

## Pipeline (about 18 model calls per image)
1. **Two lineages, run concurrently.** These are sedge-wren's chains, kept separate to the end:
   - Astra-medium (lineage 1) and Sol-medium (lineage 2) drafts, render-checked, ≤2 repairs;
   - then a Sol-high rewrite and an Astra-high rewrite;
   - then **2 Sol-high post refines** of the lineage's own result (refine steps 2 and 3 of 3);
   - every rewrite is kept only if it renders;
   - every rewrite turn also shows the module's SIDES sheet (right / back / left / top), with the instruction to
     complete any missing, flat or hollow back or side (idea from `SmoothTight/newvictor`);
   - each lineage ends with sedge's checked wrap + flat-object tilt finish.
2. **Judge-faithful final duel**, which replaces sedge's single Astra-low pick:
   - both finished lineages are rendered from the contest judge's 4 front cameras at 1024 px (white, neutral);
   - each view is judged with the judge's own stage-1 prompt in both presentation orders: 8 Sol-low calls;
   - the lower summed penalty wins; a tie or an unusable reply keeps lineage 1, as sedge's tie rule does.

## Where the ideas come from
- **Base and v5 additions:** Claude research round 2. sedge-wren backbone, newvictor SIDES feedback, concurrent chains,
  render cache, same-id resends. v5 measured locally, pooled over 2 runs: leader +31%, top 3 +4.7%.
- **v7 additions:**
  - candidates stay in independent lineages until the end;
  - the only cross-candidate decision is a copy of the judge's stage 1.
  - **Why:** our loops 2–4 measured that LLM panels swapping in other candidates lost quality (loop 4's 4 swaps:
    net −5).
- **The stage-1 prompt texts are 404's judge-service prompts** (as in our local judge).

## Budget and determinism
- **Fixed shares:** each lineage 2.2M credits / 25 calls; each duel call 75K / 1. Sum = 5.0M / 58, so an image can
  never exceed 5M / 60.
- **Deterministic:** each part's budget guard sees only its own spend, so completion order cannot change which calls
  are made (replay-safe).
- **Astra cap:** the Astra output cap is 24,000 tokens (max observed 15,909). This only lowers the worst-case
  reservation.
- **Expected spend:** about 2.5–3.5M credits per image.
- **24 h fit:** about 2,400 s of model time per image, so 128 images ≈ 14 h at 6 concurrent calls, ≈ 7 h at 12.

## Changes vs v5 (`claude_loop7`)
| | v5 | v7 |
|---|---|---|
| Pick | sedge's 1-call Astra-low panel, before post | 8-call stage-1 copy duel, after both lineages finish |
| Post refines | 3 on the picked chain (Sol, Astra, Sol) | 2 per lineage, both lineages (Sol, Sol) |
| Calls per image | ≈ 10 | ≈ 18 |
