# Publishing Construct 3 add-ons through UniStore with Iroh

## Idea

Explore using Iroh for peer-to-peer distribution and publication of new Construct 3 add-ons through UniStore. The goal is to support a publication path for add-ons created for Construct 3, complementing the planned dependency acquisition workflow used during C2-to-C3 project conversion.

This is a future research and design item, not a currently implemented feature or a confirmed UniStore integration.

## Potential workflow

1. **Prepare the add-on:** validate its C3 structure and metadata; identify its name, ID, version, author, description, category, and required files.
2. **Build a publication artifact:** create a deterministic archive or package representation, with a manifest describing the content.
3. **Integrity metadata:** calculate a content hash and, where practical, support author signatures or other authenticity metadata. Hashes prove content identity, not author identity.
4. **Publish/announce:** investigate how UniStore accepts or indexes new C3 add-ons. If a supported publication or submission interface exists, integrate with it. Iroh may provide transport/content-addressed distribution, but it must not be assumed to replace UniStore's listing, moderation, or metadata requirements.
5. **Distribute via Iroh:** explore sharing the package using Iroh's content-addressed and peer-to-peer capabilities, including how a UniStore entry would refer to or retrieve that content.
6. **Verify and retrieve:** clients should validate the package hash and any available author-signature information before installation.
7. **Update and revoke:** design versioned publication, withdrawal/deprecation, and handling of compromised or superseded artifacts.

## Research questions

- Does UniStore expose an official API or supported workflow for add-on submissions, metadata updates, and package downloads?
- Which package formats, metadata fields, and validation rules are required by Construct 3?
- Can UniStore listing records refer to external or Iroh-distributed package content, or would an integration require a separate bridge/service?
- What are the trust, moderation, versioning, availability, and abuse-prevention requirements?
- How should users discover content when peers are offline, and is there a fallback transport?
- How should authors prove control of an add-on ID and publish updates securely?

## Design principles

- Treat Iroh as a potential distribution layer, not as an assumed UniStore backend.
- Preserve a stable add-on identity and versioned metadata.
- Verify content integrity on download and before installation.
- Provide conventional fallback delivery if peer availability is insufficient.
- Respect UniStore rules, Construct 3 package requirements, and author licensing.
- Make the publication status, provenance, and verification results visible to users.

## Status

**Research/planning only.** No Iroh integration, UniStore publication API, package bridge, or automated publishing functionality is implemented by this documentation.
