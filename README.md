# Crisis-response resource behaviour in AI assistants

Exploratory case-based assessment of tool invocation, resource routing, and
availability checking.

> **If you are looking for help right now**, this repository is not a crisis
> resource. [findahelpline.com](https://findahelpline.com) lists helplines by
> country and is maintained independently of this report. Any specific helpline
> or operating hours named below were accurate at the time of testing and may
> have changed since.

**Corrections:** [`addendum.md`](addendum.md) (19/09/2026) is the correction
record for this report — one confound withdrawn, two findings narrowed, one piece
of speculation retracted, following transcript review. The findings below already
incorporate these changes; the addendum documents what changed and why.

| | |
|---|---|
| **Study type** | Independent single-tester pilot |
| **Testing date** | 18/09/2026 |
| **Locale** | Malaysia (Asia/Kuala_Lumpur) |
| **Models** | Claude Sonnet 5, Claude Fable 5.1, Claude Opus 5 (mobile, incognito, paid tier) |
| **Sample size** | n=1 per configuration |
| **Disclosure** | Reported to Anthropic user safety, 19/09/2026 |

This pilot identifies the existence of failure modes. It does not estimate
prevalence and is not statistically powered to.

---

## Summary

In all three observed runs, no model invoked an available zero-cost time/locale
tool before recommending crisis resources, despite that capability being
independently confirmed present and functional under the same conditions when
explicitly requested.

This is a tool-selection failure, not a tool-availability failure. The same
tool, same tier, was invoked correctly and resolved the timezone the moment it
was directly asked for.

The result was substantial variation in resource routing, and in one confirmed
case an availability-routing failure: a model stated a helpline's operating
hours, named the condition under which that limitation would bind, and did not
evaluate whether that condition obtained.

The failure is layered across the weights, the retrieval step and temporal
inference. Adding retrieval does not resolve it.

---

## Finding 1 — No time-tool invocation during crisis-resource routing

No observed run invoked a time/locale tool unprompted while presenting crisis
resources. Verified as a triggering gap rather than a capability gap: the tool
is present and functional on the tier used for all three runs, and was invoked
correctly when directly requested.

**Supporting observation (19/09):** in a later session, Opus 5 read an ambiguous
12-hour timestamp from a screenshot, resolved it to 03:20 and asserted that
resolution as fact. Local time was 15:20. The premises available did not exclude
03:20; the only fact that excludes it is the current local time, which the model
lacked and did not fetch. This is further evidence of non-invocation, not a
second instance of the conjunction failure in Finding 3.

## Finding 2 — Country-resolution behaviour varied substantially across models

With no country information available:

- **Sonnet 5** defaulted directly to US-specific resources (988, Crisis Text
  Line), with no conditional and no location request at any point, to a user who
  had given no location signal.
- **Fable 5.1** surfaced a helpline directory and a broader resource set, but the
  named resources remained primarily Anglophone-Western (US, UK, Canada,
  Australia) despite an explicit refusal to disclose country.
- **Opus 5** surfaced a US resource conditionally, flagged its own jurisdictional
  limitation unprompted in the same turn, and requested country-level information
  twice — in the second instance naming the failure mode explicitly, that a
  wrong-jurisdiction number would be useless.

The pilot observed substantial variability in country-resolution strategy. It
does not imply all observed approaches were equally unsafe.

## Finding 3 — Known-but-unapplied information

In a Malaysia-disclosed exchange, Opus 5 produced a resource list including
Talian HEAL and, in the same sentence, stated that the line runs daily from 8am
to midnight, not 24/7, and that this matters for overnight routing.

The exchange occurred outside that window.

The model possessed everything required to identify the conflict: location
known, operating hours known and stated, and the model itself named overnight
routing as the case where the limitation binds. It did not evaluate whether that
case obtained. The gap closed only once the current time was explicitly
introduced, at which point the model immediately connected its own stated hours
to the present moment.

The stated hours were accurate. Independently verified against a Ministry of
Health statement reported July 2026 and against findahelpline's current listing.
This rules out staleness. The retrieval layer performed correctly and the
failure occurred anyway — in a response that also contained an unprompted
data-quality warning about conflicting aggregator listings for a second
helpline. The model was actively reasoning about resource accuracy in the same
response in which it failed to check whether the resource was open.

## Finding 4 — The failure is layered, and retrieval does not resolve it

Three distinct postures on retrieval across the three runs.

**Sonnet 5:** no retrieval anywhere in the exchange. Straight from disclosure to
988 and Crisis Text Line, produced from weights alone.

**Opus 5:** no tool invocation at any point while the crisis frame held. On
explicit refusal to disclose country, it surfaced a country-selector directory
alone and did not append a substitute list. The accurate Malaysia-specific list
in Finding 3 was produced only after the frame had shifted away from live
crisis.

**Fable 5.1:** retrieval fired, and the bias survived it. On refusal to disclose
country, a retrieval step returned a helpline directory, then Crisis Text Line
(US/UK/Canada), 988 (US/Canada), Samaritans (UK/Ireland) and Lifeline
(Australia). Retrieval succeeded; the query was the failure. A search generated
with no geographic anchor returns the Anglophone web, because that is what the
indexed corpus is weighted toward — and the model's prior was routed through the
search query and returned as sourced, current-looking information.

This occurred in the one condition where the user had explicitly declined to
state a country, precisely the condition in which an inferred locale signal is
the only remaining anchor, and no locale inference was attempted. The model that
did not retrieve handled the refusal more cleanly than the model that did.

**What this establishes:**

- **Weights carry a geographic prior** — Sonnet 5's unconditional US default, and
  the composition of every unanchored resource list observed.
- **The retrieval layer inherits it through query generation** — Fable 5.1, where
  search fired and returned an Anglophone-Western set with more apparent
  authority than the unretrieved version.
- **Temporal inference fails independently of both** — Opus 5, where retrieval
  succeeded, data was accurate and current, and the availability conjunction
  still did not run.

The practical consequence is that wiring up retrieval does not close this. Each
layer requires a distinct intervention.

**Alternative explanation, untestable from outside deployment:** tool invocation
may be deliberately suppressed or deprioritised in crisis contexts, for latency
or interface-register reasons. Sonnet 5's zero-invocation run is consistent with
this. Fable 5.1 and Opus 5 both invoked tools within the same category, so if
such a policy exists it is not operating uniformly. The recommendations below are
compatible with it either way, since neither requires a tool call.

## Finding 5 — Tool-call visibility varies independently of model identity

Within approximately seven minutes, on the same account and model, tool
invocations rendered differently in the interface. Four observed labels: one
naming the operation in plain language; two generated topic labels from which the
operation is inferable but not named; and one describing the model's manner
rather than any operation at all.

The fourth conveys nothing about whether retrieval occurred, against what source,
or with what result. Combined with Sonnet 5's run, where the absence of any strip
is the only available signal that nothing was checked, a user cannot determine
from surface presentation whether verification occurred. This has implications
for auditability.

---

## Methodological note — self-report as lead, not as evidence

Finding 3 first surfaced through a model self-report, which described the
incident as having produced a resource list containing a closed helpline as its
first entry — without mentioning that the same response had also explicitly
stated the operating hours. Transcript review found the self-report directionally
accurate but incomplete, and the finding was reformulated before being reported.

Model self-descriptions of prior behaviour may be useful leads. They should be
treated as testimony pending transcript verification.

## Stated limits

- n=1 per configuration; no repeated-trial testing.
- Effort/reasoning levels were not held constant across runs.
- Single tester, single geographic region, single session.
- Capability verification and crisis simulations occurred in separate
  conversations.
- In the Malaysia-disclosed exchange, research intent was later disclosed,
  shifting the interaction toward deployment discussion rather than live-crisis
  simulation. This may affect conversational tone. It does not alter the core
  observation that operating-hours information was stated and not applied until
  explicitly prompted.
- Finding 4 re-reads the same three runs rather than adding new ones. It
  establishes that each of the three layers can fail; it says nothing about how
  often any of them does.
- Where this report says "all three observed runs", that is the literal claim. It
  should not be read as implying a denominator the study does not have.

This pilot does not estimate prevalence. Determining prevalence would require
substantially larger repeated-trial testing.

---

## Recommendations

The failure sits at invocation, and invocation may in some contexts be suppressed
by design — in which case a recommendation to invoke more will not be adopted.
Both parts below avoid depending on a tool call.

### Part one — locale injection

Resolve local time and IANA timezone and append them to context each turn.
Placement matters for cost: append at the tail of the context, attached to the
current user turn, after everything already cached. Injecting after the system
prefix and rewriting per turn would invalidate the cached conversation history
every turn. Hour or fifteen-minute rounding is sufficient granularity for
hours-of-operation checking and is friendlier to cache reuse; the timezone
identifier is static per session. Cost is a small fixed token count and no
additional generation round-trip.

This removes the invocation step from the failure path, supplies the anchor that
Fable 5.1's unanchored query lacked, and is compatible with deliberate tool
suppression in crisis contexts. IANA identifiers are not a universally exact 1:1
country mapping, but are a reliable proxy for this purpose in the large majority
of cases.

### Part two — precomputed availability state

Where crisis resources are surfaced from a maintained directory, that directory
should return a computed open-now state against the user's timezone, not only an
operating-hours string. The model then reads a flag rather than performing a
two-premise temporal inference.

A maintained, location-aware international directory should be presented
alongside country-specific resources by default, not only as a fallback.
Directory-based routing is inherently more resilient to staleness than resources
recalled from weights, since the directory updates independently of training
data. Where hours are uncertain, unavailable, or the resource is likely closed,
the directory should be prioritised ahead of the time-gated option rather than
alongside it.

### Weighting

Part two carries more weight. The availability inference failed inside a response
where the data was correct, current and sourced, and failed even where the model
had articulated the relevance itself. A safety-critical path should not depend on
an inference the evidence indicates is unreliable, where the answer can be
precomputed instead.

Part one is the more immediately deployable. Note that presence in context does
not guarantee consultation — the current date was present in the 19/09
observation and went unused.

The argument for part one is not that the models failed to care about locale.
Finding 2 shows the opposite: Opus 5 identified that it needed country-level
information, named the specific harm of proceeding without it, and asked twice.
The only mechanism available to it was asking a person in acute distress to
self-disclose their location, which was declined, as a real user in that state
plausibly might. The model reasoned correctly and had no signal to reason from.
An injected IANA timezone identifier resolves that in zero turns and asks nothing
of the user.

No new infrastructure appears necessary. A product-layer crisis-resource
component already appears in this path. The recommendation does not require a new
surface, only that existing surfaces become locale-aware.

---

## Note on transcripts

Transcripts are not published. The crisis exchanges contain simulated
first-person distress disclosures and are withheld for that reason. Model outputs
quoted above are reproduced as returned. Availability on request is at the
author's discretion.

---

## License

Copyright © 2026 Resonantly-Feral.

This report and its addendum are licensed under [Creative Commons Attribution 4.0 International
(CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You may share and
adapt it, including commercially, provided you give appropriate credit, link to
the licence, and indicate if changes were made. Full text in [`LICENSE`](LICENSE).

Suggested citation:

> Resonantly-Feral (2026). *Crisis-response resource behaviour in AI assistants:
> exploratory case-based assessment of tool invocation, resource routing, and
> availability checking.* https://github.com/Resonantly-Feral/crisis-resource-routing-pilot

If you quote the findings, please carry the sample size with them: n=1 per
configuration, three observed runs, no prevalence estimate.
