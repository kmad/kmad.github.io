---
title: "Auditing a Codebase for 87 cents in 50 lines of code using RLMs"
---

Before we begin: *Of course* this doesn't replace a proper (human) audit or security testing process, nor should it. But this was a fun afternoon experiment.

I wanted to try DSPy’s new **Recursive Language Model (RLM)** module for something deeper than just codebase documentation, as inspired by [@dbreunig](https://gist.github.com/dbreunig/bab62de16f173f040bb51453b32c6aa2). Instead of generating docs over a codebase, why not something a little more substantive, like a security audit?

Here’s the approach: clone the OWASP Damn Vulnerable Serverless Application (DVSA) (a purposely insecure project) and run RLM against it. In theory it should explore the codebase, break down the target areas and hand off to sub-lms for further analysis. The useful part here is that we have a `LESSONS` folder that we can compare our results to (we'll remove this from the context before starting so it can't cheat).

After modifying Drew's script, I ran it with `kimi-k2.5` and `grok-4` on openrouter. Interestingly, grok struggled with the RLM format and the report kept getting cut off, so the results below are for kimi only. The results were pretty impressive considering I changed about 3 lines of code, and the whole thing is less than 50! It's worth considering some of this may be in the training data, but for a simple script it seems this would be viable to build off of.


### The Setup

The entire pipeline is almost embarrassingly simple:

```python
import dspy
import os
from typing import Any

# LM Setup - must have OPENROUTER_API_KEY set
lm = dspy.LM("openrouter/moonshotai/kimi-k2.5", max_tokens=16000)
lm_mini = dspy.LM("openrouter/moonshotai/kimi-k2.5", max_tokens=16000)
dspy.configure(lm=lm)

# DSPy Signature & Program
class CodeScanner(dspy.Signature):
  """
  Review the provided application source code in detail. 
  Focus specifically on identifying security vulnerabilities, 
  insecure coding patterns, and other areas of concern.
  """
  source_tree: dict[str, Any] = dspy.InputField()
  documentation: str = dspy.OutputField(description="Generated markdown documentation.")

def load_source_tree(root_dir: str) -> dict[str, Any]:
    ...

source_root = "~/dev/DVSA/"
source_tree = load_source_tree(source_root)
del source_tree['CONTENT'] # Remove the 'lessons' we will compare against.

code_scanner = dspy.RLM(
    CodeScanner, 
    max_iterations=35,
    sub_lm = lm_mini,
    verbose=True
)

# Load and run
source_tree = load_source_tree("~/dev/DVSA/")
result = code_scanner(source_tree=source_tree)
```

That's it. The `dspy.RLM` handles all the recursive decomposition where the module breaks down the codebase analysis into subtasks, delegates to the sub-language model, and synthesizes results automatically. In this case the sub-lm is the same as the main.

## Results

It did pretty well! Here's an excerpt of part of the trajectory:
```
Reasoning: I've found several critical vulnerabilities so far:

1. **admin_shell.js**: Remote Code Execution (RCE) via `eval(cmd)` - CRITICAL
2. **feedback_uploads.py**: Command injection via `os.system()` with user-controlled filename - CRITICAL
3. **user_create.py**: Privilege escalation via user-supplied 'Admin' attribute
4. **admin_get_orders.py**: Code injection via `eval(fe)` on DynamoDB FilterExpression - CRITICAL
5. **admin_update_orders.py**: Insecure deserialization via `jsonpickle.decode()` - CRITICAL
6. **template.yml**: Weak password policy (6 chars, no complexity), overly permissive CORS, DynamoDB CRUD on '*' (all tables)

I need to continue examining:
- The rest of admin_update_orders.py for the full deserialization flow
- Other order functions for IDOR vulnerabilities
- Client-side components for XSS vulnerabilities
- Check for any hardcoded secrets or sensitive data exposure
Code:
```
```python
# Continue examining admin_update_orders.py for deserialization vulnerability
print("=" * 60)
print("ADMIN_UPDATE_ORDERS.PY - Full file (deserialization)")
print("=" * 60)
admin_update_full = source_tree['backend']['functions']['admin']['admin_update_orders.py']
print(admin_update_full[3000:])
```



Unsurprisingly, it didn't catch everything, and rerunning this a few times during the experiment led to different issues being caught. I'm sure an ensemble approach would do just fine here.

**Summary:**
| Category | Count |
|----------|-------|
| Fully Caught | 3 (#3, #4, #7) |
| Partially Caught | 3 (#1, #5, #10) |
| Completely Missed | 4 (#2, #6, #8, #9) |


**Lesson Breakdown:**
| Lesson | Topic | Status     | Notes                                                         |
|--------|-------|------------|---------------------------------------------------------------|
| #1     | Event Injection              | Partial | Caught S3 command injection; missed node-serialize code injection          |
| #2     | Broken Authentication        | Missed  | JWT bypass & open billing API not documented                                |
| #3     | Sensitive Info Disclosure    | Caught  | Admin receipt access via S3                                                |
| #4     | Insecure Cloud Config        | Caught  | S3 public write & command injection                                        |
| #5     | Broken Access Control        | Partial | Caught IDOR/privilege escalation; missed payment bypass                    |
| #6     | Denial of Service            | Missed  | Billing concurrency abuse not mentioned                                    |
| #7     | Over-Privileged Functions    | Caught  | Comprehensive IAM policy violations                                        |
| #8     | Logic Vulnerabilities        | Missed  | Race condition/TOCTOU not documented                                       |
| #9     | Vulnerable Dependencies      | Missed  | node-serialize, node-jose, shell-quote not mentioned                       |
| #10    | Unhandled Exceptions         | Partial | Generic error disclosure noted; specific examples missed                   |



## What the Model Missed

The RLM approach failed to detect 4 out of the 10 lessons identified in the security review. Specifically, the following categories were completely missed (which kimi also helped summarize):

| Completely Missed | Description |
|-------------------|-------------|
| #2                | Broken Authentication: Issues like JWT bypass and open billing API vulnerabilities were not flagged. |
| #6                | Denial of Service: The model overlooked potential for abuse via billing concurrency, which could enable attackers to deliberately exhaust system resources. |
| #8                | Logic Vulnerabilities: Race conditions and time-of-check-to-time-of-use (TOCTOU) flaws, which often require awareness of timing and system state transitions, were not identified. |
| #9                | Vulnerable Dependencies: The presence of known-vulnerable libraries (such as node-serialize, node-jose, shell-quote) went undetected by the static analysis pipeline. |

The results are instructive. Each of these misses highlights areas where RLM-based static analysis struggles: vulnerabilities that hinge on nuanced runtime behavior, system workflow, or external package risk—issues that often escape detection without dynamic techniques or explicit vulnerability lists. These seem entirely solvable with just a bit of scaffolding and tool calling.


## Key Observations

**Cost efficiency.** *The results above cost $0.865*. That's insane! The RLM was configured for a maximum of 35 iterations, which can be increased dramatically. The 80 cents might even include a few false-starts as I was working out bugs.

**Static vs. Dynamic analysis gap.** The 60% catch rate for static vulnerabilities is pretty good for 50 lines of code. The misses require either a more nuanced view, running the program, or both.

**Zero prompt engineering.** I didn't write a single example or optimize prompts. The RLM module handled decomposition strategy automatically. Compare this to traditional approaches where you'd craft specific prompts for each vulnerability type (which you still could). The models are simply that good.


**Composability.** The generated documentation followed a consistent structure (per-file analyses → project-wide concerns → recommendations) without explicit formatting instructions. The RLM's recursive nature naturally produces hierarchical outputs which works well for codebases.


The full writeup from the model can be found [here](../assets/rlm/rlm_security_writeup.html) - the content is from the model, I just spruced up the markdown into an HTML file.

---

*Experiment run January 2026 using kimi-k2.5 via OpenRouter. Results will vary with different codebases and models.*
