# Prompts

Each agent has a system prompt that defines its role and output format, and a user prompt that carries the research and the audience. The full text of every prompt is in [`workflow/multi_ai_agents.json`](../workflow/multi_ai_agents.json) under each agent's `systemMessage`.

## Shared user prompt

All three agents receive the same input:

```
Article Content:
{{ $('Aggregate').item.json.data.toJsonString() }}

Target Audience:
{{ $('Set Search Fields').item.json.targetAudience }}
```

`Article Content` is the JSON array of `{ title, raw_content }` objects produced by the `Aggregate` node.

## System prompt structure

The three system prompts follow the same five-part layout:

| Section | Purpose |
| --- | --- |
| **System Role** | Gives the model a specific persona (LinkedIn content creator, Twitter content creator, blog writer) |
| **Task Specification** | A numbered list of requirements for the output |
| **Specifics and Context** | Why the task matters and what a good result looks like |
| **Examples** | One or two worked input/output pairs (few-shot prompting) |
| **Reminders** | The most important constraints, repeated at the end |

Repeating the key constraints at the end and showing a worked example both help keep the output in the expected format.

## Per-agent requirements

| | LinkedIn | X | Blog Writer |
| --- | --- | --- | --- |
| Length | Under 3,000 characters | Within 280 characters | Two paragraphs |
| Tone | Concise, human, conversational | Approachable, engaging | Professional yet approachable |
| Format | Plain text, frequent line breaks, mobile-friendly | Single short message | Introduction flowing to conclusion |
| Emojis | 1–2 | 1–2 | None specified |
| Hashtags | 3–5 | 1–3 | None |
| Call to action | Required | Required | Not required |
| Audience | Tailored to the target audience | Tailored to the target audience | General audience |
| Output | Post text only | Post text only | Article text |
| Examples given | 1 | 1 | 2 |

## Notes

- The LinkedIn and X prompts both end with an instruction to output only the post text. This matters because the output is written straight into a spreadsheet cell, where any preamble such as "Here is your post:" would end up in the content.
- The Blog Writer prompt asks for a general audience, although the target audience is still passed in the user prompt. Updating its task specification to reference the target audience would bring it in line with the other two.
- Length limits are requested in the prompts but not checked by the workflow.
