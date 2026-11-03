# Generated component workflow

## Decide the operation

| Intent | Read first | Allowed write path |
| --- | --- | --- |
| Find or reuse | capabilities, then list; metadata detail only if needed | none |
| Create | capabilities, then list to avoid duplicates | create preview → review → finalize |
| Update | capabilities, then source detail for the current version | baseline preview → review diff → finalize |
| Archive | metadata detail and latest `updatedAt` | archive only after explicit user request |
| Supply a template component | capabilities, components, templates | finalize component, then generate template using exact IDs |

Before any component source is written, read `generated-component-runtime.md` and call `get_generated_component_capabilities`. Prefer existing components when their behavior and visual role fit; list responses are summaries and never contain bundles. Request `get_generated_component(detailLevel="source")` only to update a component or when metadata cannot establish reuse.

## Create or update

For a new component, send the complete minimal source, strict Props Schema, representative Props, and all referenced preview variables to `create_generated_component_preview`.

For an update, first read current source detail and pass all three immutable baseline fields: `componentId`, current `baseVersionId`, and current `expectedSourceHash`. Modify only what the user asked for. A same-file helper is allowed when necessary; extra features, exports, application entry points, iframe code, and repository files are not.

On `COMPONENT_PREVIEW_BASE_CONFLICT`, read the current source detail again, compare the new baseline, and regenerate against it. Never overwrite a newer version or reuse the stale hash.

## Compose personal components

Read `list_generated_components` and, when needed, `get_generated_component` to understand each component's description, Props Schema and required article variables. Reference an existing version with `dependencies: [{alias, componentId, versionId}]` and `import Alias from '@growthtrace/components/Alias'`. Preserve current dependency locks during an update unless changing them is part of the request. Both source and selected versions participate in approval identity; a version upgrade requires a fresh preview. The limit is 10 direct dependencies, 20 transitive components and 4 levels, with no cycles. Archived children revoke subsequent parent preview/script reads.

## Validate and review

Compilation repair and visual candidates are separate loops:

1. For a compile/input diagnostic, change the source or payload using its structured fields and submit a new preview. These repairs do not consume a visual candidate round.
2. Do not repeat an identical diagnostic with unchanged effective input. Re-read capabilities; if unsupported, offer a supported alternative.
3. After compilation succeeds, inspect the returned `previewUrl` only if the host has a browser. Check available desktop and mobile viewports, overflow, readability, interactions, and runtime.
4. If the host has no browser, do not use Shell, download a bundle, or claim visual validation. Give the user `reviewUrl` and say compilation passed but agent visual inspection was not run.
5. If only one viewport is available, state exactly which viewport was checked.
6. Visual adjustment is limited to three candidate rounds. After three, show the current result and remaining differences and ask whether to continue or accept it.

`agentRuntimeStatus` is evidence from the token preview only. `ownerRuntimeStatus` is evidence from the authenticated review page and is required for approval/finalization. Never infer one from the other. Give the user `reviewUrl`; only the user can approve or reject. Poll `get_generated_component_preview` without refreshing the token. Call `refresh_generated_component_preview_url` only when the URL is lost or expired; it invalidates the old URL and must not be called while an existing preview page is reporting status.

After `decision=APPROVED` and `ownerRuntimeStatus=READY`, call `finalize_generated_component_preview` with a new idempotency key. On `REJECTED`, use the feedback for a new candidate. On `PENDING`, wait. Final output separately states compilation status, agent inspection coverage, owner review status, and saved component/version IDs.

For article display composition, finalize each missing component first, then follow the [article display skill](../../growthtrace-article-display/SKILL.md) and refresh `get_article_display_context`. Write MDX using the returned exact aliases and `componentId + versionId` bindings, and submit it through `save_article_display_content`. The default is a candidate for review; only explicit authorization for the article and current task permits `apply_article_display_content`, followed by the applied preview. Display application authorization never replaces component approval. Component finalization itself does not save or apply an article display draft.

## Fill missing information before asking

1. Missing site facts: use the relevant MCP list/search/get tool.
2. Missing runtime facts: call capabilities; do not treat existing source as the capability list.
3. Compile uncertainty: use diagnostics and repair autonomously.
4. Safe defaults: empty optional description/Props/variables, responsive basic layout, and no automatic publication. Never invent non-optional domain data.
5. User-only decisions: OAuth/scope, approval, missing brand assets, core interaction meaning, public visibility, and accepting a material fallback must be decided by the user immediately.
6. Unsupported platform capability: state the verified limitation and supported alternatives. Ask only when choosing an alternative materially changes the result.

Only after these paths are exhausted should lack of information be reported as a blocker.
