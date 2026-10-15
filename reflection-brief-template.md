Harness Engineering Capstone — Reflection Brief
Name: Harsha Tarun Bimini
Date: 2026-10-15
Environment
- Model(s): Claude via the Anthropic API / capstone environment.
- OS / Python: Linux workspace; Python environment supplied by the capstone project.
- Approx. API spend: Not separately recorded in the final evidence package; the submitted evidence focuses on the measured runs and test results.
Evidence note: The answers below use the latest evidence from the four systems, especially S1 run 20261015_063418 and S2 run 20261015-063800. S4 references use the required 04-fork-scratchpad/solution/data/ location rather than root-level scratch files.

Part 1 — System-specific reflection
S1 — Retry with error feedback / loop control
Q1. What controls the loop, and how do you know it terminates?
The loop is controlled by explicit retry/turn limits and by the model producing a terminal end_turn when the task is complete. The latest S1 evidence is run 20261015_063418, which shows the system routing five claims and recording stop reasons rather than allowing an unconstrained agent loop. This makes termination observable in the trace instead of relying only on a prompt such as “keep trying.” The run evidence therefore demonstrates both bounded execution and an explicit stopping condition.
Artifact: 20261015_063418 S1 run trace / routing evidence.
Q2. Which anti-pattern did you avoid or fix?
The main anti-pattern was allowing the agent to repeatedly retry without a clear success condition, producing unnecessary turns and cost. The corrected design treats retry as a bounded control-flow mechanism: the agent gets useful error feedback, attempts a correction, and then stops when the result is acceptable or the retry budget is exhausted. This keeps the LLM inside a controlled workflow instead of making the LLM itself responsible for deciding whether the system should run forever. The latest S1 run provides observable stop-reason evidence for this behavior.
Artifact: 20261015_063418 S1 run trace.
Q3. How did tool design help the agent?
The tools expose the operations needed to perform the extraction/review work without giving the model an unnecessarily broad interface. The workflow can therefore distinguish between doing work, receiving validation/error feedback, and deciding whether another attempt is warranted. This is safer and easier to debug than giving the model a generic tool and asking it to improvise the whole control flow. The latest run trace shows the resulting tool/turn sequence and terminal stop reason.
Artifact: 20261015_063418 S1 trace.
Q4. What numbers demonstrate that the design works?
The strongest current evidence is the S1 run 20261015_063418, which processed and routed 5 claims and recorded stop-reason traces for the executions. This is important because the evidence is not only a final output; it also records how execution ended. The test suite also provides automated regression coverage for the system. Together, the run evidence and tests demonstrate that the loop-control behavior is observable and repeatable.
Artifact: 20261015_063418; S1 test output.
S2 — Context / token budget
Q5. What reduction did you achieve?
The latest S2 evaluation reduced the measured context from 38,708 tokens to 16,801 tokens, a reduction of 56.6%. This demonstrates that the system did not simply rely on a larger context window; it actively reduced the amount of information passed through the workflow. The reduction was measured in the submitted evaluation rather than estimated from prompt length. This makes the optimization directly comparable and reproducible.
Artifact: S2 run 20261015-063800; budget.json / evaluation evidence.
Q6. What should be summarized, and what must be preserved?
Information that is repetitive, verbose, or already represented elsewhere can be summarized because preserving every byte adds context cost without adding decision value. Information that affects correctness—facts, constraints, identifiers, decisions, errors, and other state needed for the next step—should be preserved explicitly. The S2 design therefore treats context as a budget: remove redundant prose while retaining the facts required to make the same decision. The measured 56.6% reduction is evidence that this principle was applied in practice rather than only described conceptually.
Artifact: S2 budget.json, evaluation output from run 20261015-063800.
Q7. Why keep a facts block?
A facts block provides a compact, stable representation of information that must survive summarization. It separates durable facts from conversational wording, which reduces the chance that an important constraint disappears when context is compressed. This is especially useful when multiple stages need the same facts but do not need the full history of the conversation. The S2 evaluation demonstrates that the system can reduce context substantially while retaining the information needed for the workflow.
Artifact: S2 evaluation/evidence from 20261015-063800.
S3 — Path-scoped instructions / skills
Q8. Why use path-scoped rules?
Path-scoped rules keep instructions close to the code or artifact they govern, reducing the blast radius of a rule. Instead of making every instruction globally active, the system can apply a rule only when the agent is operating in the relevant part of the repository. This improves predictability because unrelated files do not inherit instructions they do not need. The S3 validator provides a deterministic check that the expected configuration is present and valid.
Artifact: S3 validator output: OK; .claude/ path-scoped configuration.
Q9. What did the forked skill add?
The forked skill turned the relevant project knowledge into a reusable, explicit procedure rather than leaving the behavior implicit in a large prompt. This makes the behavior easier to invoke consistently and easier to update independently of the main workflow. It also narrows the instructions to the context where they are useful. The S3 validator confirms that the resulting skill/configuration structure satisfies the required checks.
Artifact: S3 .claude/ skill files; validator output OK.
Q10. How did scope affect reliability?
Narrow scope reduces accidental instruction leakage and makes it easier to reason about which rules are active for a particular task. A globally applied instruction can unexpectedly affect unrelated work, while a path-scoped rule has a smaller and more understandable blast radius. In S3, the validator gives deterministic evidence that the required scope/configuration is correctly wired. This turns configuration correctness into something that can be tested instead of manually inspected each time.
Artifact: S3 validator output OK; .claude/ configuration.
S4 — Multi-shift state / recovery
Q11. What work should be pushed down from the model?
Deterministic bookkeeping should be pushed down into code rather than repeatedly asking the model to remember and recompute it. Examples include maintaining compact state, counting/aggregating shift information, checking invariants, and writing the state artifact. The model should focus on interpretation and decisions while deterministic code handles durable state transitions. The S4 shift evidence demonstrates this separation through a run with 0 new defects and a compact persisted state.
Artifact: S4 shift output under 04-fork-scratchpad/solution/data/.
Q12. How does the system recover after a crash?
Crash recovery comes from persisting the minimal state required to resume rather than depending on the model's conversational memory. After a restart, the system can reload the compact hot state and continue from the recorded state instead of reconstructing the entire prior conversation. This makes recovery deterministic and reduces the amount of context that must be replayed. The latest S4 evidence includes the persisted hot-state artifact and the corresponding shift output.
Artifact: 04-fork-scratchpad/solution/data/hot_state.json and S4 shift output.
Q13. Why keep state small?
Small state is easier to validate, persist, load, inspect, and recover. The latest S4 evidence reports a 643-byte hot state, which is far smaller than carrying an entire conversation or shift history forward. Keeping only the information required for the next decision also limits context growth and reduces the recovery surface. This is a concrete example of pushing durable state into a deterministic artifact rather than into model memory.
Artifact: 04-fork-scratchpad/solution/data/hot_state.json — measured size: 643 bytes.
Part 2 — Cross-system reflection
Q14. What are the three layers of a reliable agentic system?
The three useful layers are deterministic code/state, LLM reasoning, and evaluation/validation. Deterministic code should own boundaries, retries, persistence, schemas, and other operations where exact behavior matters. The model should handle interpretation, synthesis, and other reasoning tasks where flexibility is valuable. Tests, validators, traces, and measured runs then verify that the combined system behaves as intended. The four systems demonstrate this pattern through bounded runs, token measurements, deterministic validation, and persisted state.
Artifact: S1 stop-reason trace; S2 budget/evaluation evidence; S3 validator OK; S4 persisted state and shift output.
Q15. What belongs in deterministic code versus the prompt?
Rules that must always be enforced—retry limits, file/path boundaries, schemas, state persistence, validation, and hard stop conditions—belong in deterministic code. Prompts are better for explaining goals, reasoning strategies, prioritization, and how the model should interpret information. Putting hard constraints only in prompts makes reliability depend on the model following instructions perfectly. The capstone systems therefore use code and artifacts to enforce important boundaries while using prompts/skills to guide reasoning.
Artifact: S1 loop/stop evidence; S3 validator; S4 hot-state artifact.
Q16. What are the two faces of context?
Context has both a reasoning face and a state/data face. The reasoning face is the information the model needs to make the current decision, while the state/data face is the durable information the system must preserve across steps or restarts. Treating them as identical leads to unnecessarily large prompts and fragile recovery. S2 demonstrates context reduction, while S4 demonstrates compact durable state.
Artifact: S2 budget.json / evaluation run 20261015-063800; S4 data/hot_state.json.
Q17. How did the project improve reliability?
Reliability improved by moving critical behavior away from implicit model memory and into observable mechanisms: bounded loops, explicit stop reasons, measured context budgets, path-scoped configuration, deterministic validation, and persisted state. Each mechanism gives a different kind of safety: control-flow safety, context safety, configuration safety, or recovery safety. The evidence is measurable rather than anecdotal: S1 records five routed claims and stop reasons, S2 records a 56.6% token reduction, S3 prints OK, and S4 records zero new defects with a 643-byte state artifact. This combination makes failures easier to diagnose and regressions easier to catch.
Artifact: S1 20261015_063418; S2 20261015-063800; S3 validator OK; S4 data/ artifacts.
Q18. What is the blast radius of a bad instruction or bad state?
A bad global instruction can affect unrelated parts of the repository, while a path-scoped instruction limits the damage to the area where it is relevant. Similarly, a bad persisted state can contaminate later decisions if the system blindly trusts it, so state should be small, structured, and validated. The project reduces blast radius by scoping configuration, validating state/output, and keeping deterministic boundaries outside the model. This is one reason the systems favor narrow interfaces and explicit artifacts over a single unrestricted agent loop.
Artifact: S3 .claude/ path-scoped configuration and validator OK; S4 data/hot_state.json.
Part 3 — What I learned
Q19. What broke or caused the biggest problem?
The biggest submission problem was not the core four-system evidence; it was the missing reflection layer. The implementation evidence was present, but leaving the reflection brief as the original blank template meant the reviewer could not grade the reasoning behind the artifacts. This showed that a technically working system can still fail evaluation if the required evidence and explanation are not packaged correctly. The lesson is to treat the final submission format and evidence mapping as part of the engineering task, not as paperwork to complete at the end.
Artifact: final reflection brief; latest S1/S2/S3/S4 evidence package.
Q20. What would I change next time?
I would build the evidence and reflection structure at the same time as the implementation rather than after the systems are finished. For every user story, I would record the run ID, key metric, test result, and artifact path immediately after verification. I would also run a final submission checklist that checks for blank reflection answers, stale/root-level evidence files, and mismatched artifact paths before uploading. This would make the final review almost mechanical and would reduce the chance of an otherwise working project being marked unable to review.
Artifact: S1–S4 evidence package; final submission structure under 04-fork-scratchpad/solution/data/.
