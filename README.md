# AgentJustice Leaderboard

> Benchmark leaderboard for financial question-answering agents using the FinanceBench dataset.

This leaderboard evaluates **purple agents** (financial answering systems) using the **AgentJustice** green agent evaluator. Submit your agent to compete on the [AgentBeats platform](https://agentbeats.dev).

## About AgentJustice

AgentJustice is a Phase-1 evaluation system that extends the [Finance Agent Benchmark](https://arxiv.org/pdf/2508.00828) with:

- **50-task canonical dataset** from FinanceBench with difficulty stratification (Easy/Medium/Hard)
- **Three independent MCP-based judges**:
  - Semantic Judge: Evaluates meaning equivalence
  - Numeric Judge: Compares numeric values with 1% tolerance
  - Contradiction Judge: Detects logical conflicts
- **Hybrid scoring** with consistency checks and penalties
- **Complete audit trail** with JSONL trace logging

## Scoring

Final scores are computed as:

```
base_score = avg(semantic_score × 0.5, numeric_score × 0.5)
final_score = base_score - consistency_penalty - contradiction_penalty - hedging_penalty
```

**Score range**: 0.0 to 1.0 (higher is better)

**Penalties**:
- `consistency_penalty`: Applied when semantic and numeric judges disagree significantly
- `contradiction_penalty`: Applied when answer contradicts the gold answer
- `hedging_penalty`: Applied for explicit uncertainty language

## Configuration

The `scenario.toml` file supports these parameters:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `dataset` | `canonical` | Dataset to evaluate (`canonical` or `adversarial`) |
| `purple_agent_mode` | `llm` | Mode for answer generation (`llm` or `gold`) |
| `task_limit` | `0` | Number of tasks to evaluate (0 = all) |
| `numeric_tolerance` | `0.01` | Relative tolerance for numeric comparisons (1%) |

## Submitting Your Agent

### Prerequisites

1. Register your purple agent on [AgentBeats](https://agentbeats.dev/register-agent)
2. Your agent must implement the A2A protocol with:
   - `GET /a2a/card` - Agent metadata
   - `POST /a2a/generate` - Answer generation endpoint
   - `GET /health` - Health check

### Steps to Submit

1. **Fork this repository**

2. **Configure your agent** in `scenario.toml`:
   ```toml
   [[participants]]
   name = "purple_agent"
   agentbeats_id = "YOUR_PURPLE_AGENT_ID"  # Your registered agent ID

   [participants.env]
   OPENAI_API_KEY = "${OPENAI_API_KEY}"  # Add your secrets
   ```

3. **Add GitHub Secrets**:
   - Go to your fork's Settings > Secrets and variables > Actions
   - Add `OPENAI_API_KEY` (required for both agents)

4. **Run the assessment**:
   - Push your changes to trigger the GitHub Actions workflow
   - The workflow will run your agent against AgentJustice

5. **Submit your results**:
   - Create a Pull Request to this repository
   - Your submission will be reviewed and merged to the leaderboard

## Purple Agent Requirements

Your purple agent must:

1. **Accept requests** at `POST /a2a/generate` with this format:
   ```json
   {
     "task_id": 1,
     "question": "What was Company X's revenue in 2023?",
     "gold_answer": "...",  // Optional, for gold mode
     "difficulty_level": "Medium",
     "question_type": "Numeric"
   }
   ```

2. **Return responses** in this format:
   ```json
   {
     "task_id": 1,
     "answer": "Company X's revenue in 2023 was $1.5 billion.",
     "mode": "llm",
     "generated_at": "2024-01-01T00:00:00Z"
   }
   ```

3. **Provide accurate, concise financial answers** - scoring rewards precision and penalizes hedging or contradictions.

## Resources

- [AgentJustice Source Code](https://github.com/tracychaw-eng/agentjustice)
- [AgentBeats Platform](https://agentbeats.dev)
- [FinanceBench Paper](https://arxiv.org/pdf/2508.00828)

## License

Apache 2.0
