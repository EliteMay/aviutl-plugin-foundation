# Foundation Rules

1. Foundation is shared infrastructure, not a plugin repository.
2. Generated plugin source remains plugin-owned.
3. Updates only touch paths declared in `managedPaths`.
4. Existing non-empty repositories are never overwritten by starter generation.
5. Generated repositories record foundation version and commit.
6. Plugin-specific README, roadmap, source and assets are protected.
7. New SDK behavior must be checked against the current AviUtl2 SDK before a template is declared compatible.
8. Build success does not replace real AviUtl2 verification.
