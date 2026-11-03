# Publishing

How the Bluebox team publishes this repository to plugin marketplaces and registries.

## MCP Registry and GitHub MCP Registry

[server.json](server.json) describes the Bluebox MCP server for the official [MCP Registry](https://registry.modelcontextprotocol.io) as `io.github.bluebox-hq/bluebox`. The [GitHub MCP Registry](https://github.com/mcp) lists servers from the MCP Registry.

The server sets no headers, so clients sign in to Bluebox through OAuth.

### Publish by hand

You need to be a member of the `bluebox-hq` GitHub organization. The registry grants the `io.github.bluebox-hq/*` namespace to members.

1. Install `mcp-publisher`:

   ```bash
   brew install mcp-publisher
   ```

   Or download a release binary from [modelcontextprotocol/registry](https://github.com/modelcontextprotocol/registry/releases).

2. Bump `version` in `server.json`. The registry rejects a version that is already published. Then merge the change to `main`.

3. On an up-to-date `main`, validate, sign in and publish:

   ```bash
   mcp-publisher validate
   mcp-publisher login github
   mcp-publisher publish
   ```

   `login github` opens a GitHub device login in your browser.

4. Check that the new version is live:

   ```bash
   curl -s "https://registry.modelcontextprotocol.io/v0/servers?search=io.github.bluebox-hq/bluebox"
   ```

5. First publish only: email `partnerships@github.com` and ask them to list `io.github.bluebox-hq/bluebox` on github.com/mcp. GitHub curates that list. After the first listing, new versions sync automatically.

Don't run `mcp-publisher status` to check a publish. It changes the lifecycle state of the published server.

If `publish` fails with HTTP 403, the name doesn't match a namespace you own. The name must start with `io.github.bluebox-hq/`, and the comparison is case-sensitive. Check that you're a member of `bluebox-hq`.

### Automate publishing later

[validate-server-json.yml](.github/workflows/validate-server-json.yml) already validates `server.json` on every pull request that changes it. To publish from CI as well, add a workflow that signs in with GitHub OIDC. OIDC needs no stored secret.

Before you add the workflow:

1. **Create the environment first.** In **Settings → Environments**, create `mcp-registry-publish` and add required reviewers. The OIDC token can publish anything under `io.github.bluebox-hq/*`, not only this server. If a workflow names an environment that doesn't exist, GitHub creates it on the first run without protection rules.
2. **Limit who can create `v*` tags.** Add a tag ruleset, because anyone who can push a `v*` tag starts a publish.
3. **Decide who bumps the version.** The skills sync replaces only `skills/`, so it doesn't change `server.json`. Either bump `version` and tag by hand, or extend the Bluebox release process to do it.

Then add `.github/workflows/publish-mcp.yml`:

```yaml
name: Publish to MCP Registry

on:
  push:
    tags: ["v*"]
  workflow_dispatch:

permissions:
  contents: read

env:
  MCP_PUBLISHER_VERSION: v1.8.1
  # sha256 of mcp-publisher_linux_amd64.tar.gz, from registry_1.8.1_checksums.txt in the release
  MCP_PUBLISHER_SHA256: a06c9096dcb9727c13555b6be26c7effa707b01f06a4c561ba7a3635443cf2cc

jobs:
  publish:
    runs-on: ubuntu-latest
    # The OIDC token can publish to the whole io.github.bluebox-hq/* namespace,
    # so the job waits for a reviewer of this environment.
    environment: mcp-registry-publish
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7
      - name: Check the tag matches the server.json version
        if: startsWith(github.ref, 'refs/tags/v')
        run: |
          tag="${GITHUB_REF#refs/tags/v}"
          actual=$(jq -r .version server.json)
          if [ "$tag" != "$actual" ]; then
            echo "FAIL tag v$tag does not match server.json version \"$actual\""
            exit 1
          fi
      - name: Install mcp-publisher
        run: |
          curl -fsSLo mcp-publisher.tar.gz "https://github.com/modelcontextprotocol/registry/releases/download/${MCP_PUBLISHER_VERSION}/mcp-publisher_linux_amd64.tar.gz"
          echo "${MCP_PUBLISHER_SHA256}  mcp-publisher.tar.gz" | sha256sum -c -
          tar xzf mcp-publisher.tar.gz mcp-publisher
      - name: Publish server.json
        run: |
          ./mcp-publisher login github-oidc
          ./mcp-publisher publish
```

To release, bump `version` in `server.json` (for example to `1.0.1`) and merge the change. Then push the matching tag with `git tag v1.0.1 && git push origin v1.0.1`, and approve the job in **Actions**. When you update `mcp-publisher`, update `MCP_PUBLISHER_VERSION` and `MCP_PUBLISHER_SHA256` together.

## Plugin marketplaces

The plugin files need no publish step. Each client reads them from this repository:

| Client | Files | Listing |
|---|---|---|
| Claude Code | `.claude-plugin/`, `.mcp.json` | Self-hosted now (`claude plugin marketplace add bluebox-hq/skills`). Directory listing: submit the repository once at <https://claude.ai/admin-settings/directory/submissions/plugins/new>. After approval, the catalog syncs nightly. |
| Cursor | `.cursor-plugin/plugin.json`, `mcp.json` | Submit once at <https://cursor.com/marketplace/publish>. |
| Google Antigravity | `.antigravity-plugin/plugin.json`, `mcp_config.json` | Install by URL now (`agy plugin install https://github.com/bluebox-hq/skills`). Marketplace: Google's interest form. The Marketplace needs a higher `version` in `.antigravity-plugin/plugin.json` for every update. |
| Kiro | `POWER.md`, `mcp.json` | Install by repository URL now. Directory listing: submit once at <https://kiro.dev/powers/submit>. |
