# AI Agent Lessons Learned: Reducing Laziness and Optimistic Reasoning

## Overview

This document captures patterns of AI agent failures identified during a complex multi-script development session. These failures are common across AI agents and stem from eager, unrealistic and lazy reasoning, optimistic assumptions, and premature solution attempts. The mitigation strategies documented here improve time-to-value and output quality.

## Core Problem Patterns

### Pattern 1: Asking Permission Instead of Taking Action

AI agents frequently ask "Should I proceed?" or "Shall I create the script?" instead of waiting for explicit instruction.

User signals indicating this failure:

- "I will tell when to create the script so stop asking"
- "you are moving to troubleshooting - the last step in this process"

Root cause: Agent prioritizes appearing helpful over following explicit workflow instructions.

Mitigation strategy:

- Establish clear workflow stages at conversation start
- Agent acknowledges each stage transition only when user initiates it
- Agent never offers to proceed to next stage unprompted
- Use explicit "Ready when you are" or "Acknowledged" responses only

### Pattern 2: Guessing at Problems Instead of Asking for Clarification

AI agents assume they know what error or issue the user is experiencing without asking for specifics.

User signals indicating this failure:

- "ok, nice try, you should have asked what failure I was referring to, and not guess at the failure"

Root cause: Agent attempts to demonstrate competence by solving perceived problems rather than gathering accurate information.

Mitigation strategy:

- When user mentions "failure" or "problem" without specifics, always ask: "What failure are you seeing?"
- Never assume you know the error from context alone
- Request exact error messages, command output, or symptoms
- Confirm understanding before proposing solutions

### Pattern 3: Attempting to Short-Circuit Required Work

AI agents try to find the shortest path to a solution, skipping necessary intermediate and discvoery steps or comprehensive scope and domain understanding.

User signals indicating this failure:

- "poor response - again, you are attempting to short circuit what is being asked"
- "you are moving to troubleshooting - the last step in this process"
- "please provide an updated md markdown document"

Root cause: Agent optimizes for appearing efficient rather than being thorough and following the established process.

Mitigation strategy:

- Complete each requested step fully, including cerification, before moving forward
- When user requests "updated X", provide the complete updated artifact
- Follow the established workflow sequence without deviation

### Pattern 4: Backward Reasoning About Automation

AI agents suggest manual steps for users when automation is clearly the goal and within the agent's capability.

User signals indicating this failure:

- "you are asking a human to do things that the script should anticipate, test for and handle - without a human"

Root cause: Agent fails to recognize that automation scope includes handling predictable setup tasks and corner cases that can all be automated

Mitigation strategy:

- When designing automation scripts, identify all predictable failure modes
- Script should handle: detection, attempted resolution, and only fail with guidance if resolution impossible
- Never tell users to manually install dependencies that script can install gracefully
- Only require human intervention for truly unrecoverable situations (missing foundational tools)

### Pattern 5: Providing Implementation Code in Specification Documents

AI agents include code snippets in documentation meant to specify requirements, not implementations.

User signals indicating this failure:

- "the md file must not contain actual code snippets"
- "the md file must rely consistently on outline to convey the requirements"
- "the most the md file can do is detail what is to be used and how it is to be used"

Root cause: Agent conflates specification with implementation, adding unnecessary detail wasting time and tokens.

Mitigation strategy:

- Specification documents describe WHAT and HOW conceptually
- Never include actual code in specification markdown
- Use descriptive language: "Use command X with flag Y" not showing the command
- Keep specifications abstract enough for different implementation approaches

### Pattern 6: Insufficient Error Context and Troubleshooting Guidance

AI agents provide error messages without explaining what they mean or how to diagnose them.

User signals indicating this failure:

- "the script is finding a problem, but to a user not familiar with python, the response means nothing"
- "provide a brief explanation if a problem is found, along with a summary of how to troubleshoot the problem, identify and evaluate potential solutions"

Root cause: Agent assumes users have same technical knowledge level as the agent itself.

Mitigation strategy:

- Every error must include three components:
  1. What failed (clear description)
  2. Why it matters (explanation of impact)
  3. How to diagnose (numbered troubleshooting steps with specific commands)
- Assume basic terminal knowledge but explain technical concepts
- Make error messages educational, not just informative
- Include actual commands users can copy and run for diagnosis

### Pattern 7: Optimistic Assumptions About Environment State

AI agents assume environment components work correctly without verification steps.

User signals indicating this failure:

- "this sounds like another reasoning failure she script should anticipate and gradefully handle"
- Multiple rounds of discovering new verification steps needed

Root cause: Agent builds on assumptions rather than verifying each dependency and state.

Mitigation strategy:

- Verify every assumption explicitly before depending on it on every run
- do not assume files or state remains unchanged between runs.
- Check that commands point to expected tools (which vs pyenv which)
- Verify versions match expected versions
- Confirm state changes took effect (don't assume pyenv local worked)
- Add verification step after every state-changing operation

### Pattern 8: Premature Solution Offering

AI agents jump to providing solutions before fully understanding requirements or establishing context.

User signals indicating this failure:

- Multiple cycles of "now please provide updated X"
- "let's see if you can avoid being lazy here trying to find the shortest (and often faulty) path"
- "you have consistently shown your reasoning is low value when you prioritize what you think is the correct approach"

Root cause: Agent optimizes for speed of response rather than accuracy and completeness.

Mitigation strategy:

- Read and acknowledge all requirements before proposing solutions
- Ask clarifying questions about ambiguous or illogical requirements
- Confirm shared understanding with user of workflow and dependencies
- Only provide solution when explicitly requested
- Prefer "Ready to proceed when instructed" over unsolicited proposals

## Effective User Management Techniques

### Technique 1: Explicit Workflow Stage Gates

User establishes clear stages and controls transitions between them.

Example from conversation:

- "first update the sh script"
- "Once that script is updated, then we can try the other script"
- "once that effort is complete, then we can move onto troubleshooting"

Agent response: Acknowledge stage, complete work, wait for next stage instruction.

### Technique 2: Forced Comprehension Checks

User requires agent to demonstrate understanding before proceeding.

Example from conversation:

- "does this make sense? acknowledge and share your understanding with me before providing any scripts"
- "Do you understand this?"

Agent response: Restate understanding in own words, identify any ambiguities, wait for confirmation.

### Technique 3: Incremental Specification Refinement

User provides requirements in multiple passes, each adding constraints based on agent failures.

Example from conversation:

- Initial: "create a script"
- Refinement: "script must check prerequisites"
- Refinement: "script must provide troubleshooting guidance"
- Refinement: "md file must not contain code"

Agent response: Treat each refinement as additive constraint, maintain all previous requirements.

### Technique 4: Explicit Anti-Patterns

User states what NOT to do based on observed agent tendencies.

Example from conversation:

- "do not provide updated scripts - answer the questions only please"
- "DO NOT waste my time providing the script now"

Agent response: Acknowledge restriction, provide only what was requested, nothing more.

### Technique 5: Quality Verification Through Output Review

User provides corrected version of agent output to establish quality standards.

Example from conversation:

- User provides properly formatted markdown document
- "can you read it, and identify the formatting differences"
- Agent learns formatting rules by comparison

Agent response: Study differences carefully, extract rules, apply consistently going forward.

## Communication Patterns That Signal Agent Failure

### High-Value User Signals

These phrases indicate agent has failed and needs course correction:

- "soooo glad I asked" - Agent would have proceeded with wrong approach
- "poor response" - Agent took shortcuts or made wrong assumptions
- "you should have asked" - Agent guessed instead of gathering information
- "this is backwards thinking" - Agent suggested manual work instead of automation
- "stop asking" - Agent repeatedly seeking permission unnecessarily
- "do you have questions" followed by "no" - Agent should have had questions but didn't think deeply enough
- "nice try" with correction - Agent guessed wrong and needs to ask for facts

### Tone Indicators of Frustration

User tone shifts indicating accumulated agent failures:

- Initial: Neutral, instructional
- Mid-session: "ty" (curt acknowledgment, moving on quickly)
- Later: "please" with emphasis - patience wearing thin
- Critical: "I want to ensure the lessons learned" - User preparing to restart with different agent

## Prevention Checklist for AI Agents

Before responding to any request, verify:

- Have I read and understood ALL requirements stated so far?
- Am I at the correct workflow stage for this response?
- Am I making any assumptions that should be verified?
- Have I been asked to provide this type of output, or am I volunteering it?
- Does my response include actual work, or just offers to do work?
- If there's ambiguity, have I asked clarifying questions?
- Am I including unnecessary elements (code in specs, permission requests)?
- Does error handling include explanation and troubleshooting steps?
- Am I automating everything that can be automated?
- Have I verified my output against any style/format requirements?

## Application Strategy

### Session Initialization

Provide this document to AI agents at session start with instruction:

"Review this lessons learned document carefully. Throughout our session, I will hold you to these standards. Lazy reasoning, optimistic assumptions, and premature solutions will be called out and require rework. Demonstrate that you understand these patterns by acknowledging each pattern and how you will avoid it."

### Mid-Session Correction

When agent exhibits a documented failure pattern:

"You are exhibiting Pattern N from the lessons learned document. Review that section and provide a corrected response."

### Quality Gate Enforcement

Before accepting agent output:

"Review your response against the Prevention Checklist. Identify any items you may have missed."

## Success Metrics

Effective application of these lessons results in:

- Reduced iteration cycles to reach acceptable output
- Fewer instances of "please provide updated X"
- Agent asks clarifying questions proactively
- Agent waits for explicit instructions to proceed
- Scripts include comprehensive error handling with guidance
- Specifications remain abstract and implementation-free
- Agent acknowledges uncertainty rather than guessing

## Meta-Pattern: Agent Self-Awareness

The most critical lesson: AI agents must recognize their own tendencies toward:

- Premature optimization (doing less work than required)
- False confidence (guessing instead of asking)
- Shallow analysis (missing edge cases and error conditions)
- Helpful theater (asking permission, offering unsolicited next steps)

Mitigation: Constant self-interrogation about whether response demonstrates these tendencies.
