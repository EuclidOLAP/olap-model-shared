<!-- Copilot instructions tailored for the olap-model-shared repo -->
# Copilot / AI contributor instructions

Purpose: short, actionable guidance so an AI coding agent can be productive immediately.

- Big picture:
  - This repository is a small TypeScript library that exports OLAP-related utilities and interfaces from `src/index.ts` and is packaged as `@euclidolap/olap-model`.
  - Build output goes to `dist/` (compiled CommonJS + declaration files). `package.json` uses `main: dist/index.js` and `types: dist/index.d.ts`.

- Key files to inspect when making changes:
  - `src/index.ts` — single entrypoint; contains exported classes, interfaces and utility functions (e.g. `OlapModelUtil.bytesToUint64`, `OlapEntityTypeChecker`).
  - `package.json` — scripts: `build` (runs `tsc`) and `prepublishOnly` (runs the build before publish). The package is published as `@euclidolap/olap-model`.
  - `tsconfig.json` — `outDir: ./dist`, `rootDir: ./src`, `declaration: true`, `module: commonjs`, `target: ES2020`, `strict: true`.

- Build / publish / debugging workflows:
  - Local build: `npm run build` (invokes `tsc`). Use this to regenerate `dist/` after edits.
  - Verify types are emitted: after build, expect `dist/index.js` and `dist/index.d.ts`.
  - Publish flow: `npm publish` (the `prepublishOnly` script runs `npm run build` automatically). Agents should not skip `npm run build` before publishing.
  - There are no test scripts or CI configs in the repo — don't assume tests exist.

- Project-specific conventions and patterns (discovered from `src/index.ts`):
  - GID interpretation: `OlapEntityTypeChecker.check(olap_obj_gid)` uses numeric thresholds:
    - >= 700000000000001 => CalculatedMetric (enum value 7)
    - >= 300000000000001 => Member (enum value 3)
    - Otherwise throws an error. Keep these thresholds intact when modifying type logic.
  - Byte handling utilities:
    - `OlapModelUtil.bytesToUint64(bytes: Uint8Array): string` expects exactly 8 bytes and returns a base-10 string representation of the unsigned 64-bit value.
    - `gidFullPath_uint8Arr_into_uint64Arr` splits a `Uint8Array` into 8-byte chunks, reverses each chunk, and converts to numbers via `bytesToUint64` (note: this code converts the BigInt string to `Number` — potential precision loss for very large IDs; any change that affects numeric ranges should consider using `bigint` consistently).
  - SQL DDL excerpts are embedded as comments in `src/index.ts` and are used as living documentation for interfaces like `User`, `Cube`, `Member`, and `UserOlapModelAccess`.

- Coding agent rules for edits
  - Keep the library API stable: update `src/index.ts` exports carefully. If you change exported types or runtime behavior, update `package.json` version accordingly.
  - Always run `npm run build` locally after code edits to regenerate `dist/` and verify TypeScript errors are resolved.
  - Do not modify `package.json.files` unless you intentionally change the published artifacts (currently `files: ["dist"]`).
  - Avoid adding runtime dependencies without justification — this library currently has only `typescript` as a devDependency.

- Examples to reference when making changes
  - Modify `OlapEntityTypeChecker` only after confirming existing numeric thresholds are still correct; tests or callers elsewhere may rely on them.
  - If changing serialization of `fullPath` / `Uint8Array` handling, update both `OlapModelUtil.bytesToUint64` and `gidFullPath_uint8Arr_into_uint64Arr` in tandem to prevent mismatch.

- What we cannot discover here (ask the human if needed):
  - Consumer code expectations (other repos may rely on numeric `gid` ranges or string vs bigint behavior).
  - Publishing credentials and npm registry settings — ask before publishing.

If anything above is unclear or you want the instructions expanded with a checklist for common PR types (fix, feature, breaking change), tell me which PR types to document and I'll extend this file.

-- End
