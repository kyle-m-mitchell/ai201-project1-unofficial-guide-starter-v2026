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
each short `campus_life` post intact. It wrote the `split_documents` loop for
that rule. Its local check showed 88 documents becoming 88 chunks with the
same text and sources as the starter. I kept the full-post design and used
zero overlap because no post is split; the README describes that observed
behavior instead of claiming that a smaller character window improved search.

**2. Questions and cutoff.** I supplied the housing and drop-deadline facts,
then chose three more campus facts and two criteria of my own. AI helped phrase
those as testable questions and targets and measured the ten retrieval
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
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
