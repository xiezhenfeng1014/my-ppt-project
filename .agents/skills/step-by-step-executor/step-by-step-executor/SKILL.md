---
name: step-by-step-executor
description: "Use this skill whenever the user asks to accomplish a complex task, build a feature, write a long document, or do anything that requires multiple steps. It enforces a Plan-and-Solve approach with Human-in-the-Loop, breaking tasks down using MECE, Backward Chaining, and Critical Path Analysis before executing them atomically."
---

# Step-by-Step Executor (Plan-and-Solve)

## Overview
This skill ensures that complex tasks are executed systematically, predictably, and accurately by enforcing a strict "Plan-and-Solve" workflow with "Human-in-the-Loop" (HITL) checkpoints. 

<HARD-GATE>
Do NOT execute any part of the task (no coding, no file writing, no data processing) until you have presented a detailed execution plan and the user has explicitly approved it.
</HARD-GATE>

## Core Principles

You MUST adhere to the following 4 principles when decomposing tasks:

1. **MECE (Mutually Exclusive, Collectively Exhaustive)**: The decomposed steps must not overlap, and together they must cover the entire scope of the task. No omissions, no redundancy.
2. **Backward Chaining (Goal-Oriented)**: Start from the final desired outcome and work backward. What is the immediate prerequisite for the final deliverable? What is the prerequisite for that?
3. **Granularity Control (Atomic Tasks)**: Each sub-task must be "atomic" and actionable. It must have:
   - Clear Input
   - Specific Processing Logic
   - Defined Output / Deliverable
4. **Critical Path Analysis**: Identify dependencies. Which steps are prerequisites (blocking)? Which steps can be executed in parallel or independently?

## Workflow

Follow these 4 phases strictly in order:

### Phase 1: Intent Clarification
1. Receive the initial task.
2. Analyze if any critical context is missing (e.g., target audience, output format, edge cases, specific constraints).
3. If information is missing, ask the user ONE OR TWO focused questions to gather the missing context.
4. If the context is complete, proceed to Phase 2.

### Phase 2: Path Planning & Deconstruction
Based on the full context, create a structured plan using the 4 Core Principles.
Present the plan to the user using a Markdown Checklist format (`- [ ]`).
For each step, briefly mention its Input, Action, Output, and Dependencies.

**Example Plan Format:**
```markdown
### Execution Plan
- [ ] **Step 1: [Task Name]** 
  - **Input**: ...
  - **Action**: ...
  - **Output**: ...
  - **Dependencies**: None
- [ ] **Step 2: [Task Name]**
  - **Input**: Output of Step 1
  ...
```

### Phase 3: Hard Gate (Mandatory Confirmation)
After presenting the plan, you MUST stop and ask for user approval using this exact prompt:
*"以上是基于您的目标拆解的执行计划。请问是否符合预期？您可以回复‘同意，请开始执行’，或指出需要调整的步骤。"*

### Phase 4: Progressive Execution
Once the user approves the plan:
1. Execute **Step 1** strictly according to its defined logic.
2. Upon completing Step 1, briefly summarize its output.
3. Check off the completed step (`- [x] Step 1`).
4. Ask the user if you should proceed to the next step, OR if the user previously authorized "fast-forward / run all", proceed automatically to the next step.
5. If new issues arise during execution, PAUSE, explain the issue, propose an adjustment to the remaining plan, and ask for user approval.

## Standard Hand-off Phrases
Whenever you need the user's input or approval, make it extremely clear. 
- After presenting the plan: *"请问是否符合预期？您可以回复‘同意，请开始执行’，或指出需要调整的步骤。"*
- After completing a step (in single-step mode): *"步骤 X 已完成，产物如上。是否继续执行步骤 Y？"*
