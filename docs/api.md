# API

- [createWorker()](#create-worker)
  - [Worker.load](#worker-load)
  - [Worker.write](#worker-write)
  - [Worker.writeText](#worker-writeText)
  - [Worker.read](#worker-read)
  - [Worker.remove](#worker-remove)
  - [Worker.transcode](#worker-transcode)
  - [Worker.trim](#worker-trim)
  - [Worker.concatDemuxer](#worker-concatDemuxer)
  - [Worker.run](#worker-run)
  - [Worker.terminate](#worker-terminate)

---

<a name="create-worker"></a>

## createWorker(options): Worker

createWorker is a factory function that creates a ffmpeg worker, a worker is basically a Web Worker in browser and Child Process in Node.

**Arguments:**

- `options` an object of customized options
  - `corePath` path for ffmpeg-core.js script
  - `workerPath` path for downloading worker script
  - `workerBlobURL` a boolean to define whether to use Blob URL for worker script, default: true
  - `logger` a function to log the progress, a quick example is `m => console.log(m)`
  - `progress` a function to trace the progress, a quick example is `p => console.log(p)`

**Examples:**

```javascript
const { createWorker } = FFmpeg;
const worker = createWorker({
  corePath: "./node_modules/@ffmpeg/core/ffmpeg-core.js",
  logger: m => console.log(m)
});
```

<a name="worker-load"></a>

### Worker.load(jobId): Promise

Worker.load() loads ffmpeg-core.js script (download from remote if not presented), it makes Web Worker/Child Process ready for next action.

**Arguments:**

- `jobId` jobId is generated by ffmpeg.js to identify each job, but you can put your own when calling the function.

**Examples:**

```javascript
(async () => {
  await worker.load();
})();
```

<a name="worker-write"></a>

### Worker.write(path, data, jobId): Promise

Worker.write() writes data to specific path in Emscripten file system, it is an essential step before doing any other tasks.

> Currently we found an issue that you should not have parallel Worker.write() as it may cause unexpected behavior, please do it in sequential for-loop like [HERE](https://github.com/ffmpegjs/ffmpeg.js/blob/master/examples/browser/image2video.html#L36)

**Arguments:**

- `path` path to write data to file system
- `data` data to write, can be Uint8Array, URL or base64 format
- `jobId` @see Worker.load()

**Examples:**

```javascript
(async () => {
  await worker.write(
    "flame.avi",
    "http://localhost:3000/tests/assets/flame.avi"
  );
})();
```

<a name="worker-writeText"></a>

### Worker.writeText(path, text, jobId): Promise

Worker.write() writes text data to specific path in Emscripten file system.

**Arguments:**

- `path` path to write data to file system
- `text` string to write to file
- `jobId` @see Worker.load()

**Examples:**

```javascript
(async () => {
  await worker.write("sub.srt", "...");
})();
```

<a name="worker-read"></a>

### Worker.read(path, jobId): Promise

Worker.read() reads data from file system, often used to get output data after specific task.

**Arguments:**

- `path` path to read data from file system
- `jobId` @see Worker.load()

**Examples:**

```javascript
(async () => {
  const { data } = await worker.read("output.mp4");
})();
```

<a name="worker-remove"></a>

### Worker.remove(path, jobId): Promise

Worker.remove() removes files in file system, it will be better to delete unused files if you need to run ffmpeg.js multiple times.

**Arguments:**

- `path` path for file to delete
- `jobId` @see Worker.load()

**Examples:**

```javascript
(async () => {
  await worker.remove("output.mp4");
})();
```

<a name="worker-transcode"></a>

### Worker.transcode(input, output, options, jobId): Promise

Worker.transcode() transcode a video file to another format.

**Arguments:**

- `input` input file path, the input file should be written through Worker.write()
- `output` output file path, can be read with Worker.read() later
- `options` a string to add extra arguments to ffmpeg
- `jobId` @see Worker.load()

**Examples:**

```javascript
(async () => {
  await worker.transcode("flame.avi", "output.mp4", "-s 1920x1080");
})();
```

<a name="worker-trim"></a>

### Worker.trim(input, output, from, to, options, jobId): Promise

Worker.trim() trims video to specific interval.

**Arguments:**

- `inputPath` input file path, the input file should be written through Worker.write()
- `outputPath` output file path, can be read with Worker.read() later
- `from` start time, can be in time stamp (00:00:12.000) or seconds (12)
- `to` end time, rule same as above
- `options` a string to add extra arguments to ffmpeg
- `jobId` @see Worker.load()

**Examples:**

```javascript
(async () => {
  await worker.trim("flame.avi", "output.mp4", 1, 2);
})();
```

<a name="worker-concatDemuxer"></a>

### Worker.concatDemuxer(input, output, options, jobId): Promise

Worker.concatDemuxer() concatenates multiple videos using concatDemuxer. This method won't encode the videos again. But it has its limitations. See [Concat demuxer Wiki](https://trac.ffmpeg.org/wiki/Concatenate)

**Arguments:**

- `input` input file paths as an Array, the input files should be written through Worker.write()
- `output` output file path, can be read with Worker.read() later
- `options` a string to add extra arguments to ffmpeg
- `jobId` check Worker.load()

**Examples:**

```javascript
(async () => {
  await worker.concatDemuxer(["flame-1.avi", "flame-2.avi"], "output.mp4");
})();
```

If the input video files are the same as the output video file, you can pass an extra option to speed up the process

```javascript
(async () => {
  await worker.concatDemuxer(["flame-1.mp4", "flame-2.mp4"], "output.mp4", "-c copy");
})();
```

<a name="worker-run"></a>

### Worker.run(args, jobId): Promise

Worker.run() is similar to FFmpeg cli tool, aims to provide maximum flexiblity for users.

**Arguments:**

- `args` a string to represent arguments
- `jobId` check Worker.load()

**Examples:**

```javascript
(async () => {
  await worker.run("-i flame.avi -s 1920x1080 output.mp4");
})();
```

<a name="worker-run"></a>

### Worker.terminate(): Promise

Worker.terminate() terminates web worker / worker\_threads, after terminate(), you cannot use this worker anymore.

**Examples:**

```javascript
(async () => {
  await worker.terminate();
})();
```