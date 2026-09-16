# PKI Architect Capstone

**In-progress portfolio snapshot** — this project is still under active development.

**Architect:** Lucy Arukwe

**Role:** PKI Architect
**Organization (scenario):** Sentinel Federal Services
**Industry:** Government Contractor
**Engagement length:** Weeks 17–24

## Project overview

Establish controlled trust across enclaves and partners with provable separation of duties, documented exceptions, and controlled release signing.

## Business & security problem

Sentinel Federal Services operates controlled enclaves alongside corporate IT and exchanges signed material with partners. Exceptions have been granted verbally, restricted paths are inconsistently enforced, and release signing has drifted outside the controlled process.

## Constraints the design had to satisfy

- Separation of duties between requestor and approver is mandatory.
- Enclave-to-corporate paths are restricted and must be explicitly authorized.
- Partner trust must be scoped; a partner credential must not be usable enterprise-wide.
- Signing releases requires controlled approval.

## Required outcomes

- Enclave-scoped trust with documented exceptions.
- Provable separation of duties in approvals.
- Controlled release signing evidence.
- Deterministic workload evidence across restricted paths.

## Change & incident response

Stage 4 change/incident work has not yet been completed in this portfolio snapshot.

## Skills demonstrated

- _No demonstrable work has been recorded in this snapshot yet._

## Repository contents

```
README.md                                  this overview
GITHUB-INSTRUCTIONS.md                     how to publish this package
docs/architecture.md                       zones, components, CA hierarchy, key protection
docs/certificate-strategy.md               pools, profiles, ownership, approvals
docs/lifecycle-and-status.md               issuance, renewal, revocation, CRL/OCSP
docs/workload-testing.md                   workload runs, failures, diagnosis
docs/change-and-incident-response.md       Stage 4 response (when completed)
docs/final-defense-summary.md              checkpoint and defense statements
evidence/evidence-summary.md               evidence index
data/project-summary.json                  public-safe summary data
```

## About this package

This portfolio artifact is a sanitized representation of a CyberVisionaries Institute PKI Architect Capstone project.

It contains no account identifiers, no assignment or project identifiers, no signatures, and no instructor-only material. It is **not** the signed backup file used to restore capstone work, and it cannot be imported into the capstone workspace.

Snapshot generated 2026-09-11.

