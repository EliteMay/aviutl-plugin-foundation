# Foundation Contract

Generated projects contain `.aviutl-plugin-foundation.json`.

Example:

```json
{
  "schemaVersion": 1,
  "foundationVersion": "0.1.0",
  "foundationCommit": "<commit>",
  "template": "native-filter",
  "managedPaths": [
    ".github/workflows/foundation-build.yml",
    "cmake/foundation/",
    "tests/foundation/"
  ]
}
```

The Hub may update only those managed paths after checking repository identity, branch and clean worktree state.
