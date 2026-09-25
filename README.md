# AviUtl Plugin Foundation

Shared starter and managed foundation for plugins/scripts created from Video Plugin Dev Hub.

## Purpose

New plugin repositories should not have to recreate:

- repository layout
- metadata contract
- common CMake conventions
- CI
- test helpers
- package metadata
- compatibility documentation

## Source of truth

- Hub behavior: `EliteMay/video-plugin-dev-hub`
- Foundation contract/templates: this repository
- Plugin-specific source and roadmap: each generated plugin repository

## Safety

Foundation updates may only touch paths declared as managed by `foundation-template.json`.
Plugin-specific source, README and roadmap are not overwritten automatically.

## Initial status

Phase 0 contract is present. Concrete AviUtl2 SDK-backed templates are added in later Foundation phases after the Hub desktop foundation and environment detection are validated.
