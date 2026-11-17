# Proposed changes: Chapter 3 flow and diagrams (DRAFT, for discussion)

Source doc: `rag-technical-solution-architecture-part-2-v112.docx`
Inputs reviewed: `decoding.txt`, `prefill.txt` (chat transcripts with suggestions), diagrams `image10`–`image15`, alt-text on `image3` / `image20`.
Status: analysis only. Nothing in the .docx has been modified.

## 1. Diagram changes

| # | Image (doc position) | Change | Source |
|---|---|---|---|
| D1 | `image10.png` (2.1, "A retrieval vector is not a decoder token embedding") | The generation plane shows `size d_model` (plain underscore) while the retrieval plane shows `d` with a true subscript (`d_retrieval`). Re-render `d_model` as `d` with subscript `model`, same font, size and baseline as `d_retrieval`. Change nothing else. | User request |
| D2 | `image14.png` (3.2, "From hidden state to selected next token") | Raw logits split into two branches that look like one flow. Make it clear that greedy and sampling are alternative decoding configurations (set per request, never switched mid-generation). Keep the two lanes under the shared "Raw logits" box and label them "Greedy config (T = 0)" and "Sampling config (T > 0)". Do not add a decision diamond. Keep the existing "Softmax not required" and "T = 1" notes. | `decoding.txt` (second half) |
| D3 | `image15.png` (3.3, "Prefill, decode and the per-layer KV cache") | (a) Draw one shared "Decoder layers 1…L" block with a wide parallel path (prefill) and a looping single-token path (decode). (b) Move the Per-layer KV cache to a central position so prefill writes to it and decode reads and appends in a loop. This removes the long cross-diagram arrows. (c) Optionally add "Q is never cached; only K and V are". Keep the GQA/MQA footer and the two callouts. | `prefill.txt` |
| D4 | `image3.jpeg` (Ch. 1) | Pending edit, from the alt text. Make the bottom-left yellow caption box's font match the other diagram text, keep the text "A chunk record is the unit of retrieval, provenance and citation.", and give the box a clean solid border. Change nothing else. | Alt-text instruction |
| D5 | `image20.jpeg` (5.1) | Pending edit, from the alt text. Remove only "5.1 " from the title so it starts "Exception flows…". | Alt-text instruction |

Cleanup: after D4 and D5 are applied, replace the instruction-style alt text on `image3` and `image20` with real descriptive alt text.

Style constraints for all regenerated images (same as the existing set): hand-drawn marker look, cream paper background, pastel fills (blue, yellow, green, purple, pink) with dark outlines, the same handwriting font, and the same canvas size and aspect ratio (about 1150 px wide in the originals).

### Execution note
The `codex` CLI is **not installed** on this machine. Options:
1. Install and authenticate Codex, then run one image edit per image with a strict "change only X" prompt, the original image as reference, and the style constraints above.
2. Use another image-edit tool.
3. Hand-edit D1, D4 and D5 (small text edits) and regenerate only D2 and D3.

Whichever tool is used, I would diff each result against its original to confirm that nothing outside the intended region changed.

## 2. Text and flow changes for Chapter 3

Problems for a technical reader:
- **Order of ideas:** The chapter jumps from the 3.1 layer figure to equations and a parameter/runtime table. Then 3.2 goes into logits. 3.3 opens with the prefill/decode result before explaining what K and V are. The reader sees "Ana/Rao", "KV cache" and "Q, K, V" before the attention mechanism is introduced.
- **Missing bridge:** There is no bridge between "decoder layer" (3.1) and "attention inside the layer" (3.3). Attention is the only place Q, K and V exist, so 3.3 has to teach it and the cache at the same time.
- **Dense paragraphs:** Several paragraphs are heavy. Examples: the 3.2 temperature/top-k/top-p paragraph, and the 3.3 paragraph that carries a table, prefill and decode together.
- **Overclaim:** The closing bridge says answer quality "collapses" when the corpus is stale.
- **Determinism:** "deterministic token selection" is stated without a qualifier (see item 6 below).

Proposed restructure (same content, better order):

1. **3.0 Intro, mini-roadmap.** Add a three-bullet map: what the model computes (3.1), how it picks a token (3.2), how it makes that cheap (3.3). Keep the Ch.2 → Ch.3 handoff sentence.
2. **3.1 Decoder state.** Keep the figure and the residual equations. Add a short "attention is the sublayer where tokens look at each other; the MLP works per position" sentence. Move the learned-parameters vs request-time table to the end of 3.1 as a recap.
3. **3.2 Logits to token.** Order the content as: LM head → logits → greedy vs sampling (the two configurations) → worked numeric example → temperature, top-k and top-p, in short bullets instead of one paragraph → "probability ≠ factual support". Add a note that some hosted APIs return logprobs.
4. **3.3 split into two parts.**
   - **3.3a Attention in one layer.** Q, K and V, the formula and the Q/K/V table, using the Ana/Rao example.
   - **3.3b KV cache.** Start with the problem (recomputing past K and V at every step). Then prefill vs decode, the figure (D3), capacity limits, the MHA/GQA/MQA factor, and a sentence on prefix caching that links back to Ch.2's stable prompt prefix. Add total context = input + output to the limits table.
5. **Closing bridge.** Change "collapses" to "degrades". Say that decoding choices do not fix a stale or over-permissive corpus.
6. **Wording fixes.**
   - Qualify "deterministic" (floating-point and batching effects can still change outputs).
   - Fix the 3.3 opening paragraph's non-body style (`NormalWeb`).
   - Check that the missing-looking symbols ("Top-", "At ") are actually equation objects that render in Word.
   - Re-number the citations in order of first appearance (currently [1][5], [20], [1][6], [19]…).

### Suggestions from the transcripts I would NOT adopt as-is
- `decoding.txt` first half discusses an RRF vs normalized-blending diamond. That belongs to the Chapter 1.4 fusion diagram, not Chapter 3. Check whether 1.4's figure needs it (separate discussion).
- `prefill.txt` says that without the cache, "generation complexity [goes] up to O(N²) per step instead of O(1) compute per historical token". This is imprecise. Without the cache, each step re-projects all N tokens and costs about O(N²) attention. With the cache, each step costs O(N) attention plus O(1) projection work for the new token. If we state complexity at all, use these figures.
- The Gemini analogy text (book summary index) is fine for chat but too informal for the guide.

## 3. Open questions for discussion
1. Which image tool should I use (install Codex, or something else)? Is Codex expected to be available on another machine?
2. Do you want the 3.3 split into 3.3a and 3.3b, or should I keep one section and reorder it?
3. Is D3's re-layout (one shared decoder block, central KV cache) wanted, or only text fixes to the current figure? It is the largest visual change.
4. Should the D4 and D5 pending edits be included in this pass?
5. Should I edit the .docx in place with a script (the equations are Word equation objects and are risky to touch), or deliver a new version, for example v113?
6. Should I check the Chapter 1.4 fusion diagram for the RRF vs normalized-blending point?

---

## 4. Status after the first pass (saved as `...-v113.docx`; v112 untouched)

**Done**
- D1: `d_model` is now `d` with subscript `model`, drawn from the original glyphs (same font, colour, background).
- D4, D5: the images were already edited; only the stale instruction alt text was replaced with real descriptions.
- Chapter 3 text: roadmap paragraph, attention before cache, merged duplicate prefill/decode text, a "RAG implication" line closing 3.1, 3.2 and 3.3, a rewritten bridge to Chapter 4, "collapses" → "degrades", determinism qualified, total-context-window row added to the limits table, prefix-caching sentence added.
- Removed: the learned-parameters vs request-time table (folded into one paragraph).

**Not done (needs an image-generation model; Copilot CLI cannot draw raster images in this style)**
- D2 (`image14`): two lanes labelled "Greedy config (T = 0)" and "Sampling config (T > 0)" under the shared raw-logits box, no decision diamond.
- D3 (`image15`): one shared "Decoder layers 1…L" block, central per-layer KV cache, prefill writes and decode reads and appends.
- Both: keep the pencil/stencil look, pastel fills, handwriting font, cream background and the existing 16:9 canvas; change nothing else.

**Still open**: citation numbers are not in order of first appearance; check the equations render in Word.

## v114 (current): Chapter 3 compressed

Based on v113; v112 and v113 are untouched. Supersedes the earlier plan above (images D2/D3 were supplied by the user from M365/Gemini; no Codex was used).

- Chapter opening now explains that the chat template serializes the Chapter 2 messages before tokenization [7].
- 3.1 reduced to one figure and two paragraphs; 3.2 and 3.3 trimmed; the 3.2/3.3 handoff duplication removed.
- New 3.4 "Stopping and output limits" with a stop-condition table (no figure). The streaming sentence moved here from 3.3.
- The equations, softmax calculation and Q/K/V table stay inline in 3.1, 3.2 and 3.3 for a single reading (a separate "Deep dive" section was tried and dropped). Dense paragraphs were split where the ideas differ: learned vs request-time values, KV memory vs runtime policies, stop conditions vs finish-reason reporting, and streaming risk vs service requirements.
- Cross-references to 3.4 added to the Chapter 2 token budget, 5.1.3, and the Generation checklist line.
- Not checked visually (no renderer available). Open in Word to confirm equations, table widths, spacing and figure sizes.
- 3.4 now cites two new references: Hugging Face "Text generation" (EOS token, stop strings, max new tokens) and the OpenAI Chat Completions API reference (finish reason).
- All citations renumbered in order of first appearance ([1]–[24]); the reference list was reordered to match, and adjacent citations are sorted and spaced consistently. The three in-text hyperlinks still point to their original sources.

## v114 follow-up: Chapters 1, 2 and 4

**1.4:** deleted the duplicate fusion table; replaced the vendor claim (Weaviate: ranked and relative-score fusion; Azure AI Search: RRF [8]; Qdrant dropped); tightened the opening; C5 RRF score in text is 0.03227 (fig06 still shows 0.03226).

**Chapter 1:** split the opening paragraph at "Relevance:"; added the colon after "two architectural approaches"; replaced line-break hacks in Stable chunk ID with paragraphs and removed a duplicate handoff sentence; removed "(with associated knowledge graph)" from 1.2; "(deterministic)" in 1.4a is now "through fixed queries"; 1.5 cutoff applies to the fused score, subhead casing and period fixed; citation spacing in 1.4 and 1.6.

**Chapter 2:** step list relabelled to match fig11; line-break gap after "response validation." fixed; duplicate "untrusted" statements reduced; chapter-ending scope note removed; failure-mode paragraph moved before the closing handoff and reworded; explicit pointer to section 3.4 removed from the token-budget paragraph.

**Chapter 4:** code sample uses `stable_chunk_id(identity)` and passes `content_hash` to `upsert`; opening reordered (intro, freshness definition, lifecycle steps, then identity and code); lifecycle summary sentence removed; Publish step adds a rollback line; 4.2 notes the model reads the request as one sequence; subheading is "Return a structured answer with its citation".

**Kept:** 5.1.3 still refers to section 3.4.

## v114 follow-up: Chapter 4 steps, Chapter 5 and figures

**Chapter 4:** Index step now says to embed with the same model revision that embeds user questions and to keep search on the current version while loading; Publish step says to change the active version only after all stores finish, never update in place or delete the old version first, and keep the old version for rollback.

**Chapter 5:** removed the duplicated "Ingestion publishes source versions…" sentence from the 5.1 intro; rewrote 5.1.1–5.1.3 in plainer language with Team Notes examples; 5.1.2 now says the user gets an insufficient-evidence result with no model call; the generation paragraph is split so recovery stands alone; the "Generate + validate output means…" definition now follows the roadmap table; closing section is numbered 5.2.

**Figures replaced:** fig06 (C5 score 0.03227, matches the text); roadmap (fixed "use case" and "authorized", side margins added). Both were cropped to remove empty space and their heights adjusted in the document.

**Open:** the outcome order differs between the 5.1.3 list (Completed, Degraded, Retry pending, Failed) and the exception figure footer (completed, failed, retry pending, degraded); the exception figure text is small at page width; Word rendering is unchecked.

**Exception-flow figure (Figure 5.1) replaced** with a larger-text version: typo-free, the "hard stop is a route" note dropped (the STOP boxes and 5.1.2 text already say it), footer order now Completed, Failed, Retry pending, Degraded. The 5.1.3 outcome list was reordered to match. This closes the open items above. All 20 figures in `images/` now match the images embedded in v114.
