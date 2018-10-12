# Aioli

Aioli is a framework for building fast genomics web tools by compiling existing C/C++ command-line tools into [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly) and running them in background threads in the browser using [WebWorkers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API).

For an example of what can be built with Aioli, see [fastq.bio](https://github.com/robertaboukhalil/fastq.bio), which was built by compiling [seqtk](https://github.com/lh3/seqtk) to WebAssembly, and yielded significant speedups. fastq.bio uses Aioli to manage the WebAssembly calls and communications between the main thread and the WebWorker.

## Getting Started

### What is WebAssembly?
[WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly) is a very fast, low-level, compiled binary instruction format that runs in all major browsers at near native speeds.

### What is a WebWorker?
[WebWorkers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) allow you to run JavaScript in the browser in a background thread, which keeps the browser responsive.

### Compiling into WebAssembly
1. Download the [Emscriptem SDK](https://kripken.github.io/emscripten-site/docs/getting_started/downloads.html)
2. Follow the [Emscriptem tutorial](https://kripken.github.io/emscripten-site/docs/getting_started/Tutorial.html) for details on how to compile C/C++ files into `.js` and `.wasm`.
3. Run `./generate_template.sh app.js app.wasm` to generate a sample HTML/JS template.

## Structure
### aioli.js
Main Aioli class

```javascript
// Example from fastq.bio:
// Create Aioli object and import the compiled .js file
this.aioli = new Aioli({
    imports: [ "seqtk.js" ]
});

// Initialize Aioli. This will launch the WebWorker and set up a virtual File System
this.aioli.init().then(() => console.log("Aioli Initialized"));

// Mount a File object to the virtual file system within the WebWorker
this.aioli.mount({
    files: [ myFile ]
// Once mounted, launch main() function with command-line arguments "comp" and the File object
}).then(() => {
    return this.aioli.exec({
        args: ["comp", myFile],
    });
// Once it's done running, retrieve the output
}).then(d => {
    console.log(d);
});
```

### aioli.user.js
Custom user functions. Modify this file to allow the WebWorker to call custom functions you wrote (can't directly pass functions from main thread to WebWorker).

### aioli.worker.js
Code that runs inside a WebWorker. Should not need modifying unless you are adding features.
