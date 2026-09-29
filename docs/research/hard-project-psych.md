# A hard, self-directed project with ADHD: what the evidence supports

## TL;DR

- Build **external structure**, not a system that requires sustained enthusiasm: one current milestone, a concrete next action, a visible timer, and a written restart note.
- Adult-ADHD CBT has direct trial support; body doubling, a particular Pomodoro interval, and an “interest-based nervous system” are not equivalently established treatments.
- Define “enough” before starting: a bounded practice commitment plus evidence of an attempt, not “understand the whole subject.” Exact daily quotas are personal experiments, not scientific prescriptions.
- Learn with worked examples → a small independent attempt → feedback → later retrieval. Difficulty is useful when it induces learning, not merely when it hurts.
- AI can improve assisted performance while impairing subsequent unaided performance. Structured tutoring can also improve learning: tool design and what the learner actually does matter.
- Ten options are not inherently harmful, but uncertainty and complexity make overload more likely. Ask for one justified next step; retain the decision yourself.
- There is no basis here for concluding that your thinking has permanently atrophied. Measure what you can explain and do without help, rather than how fluent AI-assisted work feels.

## Scope and how to read the evidence

Prepared 2026-09-29 for a 30-year-old software engineer rewriting a research language in Rust while learning Rust, compilers, and type theory. This is a targeted evidence review, **not a systematic review or clinical assessment**. Most learning studies do not involve adults with ADHD or months-long solo programming; transferring their findings is an inference.

**Strong** means a broad experimental/review base for the stated phenomenon; **moderate** means relevant trials or convergent but bounded findings; **limited** means small/context-specific studies, observational evidence, or expert advice. These are practical judgments, not formal GRADE ratings. A strong finding does not make every suggested implementation strongly evidenced. All project-specific practices below are adaptations, not tested prescriptions for this exact project.

Sources were checked through journal/author pages, PMC, PubMed/Europe PMC, and publisher-deposited Crossref records. Some older sources were available only as bibliographic records or abstracts, not full text; avoid treating this as a fresh methods audit of every classic. General web search returned no results in this session, so coverage—especially of 2026 work and non-indexed body-doubling studies—is incomplete. Recent AI examples below are peer-reviewed 2025 studies, not a claim to exhaustive 2023–2026 coverage.

## 1. ADHD and staying with a long project

### 1.1 Externalized organization has better evidence than “try harder”

**Finding — moderate, directly relevant clinical evidence.** Safren et al. randomized 86 medication-treated adults with persistent ADHD symptoms to 12 sessions of ADHD-focused CBT or relaxation plus educational support. CBT improved blinded symptom ratings; ADHD-rating-scale response was 67% versus 33%. Its components included a calendar/task-list system, breaking tasks into steps, timing attention, writing distractions down rather than following them, and cognitive restructuring. This supports the **package**, not a causal claim that any one checklist or timer is sufficient. Follow-up maintenance was encouraging but complicated by subsequent treatment and responder selection. [Safren et al., 2010, *JAMA*](https://pmc.ncbi.nlm.nih.gov/articles/PMC3641654/).

NICE recommends environmental modifications and, when non-drug treatment is indicated for adults, structured ADHD-focused psychological support with regular follow-up; CBT can be part or all of it. These are clinical recommendations, not proof of a productivity hack. [NICE NG87, §§1.5.15–1.5.18 and environmental modifications](https://www.nice.org.uk/guidance/ng87/chapter/recommendations).

**Practice:** Keep one small work card: `current test/example`, `next physical action`, `unknowns parked for later`. When reading prompts “I must understand this other topic first,” record it before opening anything. If impairment or anxiety remains substantial, consider ADHD-focused CBT with a qualified clinician; a project system is not a replacement for treatment.

### 1.2 “Time blindness” is a useful description, not a single settled mechanism

**Finding — limited-to-moderate.** A review of adult time-perception research found few studies, mixed results, and differing methods across estimation, reproduction, and management of time. It does not support assuming that every adult with ADHD has the same internal-clock deficit. Nor does an appealing label explain all initiation, planning, or switching difficulties. [Mette, 2023, “Time Perception in Adult ADHD: Findings from a Decade—A Review”](https://doi.org/10.3390/ijerph20043098).

**Practice:** Make time visible: choose a work interval you can realistically sustain, set an end cue, and record predicted versus actual time for a few tasks. Treat 25 minutes as a starting experiment, not a biologically privileged interval. Use the discrepancy to shrink tomorrow’s scope, not as a measure of character.

### 1.3 A cue-linked starting action is more actionable than a broad intention

**Finding — moderate general evidence; indirect for adult ADHD.** Implementation intentions specify **“if situation X occurs, I will do Y.”** A meta-analysis found benefits for goal attainment across many settings. It does not establish that a particular morning routine cures ADHD-related initiation problems. [Gollwitzer & Sheeran, 2006, “Implementation Intentions and Goal Achievement”](https://doi.org/10.1016/S0065-2601(06)38002-1).

**Practice:** “After breakfast at my desk, I open yesterday’s work card and run the one failing parser test.” Add an obstacle rule: “If I want to switch projects, I capture the idea and return to this test until the scheduled review.” This is a small executable action, not “work on the compiler.”

### 1.4 Novelty, interest, and company are plausible supports—not a complete ADHD theory

**Finding — moderate for reward-delay differences; limited for the popular formulations and named tactics.** A case-control meta-analysis found greater monetary delay discounting in ADHD (25 comparisons, N = 3,913; d = 0.43): delayed rewards were devalued more steeply on average. This is a group-level association in monetary tasks, not proof that an individual cannot pursue distant goals, nor a direct measure of novelty seeking. [Jackson & MacKillop, 2016](https://doi.org/10.1016/j.bpsc.2016.01.007). It gives a firmer rationale for making near-term feedback visible than for relying entirely on the distant reward of finishing a language.

The categorical claim that ADHD is an “interest-based nervous system,” incapable of responding to importance, is a popular clinical/self-help framing, not a diagnostic mechanism established by the guidance or studies above. Likewise, interest and novelty do not establish that daily project-hopping is necessary or beneficial. I did not confirm a robust controlled adult-ADHD evidence base for **body doubling** as an isolated intervention. Clinical support for structure or involving others should not be silently relabeled as evidence for quiet co-working. [NICE NG87, §§1.4.10, 1.5.5, 1.5.18](https://www.nice.org.uk/guidance/ng87/chapter/recommendations); [Safren et al., 2010](https://pmc.ncbi.nlm.nih.gov/articles/PMC3641654/).

**Practice:** Keep novelty *inside* the same milestone: alternate tracing an example, writing a test, and implementing one rule. If company helps, try two scheduled co-working sessions and compare start delay and completed attempts with solo sessions. Keep it if useful; do not buy an elaborate service because it is presented as proven ADHD treatment.

## 2. What counts as “enough” today?

### 2.1 Use proximal targets to make a distal ambition governable

**Finding — moderate for the principle; limited direct transfer.** Bandura and Schunk’s classic experiment found advantages of proximal subgoals for children learning arithmetic, including competence and self-efficacy, compared with distant goals or no goals. It was not a trial of adult software-project schedules. [Bandura & Schunk, 1981, “Cultivating competence, self-efficacy, and intrinsic interest through proximal self-motivation”](https://doi.org/10.1037/0022-3514.41.3.586).

**Practice:** Preserve “rewrite the language” as direction, but make the current milestone “parse and evaluate three closed arithmetic expressions.” Today’s target can be “trace and test the precedence of `1 + 2 * 3`.” That is inspectable; “learn parsing” is not.

### 2.2 Process goals can support learning, but outcomes still provide feedback

**Finding — limited-to-moderate, context-specific experiments.** Zimmerman and Kitsantas studied shifting from process goals to outcome goals during skill acquisition. Their evidence favors attending to technique while learning, then shifting toward results as skill develops—not the slogan that process goals always beat outcome goals. The task was a motor skill, not compiler construction. [Zimmerman & Kitsantas, 1997, “Developmental phases in self-regulation: Shifting from process goals to outcome goals”](https://doi.org/10.1037/0022-0663.89.1.29).

**Practice:** Pair a controllable commitment (“one focused attempt at a typing derivation, then check it”) with an observable artifact (“a derivation or a precise counterexample to my understanding”). A failed attempt with a documented blocker can satisfy the commitment. Do not make finishing an unpredictable theorem the price of permission to stop.

### 2.3 Visible meaningful progress is promising; “small wins” is not a daily quota theorem

**Finding — observational, limited causal evidence.** Amabile and Kramer’s knowledge-worker diary research connects perceived progress in meaningful work with better inner work life. Their accessible “progress principle” account is **Harvard Business Review/practitioner publishing**, not a randomized intervention proving that daily streaks or trivial checkbox completion cause productivity. Mood and progress can influence each other. [Amabile & Kramer, 2011, “The Power of Small Wins”](https://hbr.org/2011/05/the-power-of-small-wins).

**Practice:** Log one meaningful change in state: “I found why substitution captures a variable,” not “I read 25 pages.” On blocked days, the change can be a smaller, testable question. Review accumulated evidence weekly rather than resetting your judgment to zero every morning.

### 2.4 A restart plan is better justified than deliberately creating an unfinished cliffhanger

**Finding — limited-to-moderate experimental evidence for planning; weak for the stopping folklore.** Masicampo and Baumeister found that specific plans reduced intrusive effects of unfinished goals in laboratory tasks. This is not a trial of stopping mid-sentence. The Zeigarnik effect concerns memory/activation of unfinished tasks; it does not by itself imply that deliberately leaving work unfinished improves next-day motivation. Hemingway-style stopping advice is an anecdotal craft heuristic. [Masicampo & Baumeister, 2011, “Consider it done!”](https://doi.org/10.1037/a0024192).

**Practice:** Stop at the predeclared time boundary and leave: `what changed`, `what is unresolved`, `first action next time`. Prefer “run test X and inspect the left operand” to “continue parser.” No need to leave code deliberately broken just to create tension.

## 3. Learning hard technical material without endless side quests

### 3.1 Retrieval and spacing have the strongest broadly applicable support

**Finding — strong general learning evidence.** Dunlosky et al.’s review rated practice testing and distributed practice highly across learners, materials, and assessment tasks; rereading and highlighting were much less consistently useful. This concerns durable learning, not merely immediate ease or familiarity. [Dunlosky et al., 2013, “Improving Students’ Learning With Effective Learning Techniques”](https://doi.org/10.1177/1529100612453266).

**Practice:** After a section on substitution, close it and explain capture avoidance or trace one example. Check and correct immediately. Revisit the same idea tomorrow and several days later. These intervals are convenient defaults, not a universal optimal spacing schedule. Retrieve explanations and procedures, not just vocabulary.

### 3.2 For novices, worked examples can be more productive than prolonged unguided struggle

**Finding — moderate-to-strong for guided learning in studied domains; indirect for this project.** Worked-example research in algebra demonstrates that studying solutions can be a useful substitute for extensive initial problem solving. Cognitive-load theory explains why searching a large problem space can consume the capacity needed to learn the structure. Assistance should diminish as knowledge grows; neither “always solve from scratch” nor “always read solutions” follows. [Sweller & Cooper, 1985, “The Use of Worked Examples as a Substitute for Problem Solving in Learning Algebra”](https://doi.org/10.1207/s1532690xci0201_3).

**Practice:** Study a tiny, correct type-checker example; explain each branch; complete a partially missing branch; then implement an analogous construct unaided. A hint or worked example is appropriate when you cannot identify the next operation. Repeatedly staring at unfamiliar notation is not automatically a desirable difficulty.

### 3.3 Deliberate practice means targeted improvement, not maximal hours or discomfort

**Finding — moderate association, not an exclusive causal explanation of expertise.** Macnamara et al.’s meta-analysis found that deliberate-practice measures explained differing amounts of performance variation across domains, much less in education and professions than in games/music. Definitions and measurement are contested, and explained variance is not an individual causal percentage. It refutes simplistic “hours alone explain mastery” claims, not the value of practice. [Macnamara, Hambrick & Oswald, 2014](https://doi.org/10.1177/0956797614535810).

“Desirable difficulties” should be operationalized as useful learning operations—such as retrieval and spacing—not as a requirement to make every task more frustrating; see the evidence in 3.1.

**Practice:** Isolate one weakness: write three substitution examples, predict the result, compare with tests or an authoritative worked solution, then repair the error. This is a more informative practice unit than spending another evening passively watching compiler videos.

### 3.4 Multi-pass reading is a sensible expert method, not a validated ADHD intervention

**Finding — expert guidance, limited experimental evidence for the named method.** Keshav’s three-pass method separates orientation, broader understanding, and detailed reconstruction. It offers a principled alternative to treating every unknown term as an immediate prerequisite. The article is practical computing-research advice, not a controlled learning trial. [Keshav, 2007, “How to read a paper,” *ACM SIGCOMM CCR*](https://doi.org/10.1145/1273445.1273458).

**Practice:** Before opening a paper, write the question it must answer for the current milestone. First pass: contribution, assumptions, relevant section. Second: one example and the argument’s outline. Third: reconstruct only the proof or algorithm you need. Mark gaps `blocking now` or `later`; return to a gap when an exercise/test shows that it really blocks understanding. This is prioritizing depth, not pretending to understand.

## 4. AI, cognitive offloading, and skill formation

### 4.1 Assisted performance is not the same as acquired skill

**Finding — moderate causal evidence in one educational setting.** Bastani et al.’s field experiment with nearly 1,000 high-school mathematics students found that GPT-4 assistance improved practice performance, but the standard-interface group performed worse than controls when help was removed. The abstract reports a 17% relative reduction in grades for GPT Base—not 17 percentage points. Learning-oriented safeguards largely mitigated that harm. This is evidence about acquisition and subsequent unaided performance, **not proof of permanent loss of previously acquired programming skills**. [Bastani et al., 2025, *PNAS*](https://doi.org/10.1073/pnas.2422633122). The [published correction](https://pmc.ncbi.nlm.nih.gov/articles/PMC12403119/) concerns an author affiliation, not these results.

**Practice:** For a skill you want to own, attempt it before requesting a solution; ask for a single hint; then close the chat and solve a small variant. Let the agent handle packaging chores if those are not learning targets, but retain the typing rule or ownership reasoning you are practicing.

### 4.2 AI tutoring can improve learning when instruction is deliberately structured

**Finding — moderate short-term causal evidence, not universal superiority.** Kestin et al.’s randomized crossover study in undergraduate physics found higher immediate learning gains with a custom AI tutor than with in-class active learning. The design used sequential problems, pedagogical prompts, and expert-provided solutions; it was not an unconstrained chatbot. Only two lessons were studied, and long-term retention and complex synthesis were not established. [Kestin et al., 2025, *Scientific Reports*](https://www.nature.com/articles/s41598-025-97652-6).

**Practice:** Give the tutor an authoritative reference and one small objective: “Ask me to predict this borrow-checker result, wait, then explain only my error.” The study does not show that a good system prompt alone reproduces a tested tutoring platform; verify technical feedback against the compiler, tests, or source text.

### 4.3 Reports of less critical thinking are warning signals, not evidence of brain damage

**Finding — observational, limited for causation.** Lee et al. surveyed 319 knowledge workers about 936 GenAI-use examples. Higher confidence in AI was associated with less reported critical thinking; confidence in one’s own task ability was associated with more. The study also describes thinking shifting toward verification, integration, and oversight. Self-reports and associations cannot establish that AI caused skill decline. [Lee et al., CHI 2025, author publication page](https://www.microsoft.com/en-us/research/publication/the-impact-of-generative-ai-on-critical-thinking-self-reported-reductions-in-cognitive-effort-and-confidence-effects-from-a-survey-of-knowledge-workers/), [DOI](https://doi.org/10.1145/3706598.3713778).

**Practice:** Before seeing the agent’s recommendation, write your own proposed next step and why. Afterward, name one assumption you checked independently. Assess progress with unaided explanations and small implementations, not a subjective diagnosis that your mind has “atrophied.”

### 4.4 Offload remembering the task; preserve generating the reasoning you want to learn

**Finding — strong memory phenomenon, indirect AI application.** The generation-effect meta-analysis summarized 86 studies and found a memory advantage for generating information over reading it (average effect about 0.40). These are mainly controlled memory tasks, not evidence that every generated code token teaches or that all outsourcing is harmful. [Bertsch et al., 2007, “The generation effect: A meta-analytic review”](https://pubmed.ncbi.nlm.nih.gov/17645161/).

**Practice:** Use AI to maintain the task list and quiz you, but generate the explanation, prediction, or derivation yourself. If you needed a full solution, reconstruct it later without looking and then change one assumption. This reconciles useful external memory with preserving meaningful cognitive practice.

## 5. Choice overload and decision fatigue: separate the claims

### 5.1 “More choice is always worse” did not survive aggregation

**Finding — strong reason to reject universality; context-sensitive positive evidence.** Scheibehenne et al.’s meta-analysis found a mean choice-overload effect near zero with substantial variation. A later meta-analysis by Chernev et al. identified moderators: complexity, task difficulty, uncertain preferences, and effort-minimizing goals. The first result is not proof that overload never exists, and the second is not proof that ten options is a harmful threshold. Most tasks concerned consumer choice, not agents or ADHD. [Scheibehenne, Greifeneder & Todd, 2010](https://doi.org/10.1086/651235); [Chernev, Böckenholt & Goodman, 2015](https://doi.org/10.1016/j.jcps.2014.08.002).

**Practice:** Ask the agent: “Recommend one next step within this milestone, state the reason, and give at most one alternative only if it changes the decision.” Use a bounded comparison when you actually need an architecture decision, not whenever you are trying to resume work. Reducing the menu is justified by your observed friction, without needing a universal neuroscience story.

### 5.2 A failed ego-depletion replication is not proof that fatigue is imaginary

**Finding — strong negative evidence for the tested protocol; uncertainty about broader theories.** A preregistered 23-lab replication with 2,141 participants found an ego-depletion effect of d = 0.04, with a confidence interval spanning zero. This challenges the large, reliable “limited willpower resource” effect in that sequential-task setup. **Decision fatigue**, time-on-task fatigue, sleep loss, and choice overload are not interchangeable with ego depletion; this experiment does not refute all of them. [Hagger et al., 2016, “A Multilab Preregistered Replication of the Ego-Depletion Effect”](https://doi.org/10.1177/1745691616652873).

**Practice:** Preselect tomorrow’s first task and batch scope decisions into a weekly review to reduce repeated deliberation. Take breaks because they help your observed functioning, not because science established a fixed daily tank of decisions. Do not infer that a hard afternoon means the project is wrong.

## A minimal two-week experiment

This is a synthesis to test, **not a clinically validated protocol**. Start with less machinery than you think you need.

1. **One milestone:** one observable slice of the language, plus a short “not this week” list. New project ideas go into a parking list until the weekly review.
2. **Define enough before work:** for example, two 25-minute attempts plus a five-minute restart note. Adjust to available capacity; the numbers have no special evidence status. An explicit blocker can be the day’s artifact. Stop at the agreed boundary rather than escalating the quota after each success.
3. **One learning loop:** worked example if needed → own attempt → specific feedback → one later unaided reconstruction. Ask the agent for one prompt or hint at a time.
4. **Track only three things:** did I start the planned slice; what changed or became clearer; can I explain/reproduce one part without assistance? Record excessive strain too; progress is not a reason to ignore it.
5. **Weekly, change one variable:** shrink scope if starts fail; improve examples if confusion stays global; reduce answer-giving if unaided performance lags assisted output. Do not respond by rebuilding the entire productivity system.

Suggested agent instruction:

> Help me learn this one milestone. First ask for my current prediction or attempt. Give one next step or hint, not a menu. If I lack a prerequisite, show one small worked example and then ask me to do a variant. Park unrelated questions. Do not implement the part I am practicing unless I explicitly ask; if you do, schedule an unaided reconstruction. At shutdown, help me write the exact next action.

## Claims I could not confirm

- A trial-validated daily amount of work, optimal timer interval, or stopping rule for this kind of months-long ADHD project.
- A robust controlled adult-ADHD evidence base for body doubling alone; my search cannot establish that no such studies exist.
- That an “interest-based nervous system,” novelty-seeking, or a single dopamine explanation fully accounts for these difficulties—or requires switching projects daily.
- That Hemingway’s stop-mid-sentence advice is validated by the Zeigarnik effect, or that unfinished work reliably increases next-day motivation.
- That AI has permanently atrophied this user’s thinking, or that short-term educational results establish long-term loss of an adult programmer’s existing skills.
- That ten options always cause overload, or that every person has a fixed, exhaustible daily decision budget.
- That this particular workflow is optimal, or that partial comprehension should always be tolerated: some missing prerequisites really do need targeted repair.
