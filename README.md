# Client Side Voice Changer

Client Side Voice Changer is a browser-only voice conversion app built with Next.js. It records or uploads speech, transcribes it locally, synthesizes speech from the transcript, applies lightweight voice effects, and lets the user preview and download the result as a WAV file.

No audio or transcript data is sent to an application server by this project. The AI and DSP pipeline runs in the browser with Web APIs, WebAssembly, and local model assets served from `public/models`.

## Features

- Record microphone audio in the browser.
- Upload an existing audio file.
- Transcribe speech with `Xenova/whisper-tiny.en`.
- Edit the transcript before generating output.
- Generate speech with the Kokoro ONNX model.
- Select from curated Kokoro voice styles.
- Tune pitch shift and vocal brightness.
- Preview source and generated audio waveforms.
- Download the generated output as `voice-changer-output.wav`.
- Run fully client-side after the static assets are loaded.

## Tech Stack

- Next.js 16 App Router
- React 19
- TypeScript
- Tailwind CSS 4
- `@huggingface/transformers` for browser transcription
- `onnxruntime-web` for ONNX inference
- Kokoro ONNX model assets served from `public/models/kokoro`
- Web Audio API for decoding, resampling, filtering, and WAV encoding
- WaveSurfer for waveform playback

## Project Structure

```text
src/app
  layout.tsx        Application metadata and root layout
  page.tsx          Main client-side voice changer UI
  globals.css       Global styles

src/components
  AudioRecorder.tsx     Microphone recorder UI
  AudioUploader.tsx     Audio upload UI
  ConversionPanel.tsx   Voice selection, tuning, and synthesis controls
  TranscriptEditor.tsx  Editable transcript UI
  WaveformPlayer.tsx    Audio preview player

src/hooks
  useAudioRecorder.ts   Browser recording state and MediaRecorder logic

src/lib
  audioUtils.ts         Audio decoding, DSP effects, and WAV encoding
  whisper.ts            Browser transcription model loading and execution
  tts                   Kokoro model loading, voices, and speech synthesis
  rvc                   Reserved placeholder for future conversion work

src/types
  audio.ts              Shared audio and pipeline state types

public/models
  kokoro                Local Kokoro model, tokenizer, config, and voices
  speaker.bin           Additional local model asset
```

## Requirements

- Node.js 20 or newer
- npm
- A modern browser with Web Audio API support
- Microphone permission for recording
- WebGPU is optional. The app can fall back to WASM where supported.

## Getting Started

Clone the Repository:

```bash
git clone https://github.com/luffy-2606/voice-changer.git
cd voice-changer
```

Install dependencies:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```


## Model Assets

The app expects Kokoro assets to be available under:

```text
public/models/kokoro
```

The current repository includes:

- `model.onnx`
- `config.json`
- `tokenizer.json`
- `tokenizer_config.json`
- voice embedding files under `public/models/kokoro/voices`

If the model files are missing or need to be refreshed, inspect `download_model.js` before running it so the downloaded files match the model paths used by the app.

## How The Pipeline Works

1. The user records audio or uploads a file.
2. `src/lib/audioUtils.ts` decodes and resamples the audio for transcription.
3. `src/lib/whisper.ts` runs browser transcription.
4. The transcript appears in `TranscriptEditor` and can be edited.
5. `ConversionPanel` sends the final text and selected voice to Kokoro.
6. `src/lib/tts/kokoro.ts` phonemizes text, runs ONNX inference, and returns WAV audio.
7. `applyVoiceEffects` applies pitch and brightness processing with Web Audio.
8. The generated audio is displayed in `WaveformPlayer` and can be downloaded.

## Privacy Model

This app is designed around client-side processing:

- Microphone capture happens in the browser.
- Uploaded audio is processed in the browser.
- Transcription runs in the browser.
- Speech synthesis runs in the browser.
- DSP effects run in the browser.

The app does not define custom API routes for audio upload, transcription, synthesis, or storage.

## Browser Notes

The first run can take time because model assets must load into the browser. WebGPU support may improve performance, but the app should still present a WASM fallback status when WebGPU is unavailable.

Large audio files can consume significant memory during decoding, transcription, synthesis, and waveform rendering. Short speech clips are recommended.

## Troubleshooting

### The app fails to load model files

Check that `public/models/kokoro/model.onnx` and the selected voice embedding file exist. The browser fetch path is based on `/models/kokoro`.

### Transcription returns no text

Try a clearer recording, move closer to the microphone, or upload a shorter speech-only file.

### The generated audio sounds too high or too bright

Open tuning controls and reduce Pitch Shift or Vocal Brightness. Setting Pitch Shift to `0` keeps the Kokoro output closer to the selected voice.

### Build fails because of browser-only dependencies

Check `next.config.ts`. It stubs Node.js built-ins and blocks `onnxruntime-node` from entering the browser bundle.

## Scripts

```json
{
  "dev": "next dev --webpack",
  "build": "next build --webpack",
  "start": "next start --webpack",
  "lint": "eslint"
}
```

## License

No license file is currently included. Add one before publishing or distributing the project.
