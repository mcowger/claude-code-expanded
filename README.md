![Claude Code Action responding to a comment](https://github.com/user-attachments/assets/1d60c2e9-82ed-4ee5-b749-f9e021c85f4d)

# Claude Code Expanded

A fork of [anthropics/claude-code-action](https://github.com/anthropics/claude-code-action) with expanded event support, including `push` events for release automation and other tag-based workflows.

This fork tracks upstream closely (rebased on main regularly) while adding features that aren't yet in the official release.

## Key Differences from Upstream

| Feature | Upstream | This Fork |
|---------|----------|-----------|
| `push` event support | ❌ | ✅ |
| Bot actor restrictions | Requires `allowed_bots` | Relaxed for automation events |
| Release automation | workflow_dispatch only | Works with on: push: tags: |

## When to Use This Fork

Use this fork when you need to run Claude Code on:
- Tag pushes (for automated release notes)
- Direct push events (for automation workflows)
- Any automation where the actor is a bot (no `allowed_bots` workaround needed)

## Usage

Replace `anthropics/claude-code-action@v1` with `mcowger/claude-code-expanded@main`:

```yaml
- uses: mcowger/claude-code-expanded@main
  with:
    prompt: |
      Generate release notes for this tag...
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Example: Release Notes on Tag Push

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: mcowger/claude-code-expanded@main
        with:
          prompt: |
            Generate release notes for ${{ github.ref_name }} based on
            commits since the previous tag.
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

      - uses: softprops/action-gh-release@v1
        with:
          body: ${{ steps.claude.outputs.result }}
```

This replaces the two-stage workflow (release-trigger.yml → workflow_dispatch) that upstream requires.

## Upstream Documentation

For all other features, configuration, and usage patterns, see the [upstream documentation](https://github.com/anthropics/claude-code-action):

- [Solutions Guide](https://github.com/anthropics/claude-code-action/blob/main/docs/solutions.md)
- [Migration Guide](https://github.com/anthropics/claude-code-action/blob/main/docs/migration-guide.md)
- [Setup Guide](https://github.com/anthropics/claude-code-action/blob/main/docs/setup.md)
- [Usage Guide](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md)
- [Configuration](https://github.com/anthropics/claude-code-action/blob/main/docs/configuration.md)
- [Security](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md)
- [FAQ](https://github.com/anthropics/claude-code-action/blob/main/docs/faq.md)

## Keeping Up to Date

This fork automatically attempts to sync with upstream daily at 6 AM UTC via a [GitHub Actions workflow](.github/workflows/sync-upstream.yml).

### Automatic Sync

- **Success:** Changes are automatically rebased and pushed to `main`
- **Failure:** An issue is created/updated with details about the conflict requiring manual intervention

### Manual Update

If you need to update manually or resolve conflicts:

```bash
git fetch upstream
git rebase upstream/main
# Resolve any conflicts
git push --force-with-lease
```

### Handling Conflicts

The push event support is concentrated in a few files:
- `src/github/context.ts` - `push` added to `AUTOMATION_EVENT_NAMES`
- `src/modes/detector.ts` - Push event mode detection
- `docs/custom-automations.md` - Documentation

When rebasing, conflicts are usually in `context.ts` where upstream may have changed the event handling. The fix is typically accepting upstream's changes but ensuring `push` remains in `AUTOMATION_EVENT_NAMES`.

## License

MIT - See [LICENSE](LICENSE)
