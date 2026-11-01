# Vercel MCP 403 reproduction

Verified on 2026-11-01 for the `mrpan/game-hub` project. No credentials or share links are included.

## Target

- Team: `pan-65f0`
- Team ID: `team_PzVh91xYNoxyyjGdeV3CdHYi`
- Project ID: `prj_xF1W56lt6AAgsQJ1LKI5pevtpyEQ`
- Tested deployment: `dpl_6L4zbrdy5SVNm3ZyzLGQxxbuvgMM`
- Production domain: `game-hub-rho-ashy.vercel.app`

## Controlled read-only reproduction

The same connected account, deployment ID, and operation were used in both calls:

```json
{"tool":"list_deployment_aliases","arguments":{"id":"dpl_6L4zbrdy5SVNm3ZyzLGQxxbuvgMM"}}
```

Result: success; returns all three aliases, including the production domain.

```json
{"tool":"list_deployment_aliases","arguments":{"id":"dpl_6L4zbrdy5SVNm3ZyzLGQxxbuvgMM","teamId":"team_PzVh91xYNoxyyjGdeV3CdHYi"}}
```

Result: 403 `forbidden`, operation `GET /v2/deployments/{id}/aliases`, resource `aliasProject`.

Failure request ID: `iad1:hnd1:iad1:sfo1::xgfgm-1793535309303-6d0192a3f418`.

`get_project` and `list_projects` exhibit an analogous optional-team-selector failure. Without the selector, the returned project accountId matches the team ID above. The Vercel dashboard also confirms this project's ownership and a Ready production deployment. Renewing team consent did not change the result.

## Web fetch failure

`web_fetch_vercel_url` fails at stage `read_protection_bypass`, on its internal `GET /v2/deployments/{id}/aliases` lookup, before returning the target page. Its input schema exposes only `url`; callers cannot change its internal team selector. The independently reproduced selector failure is consistent with this failure, but the remote helper's source and raw outgoing request are not available, so its implementation is not proven.

The error message suggests widening permissions. This is not supported by the controlled comparison: the same connection can already read the target project, deployment, and aliases when the optional team selector is omitted.

## Safe operational workaround

For direct MCP reads, use exact resource IDs without optional team selectors and verify returned ownership. For page verification use the authenticated browser with user permission. Do not disable Deployment Protection or generate persistent bypass credentials to work around a failed metadata lookup.

## Remote fix needed

The MCP maintainer should inspect optional team-selector encoding and authorization-context handling. In the page helper, resolve and verify the deployment's owning account before choosing a scope. Add regression coverage comparing omitted and matching explicit team selectors for scoped connections. The fix must not silently choose a different account or fall back across an actual permission denial.

This repository cannot patch the remote Vercel MCP service. This document is a prepared reproduction report; it has not been sent to Vercel support.
