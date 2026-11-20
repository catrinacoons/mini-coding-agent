# Project 2: Agent Skills Component for mini-coding-agent
Group 11: Catrina Coons, Yulisa Medrano


----
# Handoff to Yulisa for 2nd Part of the Project: 

I've finished the first half of the SDLC: intent, spec, plan, plus a README draft, and my video. They're in intent/agent-skills/ and README.md is drafted. Everything from here on is yours: implementation into mini-coding-agent.py, skills from the Lab 5 and testing it, finish the README, and your half of the video. Treat spec.md as the contract and plan.md as your guide, it will walk you through the implementation. The requirement numbers (R1–R21) are meant to show up in your test names and commit messages. You can then import the skills from Lab 5, just be sure to change the file path and add the keywords. Tune the agent with a few test prompts to see if it triggers the skills from keywords correctly, then have it build the vacuum robot environment just like we made Claude Code do for Lab 5. 

If you disagree with a design decision, go ahead and change it. Just update spec.md/plan.md in the same commit. Lmk if you have any questions!

----

mini-coding-agent already has five of the six coding-agent components from class. This 
fork adds the missing one: agent skills. The agent finds skill folders such as skills/
reward-design/SKILL.md, checks each request against them using keyword similarity, loads 
the best match into its prompt, and can also load a skill on its own during a task. Every 
prompt, similarity check and executed skill is logged. We demonstrate it by having the 
agent build and train a Q-learning vacuum robot (the Lab).

## SDLC documents

intent.md - The problem, the desired outcome, constraints, open questions
spec.md - Numbered requirements R1–R21, the design, areas of concern
plan.md - Files that change, order of work (citing R#), risks, proof

## How the skills component works

The skills component from Lab 5 was integrated into the mini-coding-agent. The file structure within .claude was maintained; there was an addition of a lab/ folder containing the q_learning.py and vacuum_env.py that the mini coding agent utilizes.
The flow of the AI agent:
1. Starting from the intent.md, specs.md, and plan.md, the parent agent has the initial plan outlined with all of the instructions and conventions. Indicating that skills are reachable in the specified directories.
2. The child agent inherits the instructions/workspace, thresholds, and maximum skills. This automatically matches the delegated tasks with the skills ask() -> check_skills().
3. Then the activated skills write depth-aware JSONL events for each subagent.

   - The user submits a prompt
   - Skill discovery and matching: SkillLibrary ranks available skills against the prompt.
   - Relevant skills activate: reward-design instructions become available to the agent.
   - The Ollama model reasons about the task, deciding whether to read files, run tools, or answer.
   - Tool execution, like read_file, run_shell, load_skill, or delegate.
   - Final answer and logs, including the validation report plus JSONL skill events.

  
 ### File Structure
 ```
.claude/
└── skills/
│    ├── grid-validation/
│    │   └── SKILL.md
│    ├── transition-testing/
│    │   └── SKILL.md
│    └── reward-design/
│        └── SKILL.md
├──intent
│  └── agent-skills/
│      ├──intent.md
│      ├──specs.md
│      ├──plan.md
├──lab/
│    ├── q_learning.py
│    ├── vacuum-env.py
├──tests/
│    ├── test_skills.py
│    ├── test_mini_coding_agent.py
│
├── mini_coding_agent.py
```
### Requirements are unchanged from original Mini-Coding_Agent directions

&nbsp;
# Mini-Coding-Agent

This folder contains a small standalone coding agent:

- code: `mini_coding_agent.py`
- CLI: `mini-coding-agent`

It is a minimal local agent loop with:

- workspace snapshot collection
- stable prompt plus turn state
- structured tools
- approval handling for risky tools
- transcript and memory persistence
- bounded delegation

The model backend is currently based on Ollama.

<a href="https://magazine.sebastianraschka.com/p/components-of-a-coding-agent">
  <img src="https://substack-post-media.s3.amazonaws.com/public/images/49b97718-57f4-4977-99c8-8ad5c4d32af3_1548x862.png" width="500px">
</a>

<br>

**[The detailed tutorial: Components of a Coding Agent](https://magazine.sebastianraschka.com/p/components-of-a-coding-agent)**


&nbsp;
## Six Core Components

<a href="https://magazine.sebastianraschka.com/p/components-of-a-coding-agent">
  <img alt="Six core components of a coding agent" src="https://sebastianraschka.com/images/github/mini-coding-agent/six-components.webp" width="500px">
</a>

This coding harness is organized around six practical building blocks:

1. **Live repo context**  
   The agent collects stable workspace facts upfront, such as repo layout, instructions, and git state.
2. **Prompt shape and cache reuse**  
   A stable prompt prefix, which is separate from the changing request, transcript, and memory so repeated model calls can reuse the static parts efficiently.
3. **Structured tools, validation, and permissions**  
   The model works through named tools with checked inputs, workspace path validation, and approval gates instead of free-form arbitrary actions.
4. **Context reduction and output management**  
   Long outputs are clipped, repeated reads are deduplicated, and older transcript entries are compressed to keep prompt size under control.
5. **Transcripts, memory, and resumption**  
   The runtime keeps both a full durable transcript and a smaller working memory so sessions can be resumed while preserving important state via working memory.
6. **Delegation and bounded subagents**  
   Scoped subtasks can be delegated to helper agents that inherit enough context to help (but operate within limits).

&nbsp;
## Requirements

You need:

- Python 3.10+
- Ollama installed
- an Ollama model pulled locally

Optional:

- `uv` for environment management and the `mini-coding-agent` CLI entry point

This project has no Python runtime dependency beyond the standard library, so you can run it directly with `python mini_coding_agent.py` if you do not want to use `uv`.

&nbsp;
## Install Ollama

Install Ollama on your machine so the `ollama` command is available in your shell.

Official installation link: [ollama.com/download](https://ollama.com/download)

Then verify:

```bash
ollama --help
```

Start the server:

```bash
ollama serve
```

In another terminal, pull a model. Example:

```bash
ollama pull qwen3.5:4b
```

Qwen 3.5 model library:

- [ollama.com/library/qwen3.5](https://ollama.com/library/qwen3.5)

The default in this project is `qwen3.5:4b`. If you have sufficient memory, it is worth trying a larger model such as `qwen3.5:9b` or another larger Qwen 3.5 variant. The agent just sends prompts to Ollama's `/api/generate` endpoint.

&nbsp;
## Project Setup

Clone the repo or your fork and change into it:

```bash
git clone https://github.com/rasbt/mini-coding-agent.git
cd mini-coding-agent
```

If you forked it first, use your fork URL instead:

```bash
git clone https://github.com/<your-github-user>/mini-coding-agent.git
cd mini-coding-agent
```



&nbsp;
## Basic Usage

Start the agent:

```bash
cd mini-coding-agent
uv run mini-coding-agent
```

Without `uv`, run the script directly:

```bash
cd mini-coding-agent
python mini_coding_agent.py
```

By default it uses:

- model: `qwen3.5:4b`
- approval: `ask`


### Grid Validation

Validates the vacuum robot's 4×4 grid, starting position, obstacle, pit, and goal.

```bash
python3 mini_coding_agent.py --model qwen3.5:4b --max-steps 15 \
"Use grid-validation to check lab/vacuum_env.py. Read the file once. Verify the 4x4 grid, start (0,0), obstacle (1,1), pit (3,1), and goal (3,3). Do not use shell commands. Give a short PASS/FAIL report."
```

### Transition Testing

Checks stochastic movement probabilities, boundary handling, obstacle blocking, and terminal EXIT behavior.

```bash
python3 mini_coding_agent.py --model qwen3.5:4b --max-steps 15 \
"Use transition-testing to check lab/vacuum_env.py. Read the file once. Verify movement probabilities 0.8, 0.1, and 0.1, boundary handling, obstacle blocking, and EXIT behavior. Do not use shell commands. Give a short PASS/FAIL report."
```

### Reward Design

Validates the reward structure, including the living reward, pit penalty, goal reward, and EXIT action.

```bash
python3 mini_coding_agent.py --model qwen3.5:4b --max-steps 15 \
"Use reward-design to check lab/vacuum_env.py. Read the file once. Verify living reward -0.04, pit -1, goal +10, and EXIT behavior. Do not use shell commands. Give a short PASS/FAIL report."
```

### Run Q-Learning

Runs the vacuum robot's Q-learning implementation directly.

```bash
python3 lab/q_learning.py
```

To have the agent execute Q-learning and summarize the results:

```bash
python3 mini_coding_agent.py --model qwen3.5:4b --max-steps 15 \
"Use transition-testing to inspect lab/vacuum_env.py. Then run python3 lab/q_learning.py once using run_shell. Report the actual Q-learning results. Do not invent results or repeat commands."
```

### Run Automated Tests

Run the complete test suite:

```bash
python3 -m pytest tests/ -v
```

Run only the agent skills tests:

```bash
python3 -m pytest tests/test_skills.py -v
```

The implementation previously passed all 24 automated tests.

### View Skill Activation Logs

The agent records skill matching and activation events in `.mini-coding-agent/skill_log.jsonl`.

To view activated skills:

```bash
grep '"event": "skill_executed"' .mini-coding-agent/skill_log.jsonl
```

### Interactive Commands

Start the agent:

```bash
python3 mini_coding_agent.py --model qwen3.5:4b
```

Inside the interactive agent, use `/skills` to list available skills or enter a natural-language task.

Example:

```text
mini-coding-agent> Validate the vacuum robot grid using grid-validation.
```

### Notes

- Skills are automatically selected based on prompt matching.
- Focused prompts are generally more reliable with `qwen3.5:4b`.
- The agent may request approval before running shell commands.
- A skill activation message confirms selection, but successful task completion requires a final response.
- The automated tests provide reproducible verification of the implementation.


For a concrete usage example, see [EXAMPLE.md](EXAMPLE.md).

&nbsp;
## Approval Modes

Risky tools such as shell commands and file writes are gated by approval.

- `--approval ask`
  prompts before risky actions (default and recommended)
- `--approval auto`
  allows risky actions automatically, including arbitrary command execution and file writes by the model; use only with trusted prompts and trusted repositories
- `--approval never`
  denies risky actions

Example:

```bash
uv run mini-coding-agent --approval auto
```



&nbsp;
## Resume Sessions

The agent saves sessions under the target workspace root in:

```text
.mini-coding-agent/sessions/
```

Resume the latest session:

```bash
uv run mini-coding-agent --resume latest
```


Resume a specific session:

```bash
uv run mini-coding-agent --resume 20260401-144025-2dd0aa
```


&nbsp;
## Interactive Commands

Inside the REPL, slash commands are handled directly by the agent instead of
being sent to the model as a normal task.

- `/help`
  shows the list of available interactive commands
- `/memory`
  prints the distilled session memory, including the current task, tracked files, and notes
- `/session`
  prints the path to the current saved session JSON file
- `/reset`
  clears the current session history and distilled memory but keeps you in the REPL
- `/exit`
  exits the interactive session
- `/quit`
  exits the interactive session; alias for `/exit`

&nbsp;
## Main CLI Flags

```bash
uv run mini-coding-agent --help
```

Without `uv`:

```bash
python mini_coding_agent.py --help
```

CLI flags are passed before the agent starts. Use them to choose the workspace,
model connection, resume behavior, approval mode, and generation limits.

Important flags:

- `--cwd`
  sets the workspace directory the agent should inspect and modify; default: `.`
- `--model`
  selects the Ollama model name, such as `qwen3.5:4b`; default: `qwen3.5:4b`
- `--host`
  points the agent at the Ollama server URL (usually not needed); default: `http://127.0.0.1:11434`
- `--ollama-timeout`
  controls how long the client waits for an Ollama response (usually not needed); default: `300` seconds
- `--resume`
  resumes a saved session by id or uses `latest`; default: start a new session
- `--approval`
  controls how risky tools are handled: `ask`, `auto`, or `never`; default: `ask`
- `--max-steps`
  limits how many model and tool turns are allowed for one user request; default: `6`
- `--max-new-tokens`
  caps the model output length for each step; default: `512`
- `--temperature`
  controls sampling randomness; default: `0.2`
- `--top-p`
  controls nucleus sampling for generation; default: `0.9`

&nbsp;
## Example

See [EXAMPLE.md](EXAMPLE.md)

&nbsp;
## Notes & Tips

- The agent expects the model to emit either `<tool>...</tool>` or `<final>...</final>`.
- Different Ollama models will follow those instructions with different reliability.
- If the model does not follow the format well, use a stronger instruction-following model.
- The agent is intentionally small and optimized for readability, not robustness.
