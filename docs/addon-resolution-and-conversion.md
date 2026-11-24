# Construct 2 add-on discovery, UniStore matching, and conversion

## Goal

Automatically identify the add-ons used by a Construct 2 project and find the corresponding packages that can be converted to Construct 3. The initial conversion strategy uses UniStore as the add-on discovery and package source, and `c2-addon-assistant` as the add-on conversion workflow.

## 1. Scan the project, not just the editor's visible list

Inspect the project's serialized files and metadata to enumerate add-on types used by the project. Discovery should account for plugins, behaviors, effects, and dependencies where the project format exposes them.

Each discovered dependency should be represented in an inventory, with fields such as:

- Source add-on ID/type and name, when available.
- Category (plugin, behavior, effect, etc.).
- Files/objects or project locations that reference it.
- Whether the dependency appears built-in, third-party, or unknown.
- Discovery notes and parser confidence.

Avoid counting duplicate occurrences as multiple dependencies: keep the dependency entry unique while preserving its reference locations.

## 2. Find a C3 candidate in UniStore

For each C2 add-on in the inventory:

1. Search UniStore using the source add-on's name and ID (where applicable).
2. Collect available metadata for candidate packages, including listing name, add-on identifier, version, author, description, category, and package/download reference when provided.
3. Compare candidates. A matching name alone may be insufficient; identifiers, author, category, description, and version history can help disambiguate.
4. Mark each resolution as **exact**, **probable**, **ambiguous**, **not found**, or **needs manual review**.
5. Save the selected match and evidence to a manifest. Allow manual override and keep the decision visible.

A Construct 2 identifier and Construct 3 identifier may not be identical. The resolver should not assume cross-version ID equality unless confirmed by UniStore metadata or other evidence. Preserve both IDs and their relationship.

## 3. Download packages individually

Once a candidate is selected, acquire its package separately. The planned flow is intentionally one package at a time to keep each conversion job isolated and to make errors easier to diagnose.

The fetch stage should:

- Record package URL or UniStore reference, version, metadata, and download time.
- Validate downloaded data where checksums or signatures are available.
- Avoid executing package contents during download, parsing, or inspection.
- Apply configurable retry/backoff and clear timeouts.
- Respect UniStore rate limits, access requirements, terms, and package licensing.
- Store each package in a dedicated temporary/work directory with a stable identifier.
- Make the acquisition stage resumable and avoid fetching the same resolved version repeatedly unless requested.

## 4. Convert through c2-addon-assistant

The project will investigate and integrate the `c2-addon-assistant` tool and its underlying C2-to-C3 add-on conversion capability (including the referenced `C2C3AddonConverter` project) as an external conversion stage.

The converter adapter should provide one input package per job and collect:

- Input package identity and version.
- Conversion tool version and configuration.
- Output package/files and detected C3 identity.
- Exit status, diagnostics, and conversion warnings.
- Whether the result is ready for integration, needs manual work, or failed.

Conversion is not assumed to be perfect or universally supported. Unsupported constructs and failures must be surfaced, never hidden behind a success status.

## 5. Integrate the result into the converted project

Only successful, validated C3 add-on outputs should be staged for the converted project. Maintain a mapping such as:

`C2 source ID → UniStore candidate/package → converted C3 add-on ID → output location`

Project references should be rewritten only when a trustworthy mapping exists. If no package is found or conversion fails, preserve enough project information to make manual recovery possible and report the dependency as unresolved.

## 6. Suggested resolution manifest

The format is not yet fixed. A JSON manifest could represent each dependency with:

- `source`: original C2 ID/name/category.
- `candidate`: UniStore package ID/name/version and metadata.
- `match_status`: exact/probable/ambiguous/not-found/manual.
- `match_evidence`: which fields matched and why.
- `package`: download reference, checksum (if available), local cache state.
- `conversion`: tool/version, status, output, diagnostics.
- `integration`: C3 ID, output path, project reference updates.
- `manual_notes`: human review decisions and follow-up.

The manifest should be editable and serve as the checkpoint between automated discovery and conversion, especially for ambiguous matches.

## 7. Failure and safety behavior

- Missing or ambiguous UniStore matches must not trigger arbitrary downloads.
- Never execute untrusted packages to identify them.
- Add-on jobs should be isolated and have bounded resource use.
- A failure for one dependency should not erase results for other dependencies.
- Clearly distinguish conversion warnings from errors and from project-level blockers.
- Keep a source-project backup or write conversion output to a separate destination.
- Do not redistribute third-party add-ons or converted packages without permission and license review.
