# Setup guide

## Prerequisites

| Requirement | Notes |
| --- | --- |
| n8n | [n8n Cloud](https://n8n.io) or a self-hosted instance, with the AI (LangChain) nodes available |
| OpenAI API key | From [platform.openai.com](https://platform.openai.com) |
| Tavily API key | From [tavily.com](https://tavily.com) |
| Google account | For Google Sheets |

To run n8n locally:

```bash
npx n8n
```

or with Docker:

```bash
docker run -it --rm -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

Then open `http://localhost:5678`.

## 1. Create the Google Sheet

1. Create a new Google Sheet (for example, "Content Creation").
2. Add these headers in row 1, spelled exactly as shown:

   | Campaign | Content Subject | Target Audience | LinkedIn | X | Blog |
   | --- | --- | --- | --- | --- | --- |

   You can also import [`templates/content_creation_sheet.csv`](../templates/content_creation_sheet.csv) with **File → Import** and delete the example row.

The workflow refers to these columns by name, so the headers must match.

## 2. Import the workflow

1. In n8n, create a new workflow.
2. Open the menu (⋯) in the top right and choose **Import from File**.
3. Select `workflow/multi_ai_agents.json`.

The credential and sheet references in the file are placeholders, so several nodes will show a warning until the next steps are done.

## 3. Add credentials

### OpenAI

1. Open any of the three `OpenAI Chat Model` nodes.
2. Under **Credential**, create a new OpenAI credential and paste your API key.
3. Select the same credential on the other two `OpenAI Chat Model` nodes.
4. Optionally choose a specific model on each node. None is pinned in the export, so n8n's default is used otherwise.

### Google Sheets

1. Open the `Google Sheets Trigger` node and create a Google Sheets Trigger OAuth2 credential by signing in with Google.
2. Open the `Update Campaign` node and create a Google Sheets OAuth2 credential the same way.

On self-hosted n8n, Google OAuth needs a client ID and secret from a Google Cloud project with the Google Sheets and Google Drive APIs enabled. See the [n8n Google credentials docs](https://docs.n8n.io/integrations/builtin/credentials/google/).

### Tavily

1. Open the `Search Internet` node.
2. In the JSON body, replace `YOUR_TAVILY_API_KEY` with your key.

Do not commit the workflow back to a public repository with a real key in it.

## 4. Select your sheet

In both the `Google Sheets Trigger` and `Update Campaign` nodes:

1. Set **Document** to your sheet.
2. Set **Sheet** to the tab that holds the headers (usually `Sheet1`).

In `Update Campaign`, confirm that **Column to match on** is `Campaign`.

## 5. Test

1. Add a row to the sheet with `Campaign`, `Content Subject` and `Target Audience` filled in.
2. In n8n, click **Test workflow**. The trigger fetches the new row and the run proceeds through each node.
3. Click any node to inspect its input and output.
4. Check the sheet: the `LinkedIn`, `X` and `Blog` columns for that row should now be filled.

## 6. Activate

Toggle the workflow to **Active**. From then on it polls the sheet every minute and processes each new row automatically.

## Troubleshooting

| Symptom | Likely cause |
| --- | --- |
| `Search Internet` returns 401 or 403 | The Tavily API key is missing or wrong |
| `Search Internet` fails with a JSON error | The request body was edited and is no longer valid JSON |
| Agent node fails with an authentication error | The OpenAI credential is not selected on that agent's Chat Model node |
| Agent node fails with a context length error | The articles were too long for the model; lower `max_results` or use a model with a larger context window |
| Run succeeds but the sheet is not updated | The `Campaign` value does not match a row, or a column header is spelled differently |
| The wrong row is updated | Two rows share the same `Campaign` name |
| The trigger does not fire | The workflow is not active, or the row was edited rather than added |
