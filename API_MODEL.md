# npm Trusted Publisher (OIDC) — API model extracted from npm CLI 12.1.0 (public tarball)

Source: https://registry.npmjs.org/npm/-/npm-12.1.0.tgz
  lib/commands/trust/{index,github,gitlab,circleci,list,revoke}.js, lib/trust-cmd.js, lib/utils/oidc.js

## Endpoints
| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/-/package/<escapedName>/trust` | npm session/Bearer (write? read?) | list configs |
| POST | `/-/package/<escapedName>/trust` | npm session + 2FA (OTP) | body = ARRAY of trust configs |
| DELETE | `/-/package/<escapedName>/trust/<id>` | npm session + 2FA | revoke |
| POST | `/-/npm/v1/oidc/token/exchange/package/<escapedName>` | `Authorization: Bearer <id_token>` | id_token as the auth credential |

## POST/PUT /-/package/<pkg>/trust body shape (from optionsToBody + createConfigCommand)
```json
[{
  "type": "github",
  "claims": { "repository": "owner/repo",
              "workflow_ref": { "file": "publish.yml" },
              "environment": "prod" },
  "permissions": ["createPackage", "createStagedPackage"]
}]
```
- type: github | gitlab | circleci
- gitlab claims: `project`, `workflow_ref.file`(pipeline file), `environment`
- circleci claims: `organization_id/project_id/pipeline_definition_id/vcs_origin/context_ids` (UUIDs)
- **ONLY ONE config per package** — creating a second errors out.

## Exchange flow (lib/utils/oidc.js)
- audience = `npm:<registry hostname>` i.e. `npm:registry.npmjs.org`
- idToken passed as `//registry.npmjs.org/:_authToken`
- runs ONLY when ci-info detects GITHUB_ACTIONS | GITLAB | CIRCLE (or NPM_ID_TOKEN set)
- provenance auto-enabled when id_token payload `repository_visibility == "public"`

## Auth requirements for trust management
- write access to package; 2FA enabled at account level; GAT/USERNAME_PASSWORD rejected
- bulk: first request 2FA, "skip for 5 min" -> ~80 packages per window
