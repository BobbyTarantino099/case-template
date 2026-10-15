<!-- Replace every <PLACEHOLDER> once the case has its video. Until then, this file is the
     process to follow. The first case's video is the worked example:
     https://github.com/BobbyTarantino099/steam-price-reception/tree/main/video -->

# Video: a <N>-second trailer of the case

A narrated, animated walk through the case: the question, the data, the finding, the
objection, and an invitation to read the case. It is built from the case's own files. No number
in it is typed by hand.

**Watch it:** [on YouTube](<https://youtu.be/ID>) (unlisted: reachable from this link and the
case page, not from search).
**Script:** [`guion.md`](guion.md). Read its *Notes on the claims* before changing a sentence.

The video is rendered with [HyperFrames](https://github.com/heygen-com/hyperframes) (HeyGen,
Apache 2.0): each scene is an HTML page animated with GSAP, captured frame by frame in headless
Chrome and encoded with FFmpeg.

## What is here, and what is not

```
video/
├── guion.md              # the script: narration, visuals, and why each claim holds
├── unir_partes.py        # 0. joins the take when it was recorded in parts (optional)
├── empalmar_voz.py       # 1. splices re-recorded sentences into the take (or copies it)
├── limpiar_voz.py        # 2. denoises the voice and sets its loudness
├── datos_video.py        # 3. pulls every number on screen from the case's files
├── mezclar_audio.py      # 4. lays the music bed under the voice
├── portada.py            # 5. takes the poster frame from the render
├── portada.jpg           # the poster the case page shows until play is pressed
└── composicion/          # the HyperFrames project
    ├── index.html        # mounts the scenes on one timeline, plays the mix
    ├── compositions/     # s1.html, s2.html … one per scene, each with its own clock
    ├── assets/datos.js   # written by datos_video.py — never edit by hand
    ├── assets/fonts/     # Inter, the same type as the figures (OFL, see LICENCIA.md)
    └── package.json      # pins the HyperFrames version: renders repeat over time
```

**Not versioned** (see `.gitignore`):

- **The voice recordings** (`voz*.wav`) and everything derived from them: the cleaned voice, the
  mix and the transcript. They are personal; the published video is their public form. Without
  them the video cannot be regenerated from this repository, and that is chosen on purpose.
- **The music** (`musica*.mp3`). Its licence allows using it inside a video, not redistributing
  the file on its own, the same reason the raw data is not stored here.
- **Renders** (`composicion/renders/`) are outputs.
- **`composicion/CLAUDE.md` and `AGENTS.md`**, the generic agent instructions `hyperframes init`
  writes into every project. Run it in an empty folder to get them again.

## Rebuild

Requirements: Python 3.10+ with pandas, Node 22+, FFmpeg on `PATH`. The HyperFrames CLI runs
through `npx` at the version pinned in `composicion/package.json`. Transcription uses its local
Parakeet model (`npx hyperframes models install parakeet`, ~640 MB once). Turn telemetry off once
with `npx hyperframes telemetry disable`. **Never run `npm run publish`**: it uploads the project
to HeyGen.

From the case root, in this order. Each step reads the previous one's output:

```bash
python video/unir_partes.py      # only if recorded in parts: Part 1.wav … -> voz.wav
python video/empalmar_voz.py     # voz.wav (+ fragments)         -> voz_empalmada.wav
python video/limpiar_voz.py      # -> composicion/voz_limpia.wav (-19 LUFS, mono)
python video/datos_video.py      # the case's files               -> composicion/assets/datos.js
python video/mezclar_audio.py    # voice + music                  -> composicion/mezcla.wav (-16 LUFS)

cd video/composicion
npx hyperframes transcribe voz_limpia.wav --engine parakeet --language en   # word timings
npm run check                    # lint, layout, motion and contrast: must pass
npm run render                   # -> renders/

cd ../..
python video/portada.py          # render -> video/portada.jpg
```

## How to make it, in the order that worked

1. **Write the script and check every claim against the data before recording.** Use
   `datos_video.py` to recompute the number behind each sentence, and write the result in
   *Notes on the claims*. Check its **direction** too, not only its numbers: in the second case a
   sentence with the right numbers said the opposite of what they showed ("the more young
   players vanish", when it was the veterans). In the first case two sentences of the take overstated what the
   analysis shows. Both were found only when the data behind each scene was recomputed, and each
   cost a re-recording and a splice. Checking first is cheaper. If a claim turns out to be
   wrong, the published case is probably wrong too: correct it as well.
2. **Record the whole take** in your own voice, with the same microphone, room and distance
   throughout, a second of silence at both ends, and a few seconds of the room alone. Synthetic
   voices (`hyperframes tts`, Kokoro) were tried and dropped: too robotic, and they skip the
   pauses the script relies on. Save it as `video/voz.wav`. Or record **one part per scene**
   (`Part 1.wav`, `Part 2.wav`, …), which the second case found easier: a mistake costs one
   part, not the take, and `unir_partes.py` levels and joins them.
3. **Clean it** with `empalmar_voz.py` and `limpiar_voz.py`. Measure the noise in a pause first:
   the denoiser's floor (`nf`) must sit below it.
4. **Transcribe** to get word timings. The transcript's *text* is unreliable (it misheard "fund"
   as "fan", "old" as "all"). Only its *times* are used, and the words on screen come from the
   script.
5. **Build one scene at a time and review it before the next one.** Scene times are the
   transcript time minus the scene's start on the root. `compositions/s1.html` and `s2.html` are
   examples of the patterns that carried the first case: words synced to the voice, a counter
   from `window.DATOS`, a closing card that holds.
   Animate a line's drawing with a widening clip, not `stroke-dashoffset`: in the frame-by-frame
   render the dash draw stays invisible and then jumps to the whole line.
6. **Music last**, once the cut is final: `mezclar_audio.py` reads the video's length from
   `index.html`, and the ducking follows the voice. Try two or three tracks and choose by ear.
7. **Render, upload, embed** (next section).

## Publishing

1. Upload the render to YouTube as **unlisted**. "Private" does not work: the page would show
   "Private video". Leave embedding allowed and mark it "not made for kids", or the embedded
   player is restricted. Put the link at the top of this file.
2. Run `python video/portada.py` with `SEGUNDO` set to a settled frame of the hook.
3. In the site repo, add the video to the case's front-matter:

   ```yaml
   video:
     youtube: "<11-character id>"
     seconds: <N>
     poster: "./images/video-poster.jpg"
   ```

   and run `python scripts/cases/sync_figures.py`, which copies `portada.jpg` next to the
   figures. The video then takes the hero figure's place at the top of the case page, and the
   hero still drives the index card and the link card. The page loads nothing from YouTube until
   the reader presses play.
4. Link the video from the case README.

## Music

<Track name — page link — licence.> Pixabay Music's licence allows use inside a video without
attribution, but some of its tracks are registered with YouTube Content ID. If the upload gets a
claim, the way out is the track's licence on its page. **Save the page link when you download
the track.**
