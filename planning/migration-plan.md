# kitMSDF -> libMSDF Migration Plan
Date: 2026-02-07

## Goal
Rename and simplify: remove workers, clean API, move submodule, rename everything.

## Output (dist/)
- libMSDF.js (esbuild bundle: Emscripten glue + TS wrapper)
- libMSDF.wasm
- libMSDF.d.ts
- shader.js
- api.md

## Execution Order

### Phase 1: Submodule Move
1. `git submodule deinit msdf-atlas-gen`
2. `git rm msdf-atlas-gen`
3. `rm -rf .git/modules/msdf-atlas-gen`
4. `mkdir submodule`
5. `git submodule add https://github.com/Chlumsky/msdf-atlas-gen.git submodule/msdf-atlas-gen`
6. Create `submodule/PROVENANCE.md`

### Phase 2: Delete Workers
7. Delete `src/worker.ts`
8. Delete `src/worker-browser.ts`
9. Delete `src/worker-deno.ts`

### Phase 3: Rename shader
10. `git mv src/shaders.js src/shader.js`

### Phase 4: Source Changes
11. Modify `src/msdf-generator.ts` (~10 line change to init)
12. Modify `src/index.ts` (exports)

### Phase 5: Build Config
13. Modify `Makefile.wasm` (paths: submodule/msdf-atlas-gen/msdfgen, output name)
14. Create `tsconfig.json`
15. Modify `lulu.yaml` (name, deps, build sections, output filenames)
16. Modify `package.json` (name)
17. Modify `.gitignore` (shader.js reference)

### Phase 6: Docs & Tests
18. Rewrite `tests/test-msdf.ts` (new init API)
19. Modify `example/glyph/index.html` (import path, add critical note)
20. Rewrite `docs/api.md`
21. Rewrite `README.md`
22. Clear `go.sh`

### Phase 7: Repo Rename (manual/last)
23. `gh repo rename libMSDF`
24. Rename local directory kitMSDF -> libMSDF
25. Update git remote URL

## Key Risks
- esbuild bundling Emscripten JS: if it breaks, fallback is 2 JS files in dist
- Submodule move: git submodule operations can be finicky
- locateFile override: standard Emscripten API, well-documented

## API Change Summary
OLD: `import Factory from './msdf-core.js'; const msdf = await MSDFGenerator.init(Factory);`
NEW: `import { initMSDF } from './libMSDF.js'; const msdf = await initMSDF('./libMSDF.wasm');`
All methods after init unchanged.
