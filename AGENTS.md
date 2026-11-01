# Game Hub working conventions

All new games belong under `games/<game-slug>/`; add their entry to the root `index.html`.

## Deployment identity

- GitHub repository: `mrpan/game-hub`
- Production URL: https://game-hub-rho-ashy.vercel.app/
- Vercel project: `game-hub`
- Project ID: `prj_xF1W56lt6AAgsQJ1LKI5pevtpyEQ`
- Owning team: `pan-65f0`
- Owning account ID: `team_PzVh91xYNoxyyjGdeV3CdHYi`
- Production branch: `main`

## Vercel MCP diagnostic finding (2026-11-01)

The current connected Vercel MCP exhibits a reproducible scope-parameter issue:

- `get_project` with the exact project ID plus optional `teamId` returns 404.
- `get_project` with the exact project ID and no optional team selector succeeds and returns the owning account ID above.
- Explicit `slug` also fails; `list_projects` with no optional team selector lists both this project and `little-box-warehouse`.
- `get_deployment`, `list_deployments`, and `list_deployment_aliases` succeed without optional team selectors.

For this connection, omit optional `teamId` and `slug` when targeting the exact project/deployment ID. Verify the returned project ID and `accountId` before a consequential action. If that verification fails or the connection changes, stop and recheck the target; do not assume another default team is equivalent.

This is a verified workaround, not a fix to the remote MCP implementation. Do not interpret the scoped-call 404 alone as a missing project or ask the owner to repeatedly widen permissions.

`web_fetch_vercel_url` separately returns 403 at its internal protection-bypass alias lookup, even though a direct `list_deployment_aliases` call succeeds. Use a user-authorized browser for page verification when necessary; do not disable deployment protection to work around it.
