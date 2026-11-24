# Roadmap and project vision

## Purpose

Construct2to3ProjConverter is intended to convert Construct 2 projects into Construct 3 projects programmatically, through a CLI, including their add-ons where feasible. Conversion should be repeatable, inspectable, and honest about unsupported features.

This repository documents and will implement a **project-level conversion pipeline**. It is not simply a wrapper around an add-on converter: it discovers the add-ons used by the source project, resolves those add-ons to their Construct 3 counterparts when available, obtains the required add-on packages, and delegates individual add-on conversion to a compatible conversion tool.

## Planned end-to-end workflow

1. **Inspect the C2 project.** Parse project metadata and project files to discover every referenced plugin, behavior, effect, and other add-on type. Build a normalized inventory containing the source identifier, display name (when available), add-on category, and occurrences/references in the project.
2. **Resolve add-ons in UniStore.** For each discovered Construct 2 add-on, query UniStore using its name and/or identifier. Compare results instead of assuming a name match is definitive. Record candidate matches, confidence, identifiers, source links, and whether the candidate is a C2 or C3 add-on.
3. **Review and resolve ambiguity.** Exact ID matches should be preferred when the IDs are known to be comparable; otherwise name matching can be assisted by metadata and manual confirmation. Handle renamed add-ons, forks, duplicate names, missing listings, and multiple candidates explicitly. Do not silently substitute an unrelated add-on.
4. **Acquire matched packages.** Download the resolved add-on package individually, one at a time (or with a configurable concurrency limit), verify integrity where metadata or checksums are available, and retain provenance. The downloader must respect UniStore access rules and package licenses.
5. **Convert add-ons using c2-addon-assistant.** Feed each acquired C2 add-on into the external `c2-addon-assistant` conversion workflow, based on the discovered relationship to `C2C3AddonConverter` and the `TobyReeno/c2-addon-assistant` project. Treat each add-on as an independent conversion job. Keep logs, results, and errors per package; avoid allowing one failed add-on to stop the entire project conversion.
6. **Integrate converted add-ons.** Place successful C3 add-ons in the output project using the Construct 3 add-on/project conventions. Maintain a manifest mapping original C2 add-on identifiers to resolved UniStore packages and conversion outputs.
7. **Convert the project itself.** Convert project structures, references, and supported data. Rewrite references to successfully converted add-ons as appropriate, preserve compatible content, and identify unresolved dependencies.
8. **Report and validate.** Produce a conversion report listing converted items, unchanged/compatible items, missing add-ons, ambiguous matches, conversion failures, unsupported project features, warnings, and any manual migration steps.

## Proposed components

- **Project scanner:** reads C2 project files and discovers add-on references.
- **UniStore resolver:** searches by ID/name, compares candidate metadata, and records decisions.
- **Package fetcher:** downloads packages safely and tracks source/provenance.
- **Add-on conversion adapter:** runs `c2-addon-assistant` (and its conversion engine) for one add-on at a time.
- **Project converter:** handles project files, data structures, and reference rewriting.
- **Installer/integrator:** stages converted C3 add-ons into the target project.
- **Report generator:** emits machine-readable and human-readable migration reports.

## CLI direction (proposal)

Exact command names and options are not finalized. A future CLI may expose a staged workflow such as:

- `scan`: list C2 add-ons and project dependencies.
- `resolve`: find UniStore candidates and generate a reviewable resolution manifest.
- `fetch-addons`: download resolved packages.
- `convert-addons`: run conversion for packages.
- `convert-project`: convert the project using the manifest and conversion results.
- `validate`: check the output and generate a report.
- A single `convert` command may orchestrate all stages, with flags to pause for review, resume, or emit a dry-run report.

These commands are design ideas, not implemented CLI promises.

## Error handling and resumability

- Use per-add-on status records (discovered, unresolved, matched, downloaded, converted, integrated, failed).
- Persist the manifest and logs so a conversion can resume without repeating completed work.
- Make stages idempotent where possible and avoid overwriting the user's source project.
- Distinguish fatal project errors from add-on-specific failures.
- Preserve failed downloads and conversion diagnostics in a safe, clearly marked workspace for troubleshooting, without executing untrusted content.
- Offer dry-run mode before modifying or generating project output.

## Scope and non-goals

- This plan does not promise that every C2 add-on can be converted automatically or that every converted add-on will be behaviorally identical in C3.
- UniStore matching is a dependency-resolution process, not a guarantee of compatibility.
- Conversion should not execute arbitrary add-on code as part of discovery or matching.
- Add-on licenses and distribution permissions remain applicable to packages, generated output, and any redistribution.
- User review may be required for ambiguous dependency matches and conversion results.

## Roadmap phases

1. Document the design and research relevant C2/C3 project and add-on formats.
2. Implement project scanning and dependency inventory.
3. Implement UniStore search/matching and an editable resolution manifest.
4. Implement package acquisition with provenance and integrity checks.
5. Integrate the c2-addon-assistant conversion adapter and per-add-on job handling.
6. Implement project conversion and add-on installation/reference rewriting.
7. Add validation, reporting, tests, and resume/dry-run behavior.
8. Explore Iroh-based publishing/distribution of newly created C3 add-ons through UniStore-compatible integration (see [Iroh publishing](iroh-unistore-publishing.md)).
