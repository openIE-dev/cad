# License

openie-cad is **free to use** — both via the hosted MCP at
`https://cad.openie.dev/mcp` and locally as a binary. The source
code is not yet open; it lives in a private repository while the
substrate is still iterating fast.

## Hosted use (`cad.openie.dev`)

Free for any use, with no signup. Rate limits and per-session
"joules" accounting apply (see `/api/joules` for visibility). No
warranty.

## Self-hosted binaries

The binary distribution is on the roadmap. License terms for the
binary will be published with the first release. Until then, contact
the openie-dev team if you have a self-hosted requirement.

## Documentation

This repository (`openIE-dev/cad`) — the documentation site, the
example projects, and the recipe markdown — is **MIT licensed**.
Contributions are welcome via pull request; see
<https://github.com/openIE-dev/cad>.

## Third-party algorithms

The substrate implements published algorithms from the academic
literature; each one is cited on its [algorithm theory
page](./algorithms/lcis-bga-escape.md):

| Algorithm | Paper | License posture |
|---|---|---|
| LCIS BGA escape | Kong/Yan/Ozdal ICCAD 2007 | Algorithm is public-domain prior art; openie-cad's clean-room implementation has no encumbrance. |
| SALT shallow-light Steiner | Chen et al. ICCAD 2017 | Same. |
| PathFinder negotiation | McMurchie-Ebeling FPGA 1995 | Same; public-domain since 1995. |
| IQN-ILS coupling | Degroote 2009 / Chourdakis 2021 | preCICE itself is LGPLv3; openie-cad's MIT-clean re-derivation does not link or wrap preCICE. |
| Djordjevic-Sarkar dielectric | Svensson-Djordjevic 2001 | Published formula. |
| Cannonball-Huray roughness | Simonovich 2015/2019 (simplified from Huray 2010) | Published formula. |
| FDTD NTFF | Balanis textbook §6.7 | Published formula. |
| Aitken's Δ² | Aitken 1926 | Classical. |
