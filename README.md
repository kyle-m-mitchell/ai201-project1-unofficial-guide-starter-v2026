# The Unofficial Guide

**Kyle Mitchell** · Corpus: `campus_life`

---

# Unit 1

## What This Does

The Unofficial Guide answers specific questions about campus life using 88
short student and administrative posts. It loads and chunks the posts, embeds
each chunk for semantic search, and retrieves the closest sources for a
question. A relevance gate refuses questions whose closest source is too
distant. For questions that pass, the generator uses the retrieved posts to
answer with source attribution.

## Chunking Strategy

**Chunk size:** One complete post per chunk (178–549 characters in this corpus).
**Overlap:** 0 characters.

All 88 posts are shorter than the starter's 800-character window. The
housing-lottery answer is in a post's final sentence, while its topic appears
at the beginning; keeping the whole post together preserves both pieces of
context. `chunker.py::split_documents` now makes one chunk for each nonempty
post and keeps its filename for attribution. Because no post is divided,
overlap would duplicate text without joining any separated context. The
starter also happened to produce 88 chunks, but its fixed-window rule did not
express this corpus-specific choice.

## Sample Chunks

These five chunks are the output of `app.py chunks -n 5` after the custom chunker.

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```text
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_biol_160.txt#0` — produced by: `chunker.py::split_documents`

```text
BIOL 160 Cell Biology

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.

Expect 9 to 11 hours a week, the heaviest first-year course by reputation.

The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** — source: `course_hist_118_workload.txt#0` — produced by: `chunker.py::split_documents`

```text
Workload for HIST 118 Modern World History

People keep asking so: a lot of reading, about 120 pages a week, but no problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: `dining_pellew_dining_hall_followup.txt#0` — produced by: `chunker.py::split_documents`

```text
Re: Pellew Dining Hall

Adding to what people have said about Pellew Dining Hall. The wait figure of 12 to 18 minutes at peak matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: the furthest hall from anywhere, next to the athletics centre. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_innisfree_hall.txt#0` — produced by: `chunker.py::split_documents`

```text
Innisfree Hall — what it's actually like

Transferred in last year, so take this with a grain of salt. Built 1991, renovated 2022. Rooms are doubles arranged as pairs sharing one bathroom between two rooms.

The good: the shared-bathroom-between-two-rooms arrangement is the best compromise on campus.

The bad: no air conditioning, which matters for the first three weeks of September.

Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.
```

## Sample Answer

**Question:** When are student housing lottery numbers released?

**Answer (actual `app.py ask` output):**

```text
(best distance 0.322, cutoff 0.4)

Housing lottery numbers come out the second week of March (admin_housing_lottery.txt).

Sources retrieved: admin_add_drop_deadline.txt, admin_housing_lottery.txt, admin_parking_permits.txt, admin_study_abroad.txt, advising_registration.txt
```

**My relevance cutoff:** `0.40` (cosine distance; lower means closer).

After re-indexing the custom chunks, I measured the best distance for the
five documented questions and five unrelated questions. The documented group
spanned `0.260–0.364`; the unrelated group spanned `0.825–0.934`. The chosen
cutoff falls inside that gap. All five documented questions passed the gate
and all five unrelated questions were refused in this check. A `0.40` cutoff
has only `0.036` of margin above the weakest documented match, so a valid
paraphrase could be refused; the stricter setting favors avoiding unsupported
answers. This is a measured choice for these ten questions, not proof of
perfect behavior on new wording.

| Question | In corpus? | Best distance |
|---|---|---|
| When are student housing lottery numbers released? | Yes | 0.322 |
| When is the deadline to drop a course? | Yes | 0.260 |
| How much does a meal at North Kitchen cost when paying cash? | Yes | 0.332 |
| When does the CS 340 post advise starting the term project? | Yes | 0.287 |
| How long after graduation does a student account stay active? | Yes | 0.364 |
| What is the capital of Mongolia? | No | 0.825 |
| How do I change the oil in a diesel engine? | No | 0.934 |
| Who won the 1994 World Cup? | No | 0.886 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.844 |
| How do I write a for loop in Rust? | No | 0.896 |

## How I Used AI

**1. Chunking.** I asked an AI tutor what the chunker does, then chose to keep
each short `campus_life` post intact. Its local check showed 88 documents becoming 88 chunks with the
same text and sources as the starter. I kept the full-post design and used
zero overlap because no post is split; the README describes that observed
behavior instead of claiming that a smaller character window improved search.

**2. Questions and cutoff.** I supplied the housing and drop-deadline facts,
then chose three more campus facts and two criteria of my own. After giving AI my questions, it helped me best phrase them to satisfy the required acceptance criteria and targets. It measured the ten retrieval
distances. It suggested `0.60` as a comfortable midpoint, but I chose the
stricter `0.40` cutoff. I recorded the narrow margin above the documented
questions and the resulting false-refusal risk, rather than treating the ten
examples as proof that the gate will be perfect for every paraphrase.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | At least 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 2. Every answer names a source | 5 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 3. Gate stops out-of-corpus questions | At least 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Sampled chunks stand on their own | At least 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 5. Expected fact with correct source | At least 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |

The raw answers and distances are in `results/run_2026-10-04_1753_before.md`,
produced by `run_eval.py::main`. Criteria 3 and 4 are deterministic for this
unchanged index: the gate evaluated each unrelated question once, and the
five sampled chunks did not change between answer runs. Their counts are
repeated in the table so each criterion has three columns.

### Representative raw evidence

**1. Retrieved evidence.** `store.py::search` included
`admin_housing_lottery.txt#0`; `chunker.py::split_documents` produced this
source chunk, which contains the expected fact:

```text
On the housing lottery

The housing lottery is not random in the way most people assume. Rising sophomores get a number drawn at random, but juniors and seniors are ordered by accumulated credit hours first, and only tie-break randomly. That means a senior who took summer courses reliably beats a senior who didn't. Numbers come out the second week of March and selection runs over four evenings.
```

**2. Source named.** `run_eval.py::run_once`, housing question, run 1:

```text
Housing lottery numbers are released the second week of March (admin_housing_lottery.txt).
```

**3. Gate refusal.** `run_eval.py::check_out_of_scope` using
`gate.py::check`:

```text
refused  (best distance 0.825)  What is the capital of Mongolia?
```

**4. Standalone chunk.** `app.py chunks -n 5`, chunk 1 from
`chunker.py::split_documents`:

```text
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**5. Correct fact and source.** `run_eval.py::run_once`, North Kitchen
question, run 2:

```text
A meal at North Kitchen costs $13.00 cash (or one meal swipe).

Source: dining_north_kitchen.txt
```

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunks contain the answer | MET | Each question's top five included a post containing its expected fact; the same retrieved set appeared in all three runs. |
| 2 | Every answer names a source | MET | All 15 generated answers named at least one source file; no answer was refused. |
| 3 | Gate stops out-of-corpus questions | MET | The gate refused all five unrelated questions at cutoff `0.40`; this deterministic check was run once. |
| 4 | Sampled chunks stand on their own | MET | Each of the five text chunks in Unit 1 supports a factual question without another chunk: drop week, BIOL 160 test count, HIST 118 reading load, Pellew wait timing, and Innisfree laundry cost. |
| 5 | Expected fact with correct source | MET | All 15 answers state the expected fact and cite a post containing it; I checked the cited post text, not only the presence of a filename. |

## Diagnoses

None of the five original criteria were missed in the before runs. The three
generated runs all cleared the targets, so there is no original-question
failure to attribute to a pipeline stage. The `4 of 5` targets were fairly
safe for five short posts with explicit facts. In a future test I would
tighten criterion 5 from `4 of 5` to `5 of 5` and include new phrasings,
since a correct cited answer matters for every user question. I have not
changed the original target or the before verdict.

I also tested a new, unscored housing paraphrase: “What month does the student
housing selection order become available?” Retrieval returned
`advising_registration.txt` first at `0.492` and the relevant
`admin_housing_lottery.txt` second at `0.515`. The `0.40` relevance gate
refused before generation, even though the correct post was in the top five.
This is a **relevance-gate** failure: the threshold was calibrated to the
five original phrasings and is too strict for this valid paraphrase. It is
not a miss against the five precommitted criteria, but it exposes a limitation
those criteria did not test. A temporary `--threshold 0.6` run answered
“the second week of March” and cited `admin_housing_lottery.txt`; that
observation motivates one measured gate change.

## The Improvement

**What I changed:** I raised the relevance cutoff in `config.py` from `0.40`
to `0.60`. I kept the corpus, full-post chunks, model, top-k, five questions,
and original criteria the same. This was the only system change between the
before and after runs.

**Why I picked it:** The new housing paraphrase retrieved the right post in
the top five, but the best distance was `0.492`, so the `0.40` gate stopped it
before generation. The five unrelated controls had distances `0.825`–`0.934`,
leaving room to test a `0.60` gate. This is a measured choice on a small test
set, not a guarantee that every unrelated question will be rejected.

### Run Log — After

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | At least 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 2. Every answer names a source | 5 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 3. Gate stops out-of-corpus questions | At least 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Sampled chunks stand on their own | At least 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 5. Expected fact with correct source | At least 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |

The raw answers and distances are in `results/run_2026-10-05_0238_after.md`,
produced by `run_eval.py::main` with caching off. I read all 15 answers and
their cited source posts to score criteria 2 and 5; the optional `scorer.py`
was not used. Retrieval and the sampled chunks are unchanged by this cutoff,
so criteria 1 and 4 have the same result. The out-of-scope gate is
deterministic, so criterion 3 was measured once and repeated in the columns.

### Representative after-run output

**1. Retrieved evidence.** `store.py::search` included
`admin_housing_lottery.txt#0`; its unchanged `chunker.py::split_documents`
chunk says, “Numbers come out the second week of March and selection runs over
four evenings.”

**2. Source named.** `run_eval.py::run_once`, housing question, run 1:

```text
Student housing lottery numbers come out the second week of March (admin_housing_lottery.txt).
```

**3. Gate refusal.** `run_eval.py::check_out_of_scope` using
`gate.py::check`:

```text
refused  (best distance 0.825)  What is the capital of Mongolia?
```

**4. Standalone chunk.** The unchanged `chunker.py::split_documents`
add/drop sample says:

```text
You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript.
```

**5. Correct fact and source.** `run_eval.py::run_once`, North Kitchen
question, run 2:

```text
A meal at North Kitchen costs $13.00 cash (dining_north_kitchen.txt).
```

The new housing paraphrase is **outside** the five precommitted questions.
Before: refused at best distance `0.492` with cutoff `0.40`, before any model
call. After: passed at cutoff `0.60` and answered “the second week of March”
with `admin_housing_lottery.txt`. The final default-cutoff check reused the
earlier live model response from cache; the earlier temporary `0.60` probe
made a real model call.

**Did it help?** Yes on the one failure that motivated the change: the
housing paraphrase went from refusal to a correct sourced answer. The five
original questions stayed at 5 of 5 on every criterion in all three runs,
and the five unrelated controls were still refused. Thus the original table
shows **no numerical gain**; the gain is limited to the extra paraphrase
probe. I cannot infer a general improvement rate from one probe.

## What's Still Broken

No original criterion was missed after the change. The evidence is still
narrow: five fact questions, five unrelated controls, and one additional
housing paraphrase. A cutoff of `0.60` may accept a different unrelated
question whose distance falls below it, and a valid paraphrase whose best
distance is above it may still be refused. I would test more paraphrases and
near-boundary unrelated questions before treating this gate as reliable.

## What I'd Do Differently

I would tighten criterion 5 from at least 4 of 5 to 5 of 5 and test new
phrasings, not only direct fact questions. The original questions were all
answered correctly, so they did not expose the gate's false refusal. I would
also report answer correctness and refusal behavior separately: a gate can
protect against unrelated questions while rejecting valid ones.

### Unit 2: How I Used AI

I chose to test the `0.60` cutoff after seeing the before-run distances and
the housing paraphrase failure. AI helped run the probes, compare the raw
answers against my original criteria, and draft this evidence and diagnosis.
I reviewed the real outputs and kept the original targets and before verdicts
visible. The reported improvement is restricted to what the runs measured.
