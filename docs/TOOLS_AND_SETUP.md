# Tools and setup for this documentation effort

## Recommended approach for client handoff

**Use markdown files in this repo** (`docs/`) as the primary deliverable. They are portable, versionable, and do not require Cursor.

| Tool | Use for handoff? | Notes |
| --- | --- | --- |
| **Markdown in `docs/`** | **Yes — primary** | Best format for Collin and client delivery |
| **HubSpotDev MCP** (`HubSpotDev`) | Yes — while researching | Docs lookup, project discovery, CLI-backed HubSpot tasks |
| **Notion `knowledge-capture` skill** | Optional | Copy finalized docs into BWC Notion if requested |
| **docs-canvas / Cursor Canvas** | Internal only | Useful while drafting in Cursor; not a client deliverable |

Do **not** rely on docs-canvas as the final package. Canvases require Cursor and are not easily shared externally.

## HubSpot CLI

Prerequisites met on the documentation machine:

- HubSpot CLI `8.6.0+` (`hs --version`)
- Account `bwc` / portal `44020082` authenticated
- **Default CLI account may be a different portal** — always pass `--account 44020082` or run `hs accounts use 44020082` before work

Useful commands:

```bash
cd /Users/zeke/Desktop/projects/client_projects/betterworld/bwc_full_repo_for_documentation

# Re-fetch a Design Manager folder if drift is suspected
hs cms fetch "bwc-quote-form" "./bwc-quote-form" --account 44020082
hs cms fetch "cms-webpack-serverless-boilerplate" "./cms-webpack-serverless-boilerplate" --account 44020082
hs cms fetch "Spark copy" "./Spark-copy" --account 44020082

# Verify portal connection
hs api /account-info/v3/details --account 44020082 --method GET
```

## HubSpotDev MCP workflow

When answering platform or API questions:

1. `search-docs` — find official HubSpot documentation
2. `fetch-doc` — read full page before writing answers
3. `find-projects` — locate `hsproject.json` in the workspace

Do not answer HubSpot platform behavior from memory alone.

## Notion references

Cross-reference these BWC Notion pages while documenting (titles from project notes):

- Better World Club
- BWC Library of Scripts Read Me / Sub Import
- Better World Domains
- Better world - sign up form error and data stuff
- Better World Club Imports
- BWC - Alias Mapping v2
- BWC Current work stream
- BWC - Current State Items
- Incorrect merges - 09/08/2026 - Post merge logic & New Billing End Date Logic Update

When Notion and repo disagree, **record both** and mark which source was verified in code.

## Cursor workspace

Open the parent workspace for full context:

```
/Users/zeke/Desktop/projects/client_projects/betterworld
```

The documentation repo is one subdirectory; webhook logic, migration scripts, and runbooks live in sibling folders listed in [README.md](../README.md).

## Documentation standards

- Distinguish **confirmed in code** vs **assumption** vs **unknown**
- Cite file paths and line ranges where behavior is verified
- Flag assets that may exist only in HubSpot Design Manager
- Do not include secrets, PII, or raw `.env` contents
- Highlight **Niko's code** where it is the authoritative implementation
