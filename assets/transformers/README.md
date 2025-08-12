# Offline ORT WASM setup for transformers.js

This project is configured to run fully offline. To enable the CPU/WASM backend without using any CDN, place the ONNX Runtime Web (ORT) distribution files into this directory:

- Path: assets/transformers/dist/

index.html is already configured to point transformers.js to this local folder:
- See [JavaScript.assignment env.ORT.wasm.wasmPaths](index.html:37)
- See [JavaScript.assignment env.backends.onnx.wasm.wasmPaths](index.html:47)


Required files in assets/transformers/dist/
- ort-wasm-simd-threaded.jsep.mjs
- ort-wasm-simd.jsep.mjs
- ort-wasm.jsep.mjs
- ort-wasm-simd-threaded.wasm
- ort-wasm-simd.wasm
- ort-wasm.wasm

Notes:
- With proxy=false (current setting), worker files are not required and SharedArrayBuffer/COOP/COEP are not needed.
- Keep .mjs and .wasm filenames exactly as above; do not rename.


Recommended ORT versions
- Start with onnxruntime-web 1.19.2 or 1.20.x — both are generally compatible with recent transformers.js builds.
- If you encounter initialization errors, try switching between 1.19.2 and 1.20.0 distributions.

Where to obtain ORT files (prepare offline in advance)
- From the npm package onnxruntime-web:
  1) Download the package ahead of time on a machine with internet.
  2) Copy node_modules/onnxruntime-web/dist/* into assets/transformers/dist/ on this machine.
- Alternatively, use any internal artifact repository or offline mirror you maintain.


Server MIME types (must be correct)
Your server must serve the following MIME types to avoid the browser blocking ESM/wasm:

- .mjs -> text/javascript
- .js  -> text/javascript
- .wasm -> application/wasm
- .json, .map -> application/json

Examples:

IIS (web.config)
----------------
<configuration>
  <system.webServer>
    <staticContent>
      <mimeMap fileExtension=".mjs" mimeType="text/javascript" />
      <mimeMap fileExtension=".wasm" mimeType="application/wasm" />
      <mimeMap fileExtension=".map" mimeType="application/json" />
      <mimeMap fileExtension=".json" mimeType="application/json" />
    </staticContent>
  </system.webServer>
</configuration>

Apache (.htaccess)
------------------
AddType text/javascript .mjs
AddType application/wasm .wasm
AddType application/json .map .json

Node/Express
------------
app.use((req, res, next) => {
  if (req.path.endsWith('.mjs')) res.type('text/javascript');
  else if (req.path.endsWith('.wasm')) res.type('application/wasm');
  else if (req.path.endsWith('.map') || req.path.endsWith('.json')) res.type('application/json');
  next();
});


Verification checklist
1) Open these in the browser (adjust base path to your server):
   - .../assets/transformers/dist/ort-wasm-simd-threaded.jsep.mjs
     - Expect: HTTP 200, Content-Type: text/javascript, readable JS module
   - .../assets/transformers/dist/ort-wasm-simd-threaded.wasm
     - Expect: HTTP 200, Content-Type: application/wasm, non-zero size

2) Reload the app and watch the console:
   - If WebGPU is unsupported or fails, it should log:
     - [Embeddings] Trying CPU/WASM…
     - [Embeddings] Ready on WASM.

3) Models are loaded locally from:
   - [JavaScript.assignment env.localModelPath](index.html:31)
   - [JavaScript.assignment env.allowLocalModels](index.html:33)
   - [JavaScript.assignment env.allowRemoteModels](index.html:32)


Threads and performance (optional)
- Current config uses proxy=false and up to 2 threads to avoid COOP/COEP requirements.
- For higher performance and if you can configure COOP/COEP headers:
  - Set proxy=true and increase numThreads in index.html:
    - [JavaScript.assignment env.ORT.wasm.numThreads](index.html:43)
    - [JavaScript.assignment env.backends.onnx.wasm.numThreads](index.html:49)


Troubleshooting
- Disallowed MIME type for .mjs/.wasm:
  - Fix server MIME types as shown above.
- no available backend found:
  - Confirm the six ORT files exist in assets/transformers/dist and are served with correct MIME types.
- TypeErrors (e.g., t.getValue is not a function):
  - Try another onnxruntime-web version (1.19.2 vs 1.20.0) to match your bundled transformers.js.