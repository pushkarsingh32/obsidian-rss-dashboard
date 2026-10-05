# Saved-Article Content Source Follows the Full-Content Setting

## Status

proposed — pending the agreed #247 refactor work landing. This proposal is not current project policy; revisit it for acceptance after that work lands.

## Date

2026-10-05

## Context and problem

The **Save full content** setting is meant to choose whether a saved article uses the fetched webpage body or the feed's own article content. The `{{content}}` template placeholder chooses where the selected body appears; it does not choose its source. Existing save paths did not apply the setting consistently, and the full-content saver could prefer feed content even after a successful fetch when that feed content was longer or came from a preferred publisher.

This choice changes text written into users' vault notes, so a fallback must remain identifiable after the transient save notification is gone. The separate `{{summary}}` template variable already has a frozen compatibility contract in [ADR 0007](0007-article-metadata-pipeline-seam.md) and [#271](https://github.com/amatya-aditya/obsidian-rss-dashboard/issues/271).

## User stories

1. As a user who enables full-content saving, I want the fetched article body in `{{content}}` whenever it is available, so my saved note reflects the webpage rather than the feed's shorter or longer copy.
2. As a user whose full-content fetch fails, I want the saved RSS fallback clearly identified, so I can distinguish it from fetched article text.

For the complete acceptance criteria and accessibility checks, see [GitHub Issue #759](https://github.com/amatya-aditya/obsidian-rss-dashboard/issues/759).

## Proposed decision

The setting selects the source for `{{content}}`, while the placeholder continues to control placement:

- When enabled, use a nonempty fetched article body regardless of the feed body's length or publisher-specific preference heuristics.
- When disabled, use the RSS item's content, preserving the legacy `{{summary}}` value and behavior unchanged.
- When an enabled fetch cannot provide a body (including network, restricted, empty-extraction, or escaped-error cases), use available RSS item content and place `> RSS feed content shown because the full article could not be fetched.` immediately before it at each `{{content}}` occurrence in the note body. Do not show that marker when no RSS fallback text is present.
- Apply the setting to Dashboard saves and all standalone Reader saves: the default-save toolbar action, custom-folder action, and `s` shortcut. Web Viewer saving remains separate because it saves the webpage content directly.

The setting description will state the enabled and disabled sources, the marked fallback, and that `{{content}}` controls placement rather than source. The setting remains keyboard-operable, exposes its current state to assistive technology, and has a clear description; verify keyboard operation and visible focus, with targeted screen-reader checks when name, state, focus, or announcements change.

## Consequences

### Potential benefits

- The fetched body is selected consistently when the setting is on, including for syndicated feeds whose feed copy is longer or has a publisher-specific preference.
- Dashboard and standalone Reader save actions follow one source-selection rule.
- A saved fallback remains distinguishable from a fetched body after the save notification disappears.
- `{{content}}` placement and the frozen `{{summary}}` contract remain independent.

### Potential trade-offs

- Some preferred-host or longer-feed cases may produce a shorter saved body than the feed's copy, even though a fetch succeeded; preserving the explicit setting contract takes precedence over the former heuristic.
- The fallback marker is body text and therefore follows every `{{content}}` occurrence in a note body. A multiline `{{content}}` value in custom YAML can already invalidate that YAML; frontmatter placement is not the supported location for this body marker.
- Web Viewer continues to use its own save behavior because it does not save an RSS `FeedItem` through the full-content setting.

### Existing users and data if accepted

- Existing templates and stored feed data need no migration. The new marker appears only when full-content saving is enabled and fetched content is unavailable while RSS fallback text exists.
- Existing `{{summary}}` templates continue to produce the same byte-identical legacy excerpt.

## Considered options

### Keep the longer/richer-feed heuristic after a successful fetch

This can retain a longer or publisher-specific feed body, but it conflicts with the setting's stated purpose and makes a successful fetch fail to determine `{{content}}`. Not selected in this proposal.

### Leave the fallback unmarked

This avoids changing the note body, but loses provenance once the transient notice is dismissed and can make feed content appear to be fetched content. Not selected in this proposal.

### Leave `{{content}}` empty on fetch failure

This avoids source ambiguity, but discards usable RSS content. Not selected in this proposal; the proposed approach retains available content with a clear marker.

## Related

- [ADR index](README.md)
- [GitHub Issue #759](https://github.com/amatya-aditya/obsidian-rss-dashboard/issues/759)
- [GitHub Issue #247](https://github.com/amatya-aditya/obsidian-rss-dashboard/issues/247) and [#271](https://github.com/amatya-aditya/obsidian-rss-dashboard/issues/271) — article-metadata terminology and frozen `{{summary}}` contract.
