# NIST Code Evidence Agent

GitHub Copilot agent definitions that scan your repository for evidence of [NIST 800-53 Rev. 5](https://csf.tools/reference/nist-sp-800-53/r5/) security control implementation. The agents evaluate what controls are relevant to your codebase and assess repository-level artifacts — code, configuration, CI/CD pipelines, IaC, tests, and documentation — to produce structured findings.

> **This is not a compliance tool.** It produces evidence-based findings, not a compliance certificate.

## How It Works

The system uses two GitHub Copilot agents that run as a two-phase workflow:

### 1. Plan (`nist-evidence.plan`)

Scans your repository to build an inventory (languages, frameworks, CI/CD, IaC, security tooling, tests, auth patterns, crypto usage, input validation) and selects applicable NIST 800-53 Rev. 5 controls. Outputs a structured tasks file at `.nist-evidence/nist-evidence-tasks.md`.

### 2. Assess (`nist-evidence.assess`)

Works through the tasks file, evaluating each control against actual repository evidence. Processes controls in batches (by family or a configurable count) to stay within context limits. Produces:

- **JSON report** — `.nist-evidence/nist-evidence-report.json` with structured findings, observations, and confidence scores
- **Markdown summary** — `.nist-evidence/nist-evidence-summary.md` with tables and recommendations

Each finding includes a status (`satisfied`, `partial`, `not_satisfied`, `not_applicable`, `unknown`), confidence score, traceable observations with file paths and line ranges, and actionable recommendations.

## NIST Control Families Covered

| Family | Code | Example Controls |
|--------|------|------------------|
| Access Control | AC | AC-6, AC-14 |
| Audit and Accountability | AU | AU-2, AU-3, AU-8 |
| Configuration Management | CM | CM-2, CM-3, CM-7, CM-8 |
| Identification and Authentication | IA | IA-5 |
| Risk Assessment | RA | RA-5 |
| Security Assessment and Authorization | CA | CA-8 |
| System and Communications Protection | SC | SC-8, SC-13, SC-28 |
| System and Information Integrity | SI | SI-2, SI-3, SI-10 |
| System and Services Acquisition | SA | SA-11, SA-15 |

Only controls whose satisfaction can be meaningfully evidenced from repository artifacts are selected. Controls requiring purely organizational, physical, or operational evidence are excluded.

## Adding to Your Repository

### Option A: curl the files directly

Run the following from the root of your repository to download the agent and prompt definitions:

```bash
# Create directories
mkdir -p .github/agents .github/prompts

# Download agent definitions
curl -sL https://raw.githubusercontent.com/maxneuvians/nist-code-evidence-agent/main/agents/nist-evidence.plan.agent.md \
  -o .github/agents/nist-evidence.plan.agent.md

curl -sL https://raw.githubusercontent.com/maxneuvians/nist-code-evidence-agent/main/agents/nist-evidence.assess.agent.md \
  -o .github/agents/nist-evidence.assess.agent.md

# Download prompt definitions
curl -sL https://raw.githubusercontent.com/maxneuvians/nist-code-evidence-agent/main/prompts/nist-evidence.plan.prompt.md \
  -o .github/prompts/nist-evidence.plan.prompt.md

curl -sL https://raw.githubusercontent.com/maxneuvians/nist-code-evidence-agent/main/prompts/nist-evidence.assess.prompt.md \
  -o .github/prompts/nist-evidence.assess.prompt.md
```

### Option B: clone and copy

```bash
git clone https://github.com/maxneuvians/nist-code-evidence-agent.git /tmp/nist-agent
cp -r /tmp/nist-agent/agents/ .github/agents/
cp -r /tmp/nist-agent/prompts/ .github/prompts/
rm -rf /tmp/nist-agent
```

### Option C: Git subtree (stay in sync with updates)

```bash
git subtree add --prefix=.github/nist-agent \
  https://github.com/maxneuvians/nist-code-evidence-agent.git main --squash
```

After adding the files, add the artifact output directory to your `.gitignore`:

```bash
echo '.nist-evidence/' >> .gitignore
```

## Usage

### Interactive (VS Code / GitHub Copilot Chat)

**Plan the assessment:**

```
@nist-evidence.plan
```

```
@nist-evidence.plan Only check AU and SC families
```

**Run the assessment:**

```
@nist-evidence.assess
```

```
@nist-evidence.assess Assess in batches of 5 and publish GitHub issues for findings
```

During assessment the agent pauses between batches so you can review progress, drill into specific controls, skip families, or stop early.

### Common Interactive Commands

| Command | Description |
|---------|-------------|
| `continue` / `next` | Proceed to next batch |
| `stop` / `finish` | Finalize report with current progress |
| `explain <control_id>` | Show detailed finding with all observations |
| `show evidence for <control_id>` | List observations with file paths and excerpts |
| `what would fix <control_id>?` | Get actionable remediation steps |
| `skip <family>` | Mark remaining controls in family as unknown |
| `reassess <control_id>` | Re-evaluate a specific control |

## Output

All artifacts are written to `.nist-evidence/` by default:

| File | Description |
|------|-------------|
| `nist-evidence-tasks.md` | Assessment plan with selected controls grouped by family |
| `nist-evidence-report.json` | Structured JSON with findings, observations, and summary |
| `nist-evidence-summary.md` | Human-readable Markdown report with tables and recommendations |

### Report JSON Schema (abbreviated)

```json
{
  "schema_version": "1.0.0",
  "tool": { "name": "nist-evidence-agent", "version": "1.0.0", "run_timestamp": "..." },
  "repository": { "identifier": "owner/repo", "ref": "main", "commit_sha": "..." },
  "summary": {
    "total_controls": 20,
    "satisfied": 8,
    "partial": 6,
    "not_satisfied": 3,
    "not_applicable": 2,
    "unknown": 1
  },
  "findings": [
    {
      "control_id": "AU-2",
      "title": "Event Logging",
      "status": "partial",
      "confidence": 0.7,
      "rationale": "...",
      "recommendations": ["..."]
    }
  ],
  "observations": [
    {
      "observation_id": "AU-2_1",
      "type": "code_pattern",
      "path": "src/logger.ts",
      "polarity": "positive",
      "quality": "strong",
      "confidence": 0.85
    }
  ]
}
```

## Resuming Interrupted Assessments

The assess agent supports resumption out of the box:

- Controls already marked `[x]` in the tasks file are skipped
- The JSON report is read and merged if it exists
- Use `resume_from=<control_id>` to restart from a specific point

## File Structure

```
agents/
├── nist-evidence.plan.agent.md        # Planning agent definition
└── nist-evidence.assess.agent.md      # Assessment agent definition
prompts/
├── nist-evidence.plan.prompt.md       # Bootstrap prompt for planning
└── nist-evidence.assess.prompt.md     # Bootstrap prompt for assessment
.github/
└── agents/
    └── copilot-instructions.md        # Copilot development guidelines
```

## License

See [LICENSE](LICENSE) for details.
