# Setup and Configuration

This is a public, generalized version of the system. No real credentials, URLs, or contact details are included. Every identifier in the workflow files is a placeholder, so wire in your own services before running anything.

## Prerequisites

| Service | What it is for | Credential / value you provide |
|---|---|---|
| n8n | Runs the orchestrator, the specialist agents, and the shared sub-workflows | self-hosted or cloud instance |
| OpenAI | Agent reasoning, intent routing, and embeddings | API key |
| Anthropic Claude | Additional LLM used in parts of the orchestrator | API key |
| Supabase | Relational data plus the vector store for retrieval | project URL and service key |
| PostgreSQL | Chat memory and deal / session state | connection string |
| Redis | Cache and debounce lock for the inbound path | connection details |
| Evolution API (WhatsApp) | Inbound and outbound messaging channel | instance name and API key |
| Google Calendar | Test-drive slots and events | OAuth credential |
| Gmail | Test-drive confirmation and reschedule emails | OAuth credential |
| Google Drive | Knowledge documents feeding the vector store | OAuth credential |
| External pricing / financing API | Vehicle pricing and financing simulation | endpoint URL and any token |

## Placeholders to replace

The cleaned workflows use clearly marked tokens. Search and replace these with your own values:

- `https://example.com/...` stands in for every external URL and webhook endpoint (pricing API, credit-score API, messaging webhooks).
- `{{EVOLUTION_INSTANCE}}` the Evolution API instance name used by the messaging nodes.
- `{{FIPE_API_TOKEN}}` the token for the external vehicle-pricing API.
- `{{GOOGLE_CALENDAR_ID}}` the calendar id used by the Google Calendar nodes.
- `{{CONTACT_PHONE}}`, `{{CONTACT_EMAIL}}`, `{{DEALERSHIP_ADDRESS}}` the contact block shown in confirmation emails.
- `{{WORKFLOW_ID}}` and any cached sub-workflow names referenced by Execute Workflow and tool-workflow nodes; re-point these at the workflows you import.

Credentials on the nodes are blank by design. Open each node that talks to a service and select your own n8n credential.

## Import order

1. Import the shared sub-workflows first: `customer_lookup_provisioning_subworkflow.json` and `closing_finalize_subworkflow.json`.
2. Import the specialist agents: `test_drive_scheduling_agent.json`, `financing_simulation_agent.json`, `negotiation_agent.json`.
3. Import `orchestrator_inbound_router.json` last, then point its Execute Workflow and tool-workflow calls at the workflows you imported above.

## Steps

1. Create the credentials in n8n for each service in the prerequisites table.
2. Provision the database schema the workflows expect (customer table, chats with the active-agent field, active-deals, deal context, chat memory, and the vector store table in Supabase).
3. In every node that calls a service, select your own credential and replace the placeholder values listed above.
4. In the orchestrator, set the sub-workflow references to the agents and shared steps you imported.
5. Connect your messaging webhook (Evolution API) to the orchestrator entry point.
6. Activate the orchestrator. The specialist agents run as sub-workflows, so they do not need to be active on their own. The test-drive reminder path uses a schedule trigger, so activate that workflow if you want batched reminders.

## Notes

- The internal field and table names in SQL and expressions (for example the active-agent field and the deal tables) are kept as-is so the workflows stay internally consistent. Rename them across all files together if you prefer different names.
- Never commit real credentials. Keep secrets in n8n credentials or environment variables, not in node parameters.
