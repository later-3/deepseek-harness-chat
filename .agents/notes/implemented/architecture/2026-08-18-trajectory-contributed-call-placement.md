# Agent Note: Placement and labels for contributed Trajectory calls

Status: implemented

English | [中文](2026-08-18-trajectory-contributed-call-placement.zh.md)

## Problem

Trajectory already accepts independently contributed Tool trees from registered Conversation Definitions. The contribution carries a Conversation Location, but the Trajectory snapshot previously retained Locations only for ordinary event Nodes. After an independent root Tool settled, layout therefore treated it as an unowned result and placed it in the Turn prologue. The resulting ledger could put a Step-owned call tree before the User and Context records that caused it.

The fixed Tool and Subtool labels also hid useful domain roles supplied by an external Definition. Callers could encode those roles into Tool names, but the tag still said `TOOL` or `SUBTOOL`, and changing the core kind would duplicate presentation and interaction behavior.

## Decision

The Trajectory Builder retains the contribution Location for every independently contributed root call. Layout consults that Location for both running and settled independent roots. A resolved Step Location places the complete call tree in that Step; absent or unresolved Locations preserve the existing fallback behavior. Assistant-owned Tool calls continue to follow their Assistant Step and do not need a separate Location.

A Tool contribution may provide an optional map from root or descendant call ID to a semantic display label. Layout copies the label onto the corresponding ledger record, and the table, inspector, accessible row name, tooltip, and search index use it when present. The record remains `tool` or `subtool`; kind still owns color, icon, folding, nesting, timing, selection, and inspection behavior.

A Tool contribution may also provide compact input and output previews per call ID. Layout uses only those values in the ledger row while preserving the original `argsRaw` and Tool Result content for the inspector. An omitted preview retains the native summary; an explicitly empty preview suppresses that side of the row without deleting its detail. The contribution therefore controls presentation density, not recorded data.

The extension belongs to the existing target-specific contribution contract described by the [Trajectory assembly decision](2026-08-11-trajectory-conversation-context-assembly.md). It does not introduce a second history source, a new event family, or knowledge of any external workflow product.

## Distribution and upstream sync

The Chat-maintained derivative lives in the private `later-3/deepseek-harness-chat` repository. Its `origin/main` is the published source of record for derivative source, the official `deepseek-ai/deepseek-harness` repository is the read-only `upstream`, and isolated work uses `codex/*` branches. Chat runtime consumers still pin the rc.6 npm distribution and apply a reviewable patch, so this repository owns development and upstream-merge history while the Chat repository owns the deployed version, patch hash, and runtime drift check.

An upstream release or relevant Trajectory change starts from `origin/main` in an isolated worktree. Maintainers first determine whether upstream provides equivalent public contribution fields; an equivalent contract retires the derivative instead of preserving it. Otherwise only Location retention, semantic display labels, and compact call previews are replayed, followed by the affected tests, repository typecheck, bundle, lint, and documentation checks. The Chat patch, lock hash, version evidence, and browser verification update from that validated commit before the private default branch advances.

## Alternatives considered

**Place every independent Tool tree after the last input.** Rejected: ordering without Conversation Location would be heuristic, would fail for multiple Steps, and would discard the ownership fact already supplied by the assembler.

**Require an Assistant Tool-call head for every external tree.** Rejected: fabricating an Assistant message changes the recorded execution and makes an external workflow look like a model-originated tool call.

**Add new workflow and agent record kinds.** Rejected: domain-specific kinds would multiply core presentation behavior and couple Trajectory to one integration. Semantic labels express the role without changing interaction semantics.

**Encode the role only in the Tool name.** Rejected: the event tag, accessible name, inspector, tooltip, and search vocabulary would still report only the generic kind.

**Put the complete contributed payload in each ledger row.** Rejected: repeated plans and execution results make the chronological ledger unreadable even though the inspector already owns full Input and Output detail.

## Verification

Builder tests retain root Locations, semantic labels, and compact previews. Layout tests pin running and settled Step placement, descendant ordering, label projection, compact row text, and unchanged inspector detail. Cell and table tests pin the visible tag and inspector while preserving the underlying `tool` or `subtool` kind. Existing tests cover contributions that omit the optional maps and therefore retain the native behavior.

## Consequences

External Definitions can project a truthful domain call tree at its real Step position without forging Conversation events or forking the complete Trajectory view. A label is deliberately presentation-only, so integrations cannot use it to change hierarchy or interaction rules.

The snapshot now carries three small maps proportional to independently contributed call trees. Call IDs within one snapshot share one namespace for Location, label, and preview metadata; Definition authors must use stable, collision-free IDs and treat missing metadata as the native fallback.
