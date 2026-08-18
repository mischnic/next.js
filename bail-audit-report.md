# Audit: end-user-reachable Rust bails

Revision: `6be687032b0d653dc20dc2784b221b59c1dc291f`

A tracked-source search found **370 textual `bail!` / `turbobail!` matches**:

| Classification | Count |
|---|---:|
| Conceivably end-user-reachable production bails | **211** |
| Production bails that are internal assertions/contracts only | 120 |
| Out of scope (tests, benchmarks, build/test helpers, standalone inspection tools) | 33 |
| Non-invocation matches (macro docs/implementation) | 6 |

“Reachable” is intentionally broad: direct malformed/unsupported project input, filesystem races or corruption, malformed dependencies/tool output, unsupported but public configuration/source constructs, and runtime/environment failures all qualify. A site is “internal” only when reaching it requires a typed pipeline, graph, generated-module, marker-filesystem, or method-call contract to have already been violated.

## Reachable findings

Locations sharing one trigger class are grouped. **High** confidence means the branch directly validates or reports user/environment data. **Medium** means the data is user/environment-derived but reaches the bail through generated assets or a framework pipeline.

### Next.js API and core

| Locations | Trigger and reachability evidence | Confidence |
|---|---|---|
| `crates/next-api/src/next_server_nft.rs:247` | Installed/project filesystem lacks the expected vendored NFT context layout. | Medium |
| `crates/next-api/src/nft.rs:125` | NFT output contains a file nested under a project/dependency symlink. | High |
| `crates/next-api/src/nft_json.rs:104` | Traced asset path lies outside supported NFT roots. | High |
| `crates/next-api/src/project.rs:1078` | User project/dist path escapes the project root. | High |
| `crates/next-api/src/sri_manifest.rs:21` | Configured SRI algorithm is unsupported. | High |
| `crates/next-api/src/versioned_content_map.rs:271` | A dev/HMR request asks for a path without a source map. | High |
| `crates/next-code-frame/src/frame.rs:206,209,213,216` | Compiler/plugin/source-map diagnostic supplies zero line or column values. | High |
| `crates/next-core/src/app_structure.rs:100,1380,1986` | User app filesystem has an un-stemmable metadata item, a parallel route without page/default, or a non-directory app root. | High |
| `crates/next-core/src/bootstrap.rs:38` | Project-derived bootstrap asset identity falls outside its base path. | Medium |
| `crates/next-core/src/next_app/metadata/route.rs:127` | Referenced metadata file is absent after route discovery. | High |
| `crates/next-core/src/next_app/mod.rs:54,58,169,178` | Filesystem-derived app route has empty/slashed segments, modifies a path after catch-all, or adds a segment after a terminal page. | High |
| `crates/next-core/src/next_client/runtime_entry.rs:52` | Configured/dependency runtime entry resolves to a non-evaluatable asset. | Medium |
| `crates/next-core/src/next_config.rs:1258,1264,2158,2534,2869,2990` | User config selects unsupported CSS chunking, loose ESM externals, inconsistent unused-import options, an invalid env severity, or unknown Lightning CSS feature. | High |
| `crates/next-core/src/next_font/google/mod.rs:627,631,642,646` | Generated font request query is malformed (not exactly one entry). | Medium |
| `crates/next-core/src/next_font/google/options.rs:79,107,117,128,155,167,179,183` | `next/font` call has unsupported argument count, weights, styles, display, or axes. | High |
| `crates/next-core/src/next_font/google/util.rs:65,76` | Selected font metadata has no definable axes or lacks a requested axis. | High |
| `crates/next-core/src/next_font/local/font_fallback.rs:271` | Local-font weight text is invalid. | High |
| `crates/next-core/src/next_font/local/mod.rs:52,56`; `crates/next-core/src/next_font/local/options.rs:58,62` | Generated local-font request query is malformed. | Medium |
| `crates/next-core/src/next_font/local/stylesheet.rs:121` | User local font file has an unsupported extension. | High |
| `crates/next-core/src/next_image/source_asset.rs:68` | User image is missing or resolves to non-file content. | High |
| `crates/next-core/src/next_server/context.rs:176` | A package appears in conflicting user package-list configuration. | High |
| `crates/next-core/src/next_shared/transforms/swc_ecma_transform_plugins.rs:121,129` | Configured SWC plugin is missing or not file-backed. | High |
| `crates/next-core/src/next_shared/webpack_rules/sass.rs:75` | `sassOptions` is not an object. | High |
| `crates/next-core/src/page_loader.rs:41` | Framework installation is incomplete/corrupt and lacks `entry/page-loader.ts`. | High |
| `crates/next-core/src/raw_ecmascript_module.rs:150` | File-backed raw module disappears or is missing. | High |
| `crates/next-core/src/segment_config.rs:169` | User route segments provide conflicting sibling config values. | High |
| `crates/next-core/src/util.rs:225,461,488,489,505,506,580` | User/environment paths escape roots, expected files disappear, JSON/JSONC is malformed, or glob configuration traverses above root. | High |
| `crates/next-custom-transforms/src/transforms/next_ssg.rs:74,89` | User page exports both SSG and SSR functions. | High |
| `crates/next-napi-bindings/src/next_api/project.rs:2388,2399,2485,2597,2672` | Request/API URL is invalid, sourcemap/source is missing, or original source lies outside the project. | High |
| `crates/next-napi-bindings/src/transform.rs:158` | Transform API input requires but omits a filename. | High |
| `turbopack/crates/turbo-esregex/src/lib.rs:88` | Source/package regex uses an unsupported flag. | High |

### Persistence, task runtime, and filesystem

| Locations | Trigger and reachability evidence | Confidence |
|---|---|---|
| `turbopack/crates/turbo-persistence/src/db.rs:510,541,607,625,680,723,2219` | Cache directory is missing/malformed, contains unexpected entries, has checksum corruption/tombstone mismatch, or a prior I/O write failure left the DB failed. | High |
| `turbopack/crates/turbo-persistence/src/meta_file.rs:287` | Persisted cache meta file has invalid magic. | High |
| `turbopack/crates/turbo-persistence/src/static_sorted_file.rs:316,334,560,571,589,620,827,911,1052` | Persisted cache has corrupt block types, offsets, checksums, or entries. | High |
| `turbopack/crates/turbo-tasks-backend/src/backend/mod.rs:921` | A task is canceled by a user-driven invalidation/shutdown while awaited. | Medium |
| `turbopack/crates/turbo-tasks-backend/src/backend/operation/mod.rs:428,450` | Persistence restoration fails after filesystem/cache failure. | Medium |
| `turbopack/crates/turbo-tasks-fs/src/content.rs:587,588,599,600` | Project JSON is malformed or missing. | High |
| `turbopack/crates/turbo-tasks-fs/src/disk.rs:979,1197,1439,1474` | Project/plugin operation targets a denied path or filesystem contains an invalid symlink. | High |
| `turbopack/crates/turbo-tasks-fs/src/embed/fs.rs:87` | Requested embedded framework path is missing. | Medium |
| `turbopack/crates/turbo-tasks-fs/src/lib.rs:139` | Filesystem race leaves a symlink where resolution expected otherwise. | High |
| `turbopack/crates/turbo-tasks-fs/src/path.rs:213,224,240,367,444` | User-derived path traverses above root, contains forbidden slash suffixes, fails rebase, or has a realpath error. | High |
| `turbopack/crates/turbo-tasks-fs/src/read_glob.rs:125,197` | Project symlinks form a loop or become unresolved. | High |
| `turbopack/crates/turbo-tasks-fs/src/watcher/mod.rs:433` | Watch root is deleted/replaced or watcher state races with filesystem mutation. | High |
| `turbopack/crates/turbo-tasks/src/effect.rs:635` | Repeated user-driven invalidations cause effect reconciliation to exhaust retries. | Medium |
| `turbopack/crates/turbo-tasks/src/output.rs:17` | A task error produced by project/environment processing is read as output; this wrapper propagates the original reachable failure. | Medium |

### Chunking, resolution, and dev server

| Locations | Trigger and reachability evidence | Confidence |
|---|---|---|
| `turbopack/crates/turbopack-browser/src/chunking_context.rs:539` | User/dependency module graph contains a chunk that cannot produce an output asset. | Medium |
| `turbopack/crates/turbopack-browser/src/ecmascript/content.rs:75`; `turbopack/crates/turbopack-browser/src/ecmascript/evaluate/chunk.rs:155` | Generated chunk identity derived from project assets falls outside output root. | Medium |
| `turbopack/crates/turbopack-cli-utils/src/runtime_entry.rs:52` | Configured runtime entry resolves to non-evaluatable asset. | High |
| `turbopack/crates/turbopack-cli/src/build/mod.rs:504`; `turbopack/crates/turbopack-cli/src/dev/web_entry_source.rs:210` | User CLI entry module is not chunkable. | High |
| `turbopack/crates/turbopack-core/src/chunk/chunking_context.rs:548,776` | User worker is used in an unsupported context, or generated chunk root escapes output root. | Medium |
| `turbopack/crates/turbopack-core/src/chunk/evaluate.rs:49` | User/configured evaluated entry is not evaluatable. | High |
| `turbopack/crates/turbopack-core/src/chunk/mod.rs:93` | User config/source supplies unsupported `crossOrigin`. | High |
| `turbopack/crates/turbopack-core/src/context.rs:36,39` | A user-selected source/rule is ignored or cannot be processed where a module is required. | Medium |
| `turbopack/crates/turbopack-core/src/data_uri_source.rs:89` | Source import contains unsupported data-URL encoding. | High |
| `turbopack/crates/turbopack-core/src/environment.rs:455,473` | Installed `node` fails or prints an unexpected version format. | High |
| `turbopack/crates/turbopack-core/src/file_source.rs:74,82` | Project filesystem reports invalid symlink or file type. | High |
| `turbopack/crates/turbopack-core/src/issue/mod.rs:1194` | Fatal issue from project processing is promoted to an error; wrapper propagates the user-facing diagnostic. | High |
| `turbopack/crates/turbopack-core/src/node_addon_module.rs:138` | Native addon realpath fails due to project/dependency symlink state. | High |
| `turbopack/crates/turbopack-core/src/resolve/mod.rs:755,1152,1358,1514,3122,3306` | Source/config produces unsupported traced external or non-constant package-internal request; package/symlink paths fail realpath or containment. | High |
| `turbopack/crates/turbopack-core/src/resolve/options.rs:448,468` | Dynamic external/alias request lacks an explicit name. | High |
| `turbopack/crates/turbopack-core/src/resolve/remap.rs:208,209,215,447,477,485,522,530` | User/dependency `exports` or `imports` metadata has invalid values, keys, shape, or folder targets. | High |
| `turbopack/crates/turbopack-core/src/source_map/structured.rs:327` | User/dependency/tool source map has an unknown field. | High |
| `turbopack/crates/turbopack-core/src/version.rs:240` | File hashing encounters redirect content from an asset/symlink. | Medium |
| `turbopack/crates/turbopack-dev-server/src/html.rs:231` | Emitted chunk has an asset extension the HTML renderer does not support. | Medium |

### JavaScript, CSS, assets, loaders, and workers

| Locations | Trigger and reachability evidence | Confidence |
|---|---|---|
| `turbopack/crates/turbopack-ecmascript/src/analyzer/well_known/require_context.rs:29,33,40,53` | User `require.context` has invalid arity or non-constant/non-boolean/non-regex arguments. | High |
| `turbopack/crates/turbopack-ecmascript/src/bytes_source_transform.rs:46`; `turbopack/crates/turbopack-ecmascript/src/json_source_transform.rs:129`; `turbopack/crates/turbopack-ecmascript/src/text_source_transform.rs:43` | Project file disappears after resolution. | High |
| `turbopack/crates/turbopack-ecmascript/src/chunk/item.rs:108` | User/dependency creates an async CommonJS module, which is unsupported. | High |
| `turbopack/crates/turbopack-ecmascript/src/hmr/version.rs:37` | Project-derived HMR chunk path falls outside output root. | Medium |
| `turbopack/crates/turbopack-ecmascript/src/lib.rs:2892,2909` | Very large user/dependency source exceeds byte-position table capacity. | High |
| `turbopack/crates/turbopack-ecmascript/src/minify.rs:67` | User/dependency source cannot be parsed during minification. | High |
| `turbopack/crates/turbopack-ecmascript/src/module_fragments/mod.rs:403,423` | Splitting/parsing a user module fails before a requested fragment can be obtained. | Medium |
| `turbopack/crates/turbopack-ecmascript/src/references/esm/base.rs:639,681,878,908` | Loader cannot resolve, part import lacks export name, or external module type is unsupported by target environment. | High |
| `turbopack/crates/turbopack-ecmascript/src/references/esm/url.rs:213,332` | User URL import resolves to unsupported external asset type. | High |
| `turbopack/crates/turbopack-ecmascript/src/references/import_meta_glob.rs:471` | Matched project file cannot be made relative to the glob root. | High |
| `turbopack/crates/turbopack-ecmascript/src/references/raw.rs:189` | Raw import realpath fails because of project/dependency symlink state. | High |
| `turbopack/crates/turbopack-ecmascript/src/static_code.rs:41` | Requested static-code asset is not ECMAScript. | High |
| `turbopack/crates/turbopack-ecmascript/src/transform/mod.rs:228` | User React transform runtime is unsupported. | High |
| `turbopack/crates/turbopack-ecmascript/src/worker_chunk/module.rs:89,330` | User worker is not evaluatable or its entry asset cannot be found. | High |
| `turbopack/crates/turbopack-image/src/process/mod.rs:288,346` | Requested image output format is unavailable or source image is missing. | High |
| `turbopack/crates/turbopack-image/src/process/svg.rs:83,93,98` | User SVG lacks root/dimensions or has width without height. | High |
| `turbopack/crates/turbopack-mdx/src/lib.rs:135,139` | User MDX is missing/unreadable or resolves to unexpected content. | High |
| `turbopack/crates/turbopack-node/src/evaluate.rs:704,713` | Evaluated user configuration/plugin sends protocol messages unsupported by the basic evaluation context. | Medium |
| `turbopack/crates/turbopack-node/src/process_pool/mod.rs:145,203,383,388,395,444,951` | Node subprocess exits, times out, closes streams, sends invalid ready data, or becomes unusable. | High |
| `turbopack/crates/turbopack-node/src/transforms/postcss.rs:323,336,370,505` | PostCSS config/source is wrong file type, on another filesystem, or non-file-backed. | High |
| `turbopack/crates/turbopack-node/src/transforms/webpack.rs:246,298,704,711,742` | Loader input is non-file, crosses filesystem roots, or loader resolution/`importModule` fails. | High |
| `turbopack/crates/turbopack-nodejs/src/chunking_context.rs:386,646,691` | Project-derived Node chunk cannot produce output/be placed, or requests evaluated groups unsupported by that context. | Medium |
| `turbopack/crates/turbopack-nodejs/src/ecmascript/node/entry/chunk.rs:82,90`; `turbopack/crates/turbopack-nodejs/src/ecmascript/node/entry/chunk_list_content.rs:29`; `turbopack/crates/turbopack-nodejs/src/ecmascript/node/entry/runtime.rs:59` | Generated chunk/runtime identity derived from project assets falls outside output root. | Medium |
| `turbopack/crates/turbopack-resolve/src/node_native_binding.rs:195` | Native binding realpath fails due to dependency symlink/filesystem state. | High |
| `turbopack/crates/turbopack-wasm/src/module_asset.rs:235` | User WASM/service-worker use requests an unsupported single-chunk context. | Medium |
| `turbopack/crates/turbopack-wasm/src/raw.rs:109` | Project-derived WASM asset identity falls outside output root. | Medium |
| `turbopack/crates/turbopack/src/module_options/mod.rs:123`; `turbopack/crates/turbopack/src/module_options/module_rule.rs:224` | Module-rule configuration names an invalid built-in condition or unknown module type. | High |

## Borderline, internal, and excluded calls

The **Medium** rows are included because each can plausibly be reached while processing a real project even though the immediate mismatch is in generated/framework state. Most review-worthy: generated paths outside output roots, framework-generated font queries, unsupported evaluated-worker/runtime entries, task cancellation/restoration, effect retry exhaustion, and protocol messages emitted by evaluated user configuration.

The 120 internal sites are recurring contracts: methods documented/typed as unreachable; exact-type chunk/module downcasts; module-graph identity/index consistency; generated module-part and `next-taskless` template bookkeeping; internal NAPI mode/target preconditions; task-cell/`RawVc` and persistence write-batch invariants; post-finalization bounds; and initialization contracts.

The 33 out-of-scope invocations are in tests, benchmark/test-app utilities, `next-build-test`, `xtask`, persistence/SST inspection binaries, and the standalone trace server. Six textual matches in `turbo-tasks-macros` are documentation or the implementation that emits `anyhow::bail!`, not independent call sites.

## Reproduction and verification

- Inventory: `git grep -n -E '\b(bail|turbobail)!\s*\(' -- '*.rs'`.
- Every match was assigned exactly one classification (`reachable`, `internal`, `out of scope`, or `non-invocation`).
- A verifier re-ran the inventory, checked totals, proved that all 211 reachable identities appear above, and confirmed each still contains the macro at that source line.
- This was a read-only static audit; the dev server, build, and test suites were intentionally not run.
