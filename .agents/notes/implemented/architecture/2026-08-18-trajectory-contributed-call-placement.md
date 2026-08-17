# Agent Note: Placement and labels for contributed Trajectory calls

Status: implemented

English | [中文](2026-08-18-trajectory-contributed-call-placement.zh.md)

## Problem

Trajectory already accepts independently contributed Tool trees from registered Conversation Definitions. The contribution carries a Conversation Location, but the Trajectory snapshot previously retained Locations only for ordinary event Nodes. After an independent root Tool settled, layout therefore treated it as an unowned result and placed it in the Turn prologue. The resulting ledger could put a Step-owned call tree before the User and Context records that caused it.

The fixed Tool and Subtool labels also hid useful domain roles supplied by an external Definition. Callers could encode those roles into Tool names, but the tag still said `TOOL` or `SUBTOOL`, and changing the core kind would duplicate presentation and interaction behavior.

## Decision

The Trajectory Builder retains the contribution Location for every independently contributed root call. Layout consults that Location for both running and settled independent roots. A resolved Step Location places the complete call tree in that Step; absent or unresolved Locations preserve the existing fallback behavior. Assistant-owned Tool calls continue to follow their Assistant Step and do not need a separate Location.

A Tool contribution may provide an optional map from root or descendant call ID to a semantic display label. Layout copies the label onto the corresponding ledger record, and the table, inspector, accessible row name, tooltip, and search index use it when present. The record remains `tool` or `subtool`; kind still owns color, icon, folding, nesting, timing, selection, and inspection behavior.

The extension belongs to the existing target-specific contribution contract described by the [Trajectory assembly decision](2026-08-11-trajectory-conversation-context-assembly.md). It does not introduce a second history source, a new event family, or knowledge of any external workflow product.

## Alternatives considered

**Place every independent Tool tree after the last input.** Rejected: ordering without Conversation Location would be heuristic, would fail for multiple Steps, and would discard the ownership fact already supplied by the assembler.

**Require an Assistant Tool-call head for every external tree.** Rejected: fabricating an Assistant message changes the recorded execution and makes an external workflow look like a model-originated tool call.

**Add new workflow and agent record kinds.** Rejected: domain-specific kinds would multiply core presentation behavior and couple Trajectory to one integration. Semantic labels express the role without changing interaction semantics.

**Encode the role only in the Tool name.** Rejected: the event tag, accessible name, inspector, tooltip, and search vocabulary would still report only the generic kind.

## Verification

Builder tests retain root Locations and semantic labels. Layout tests pin running and settled Step placement, descendant ordering, and label projection. Cell and table tests pin the visible tag and inspector while preserving the underlying `tool` or `subtool` kind. Existing tests cover contributions that omit both optional inputs and therefore retain the prior behavior.

## Consequences

External Definitions can project a truthful domain call tree at its real Step position without forging Conversation events or forking the complete Trajectory view. A label is deliberately presentation-only, so integrations cannot use it to change hierarchy or interaction rules.

The snapshot now carries two small maps proportional to independently contributed call trees. Call IDs within one snapshot share a label namespace; Definition authors must use stable, collision-free IDs and treat a missing Location or label as the generic fallback.
