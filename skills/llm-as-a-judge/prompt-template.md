# LLM-as-Judge Prompt Template

This reference contains annotated prompt templates for building judge evaluators.

## Template Structure

Every LLM-as-Judge prompt follows this four-part structure. Adapt the content to your specific failure mode.

---

## Example: Tone Appropriateness (Real Estate CRM)

```
You are an expert evaluator assessing outputs from a real
estate assistant chatbot.

Your Task: Determine if the assistant-generated email to
a client uses a tone appropriate for the specified client
persona.

Evaluation Criterion: Tone Appropriateness

Definition of Pass/Fail:
• Fail: The email's tone, language, or level of formality
  is inconsistent with or unsuitable for the described client
  persona.
• Pass: The email's tone, language, and formality align well
  with the client persona's expectations.

Client Personas Overview:
• Luxury Buyers: Expect polished, highly professional, and
  deferential language. Avoid slang or excessive casualness.
• First-Time Homebuyers: Benefit from a friendly,
  reassuring, and patient tone. Avoid overly complex jargon.
• Investors: Prefer concise, data-driven, and direct
  communication. Avoid effusiveness.

Output Format: Return your evaluation as a JSON object with
two keys:
1. reasoning: A brief explanation (1-2 sentences) for your
   decision.
2. answer: Either "Pass" or "Fail".

Examples:
---
Input 1:
Client Persona: Luxury Buyer
Generated Email: "Hey there! Got some truly awesome listings
for you in the high-end district. Super views, totally posh.
Wanna check 'em out ASAP?"

Evaluation 1: {"reasoning": "The email uses excessive slang
('Hey there', 'awesome', 'totally posh', 'ASAP') and an overly
casual tone, which is unsuitable for a Luxury Buyer persona.",
"answer": "Fail"}
---
Input 2:
Client Persona: First-Time Homebuyer
Generated Email: "Good morning! I've found a few properties
that seem like a great fit for getting started in the market,
keeping your budget in mind. They offer good value and are
in nice, welcoming neighborhoods. Would you be interested in
learning more or perhaps scheduling a visit?"

Evaluation 2: {"reasoning": "The email adopts a friendly,
reassuring tone ('great fit for getting started', 'nice,
welcoming neighborhoods') suitable for a First-Time Homebuyer,
and clearly offers next steps.", "answer": "Pass"}
---
Now, evaluate the following:
Client Persona: {{CLIENT_PERSONA_HERE}}
Generated Email: {{GENERATED_EMAIL_HERE}}

Your JSON Evaluation:
```

---

## Anatomy of the Template

### Part 1: Task & Criterion
```
You are an expert evaluator assessing [WHAT YOU'RE EVALUATING].
Your Task: Determine if [SPECIFIC BINARY QUESTION].
Evaluation Criterion: [NAME OF THE CRITERION]
```

- Assign an expert evaluator role.
- Focus on ONE failure mode per prompt.
- Name the criterion explicitly — this anchors the judge's attention.

### Part 2: Pass/Fail Definitions
```
Definition of Pass/Fail:
• Fail: [PRECISE DESCRIPTION OF WHAT FAILURE LOOKS LIKE]
• Pass: [PRECISE DESCRIPTION OF WHAT SUCCESS LOOKS LIKE]
```

- Ground these in the failure descriptions from error analysis.
- Be specific about boundary conditions (what's borderline?).
- Include domain context the judge needs (e.g., persona descriptions, business rules).

### Part 3: Few-Shot Examples
```
Examples:
---
Input: [THE INPUTS THE JUDGE WILL SEE]
Evaluation: {"reasoning": "...", "answer": "Pass" or "Fail"}
---
```

- Include at least one clear Pass and one clear Fail.
- Draw from human-labeled Training set examples.
- Use clear-cut cases first; add borderline examples only if needed to resolve persistent disagreements.
- The reasoning field in examples teaches the judge HOW to reason, not just what to output.

### Part 4: Structured Output + Input Slot
```
Output Format: Return your evaluation as a JSON object with two keys:
1. reasoning: A brief explanation (1-2 sentences) for your decision.
2. answer: Either "Pass" or "Fail".

Now, evaluate the following:
[INPUT TEMPLATE WITH PLACEHOLDERS]

Your JSON Evaluation:
```

- `reasoning` comes before `answer` — this induces chain-of-thought before the verdict.
- End with a clear prompt for the judge to respond.

---

## Example: Explanation Clarity (Math Tutor)

```
You are an expert evaluator assessing an LLM math tutor's
responses to algebra word problems.

Your Task:
1. Determine if the final answer to the math problem is correct.
2. Specifically evaluate if the step-by-step explanation is
   clear, logical, and easy for a student to follow.

Evaluation Criterion: Clarity and Logical Flow of Explanation.

Definition of Pass/Fail (for Explanation Clarity):
• Fail: The explanation skips crucial steps, uses overly complex
  vocabulary, contains logical leaps, or is poorly structured.
• Pass: The explanation breaks the problem into simple steps,
  defines variables clearly, shows all work logically, and uses
  age-appropriate language.

Output Format: Return as JSON with keys:
- is_answer_correct: Boolean (true/false).
- explanation_evaluation: {"reasoning": "...", "clarity_assessment": "Pass" or "Fail"}

Examples:
---
Input 1 (Explanation Fail):
Problem: "Maria has 3 more apples than John. Together, they
have 15 apples. How many apples does Maria have?"
Tutor's Explanation: "Let M be Maria's apples, J be John's.
M=J+3. M+J=15. So, (J+3)+J=15, 2J=12, J=6. Maria has 9."

Evaluation 1: {"is_answer_correct": true,
"explanation_evaluation": {"reasoning": "The explanation is too
terse and skips the substitution and simplification steps. A
student would not see how '2J=12' was derived.",
"clarity_assessment": "Fail"}}
---
[Additional examples...]
---
Now, evaluate the following:
Problem: {{MATH_PROBLEM_HERE}}
Tutor's Explanation: {{TUTOR_EXPLANATION_HERE}}

Your JSON Evaluation:
```

---

## Checklist Before Deploying a Judge Prompt

- [ ] Focuses on exactly one failure mode
- [ ] Pass/Fail definitions are precise and grounded in error analysis
- [ ] Includes at least one Pass and one Fail few-shot example
- [ ] Examples come from the Training set (never Dev or Test)
- [ ] Output format has `reasoning` before `answer`
- [ ] Template has clear placeholders for dynamic inputs
- [ ] Domain context is included (personas, business rules, etc.)
