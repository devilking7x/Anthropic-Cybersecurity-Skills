# Standards and References — Supply Chain Security

Real standards, frameworks, and tool documentation for the procedures in `SKILL.md`. Nothing here is a placeholder.

## Standards and frameworks

- NIST SP 800-161 Rev. 1 — Cybersecurity Supply Chain Risk Management Practices for Systems and Organizations: https://csrc.nist.gov/pubs/sp/800/161/r1/final
- SLSA — Supply-chain Levels for Software Artifacts: https://slsa.dev/
- SPDX (ISO/IEC 5962:2021) — Software Bill of Materials format: https://spdx.dev/
- CycloneDX — OWASP SBOM format: https://cyclonedx.org/
- OWASP SCVS — Software Component Verification Standard: https://owasp.org/www-project-software-component-verification-standard/

## Signing and provenance

- Sigstore / Cosign — keyless signing and verification: https://docs.sigstore.dev/cosign/overview/
- Sigstore Rekor — transparency log: https://docs.sigstore.dev/rekor/overview/
- in-toto — supply chain integrity framework: https://in-toto.io/

## Automation and auditing

- Dependabot: https://docs.github.com/en/code-security/dependabot
- Renovate: https://docs.renovatebot.com/
- Package-manager audits: `npm audit`, `pnpm audit`, `pip audit`, `cargo audit`, `govulncheck`, `bundle audit`
