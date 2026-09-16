# Contributing technical evidence

## Scope

Contributions should improve the technical record for MECHREVO Xingyao 14 / P916F-STX. Identify additional firmware revisions explicitly. Findings from related platforms remain comparative until applicability is demonstrated.

Write professional technical English. Preserve firmware terminology, raw output and implementation details. Separate observations from interpretation; avoid conversational narrative, unsupported recommendations and repeated disclaimers.

## Evidence record

For each new claim, provide:

| Field | Required information |
|---|---|
| Claim and scope | Exact platform string, BIOS version, firmware-reported EC version and relevant operating-system environment |
| Classification | Evidence class and source coverage, using the [methodology](docs/reverse-engineering-methodology.md#evidence-classification) |
| Artifact identity | Parent filename, byte size, SHA-256, extraction range or member path and address space for binary analysis |
| Recorded operation | Command or operation actually performed, initial conditions, original output and observed result |
| Effects and limits | State changed, side effects observed, untested behavior and applicable firmware revision |
| Source location | Public excerpt, artifact record or validation section; identify unavailable original material separately |

Do not invent missing fields. Upload time is not measurement time; an IFR default is not a live value; a successful getter does not validate a setter. A driver or method name does not establish user-facing semantics.

## Raw evidence and redaction

Preserve command output verbatim in evidence blocks. Explain number bases, units and interpretation outside the block. Redact serial numbers, unique device identifiers, credentials and unrelated personal information before publication; mark the location and purpose of each redaction. Do not silently alter measured values or command bytes.

Do not submit raw ROM dumps, vendor installers, proprietary executables or extracted firmware resources. Supply provenance metadata instead. Firmware-writing and unrestricted EC-access utilities are outside this documentation repository's scope.

## Corrections and conflicting results

Do not overwrite an earlier result solely because a newer source differs. Record both source identities, conflicting values and conditions. Replace a claim only when evidence supports the replacement; retain useful rejected paths and unresolved source-version discrepancies.

Documentation review or a repeated file hash is not new hardware validation. A static finding must not become Live-confirmed through editorial revision.

## Submission checks

Check internal links, table structure, number bases, units, offset context and consistency with the evidence matrix. Preserve raw evidence and place limitations at the affected interface boundary. Editorial changes do not require hardware experiments; do not run speculative hardware tests merely to complete documentation.

Update [open questions](docs/open-questions.md) for unresolved behavior and [documentation status](docs/documentation-status.md) for missing source material. Submit a pull request describing the evidence added or corrected.

Original contributions use the documentation license in [LICENSE](LICENSE). Identify third-party excerpts and provenance separately; see [NOTICE.md](NOTICE.md).
