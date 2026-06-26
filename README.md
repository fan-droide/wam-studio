# WAM Studio — latency-test fork

> Fork of [Brotherta/wam-studio](https://github.com/Brotherta/wam-studio).
> See the [upstream README](https://github.com/Brotherta/wam-studio#readme) for the full project description, citation, and Docker instructions.

## Purpose

This fork replaces WAM Studio's built-in threshold-based latency calibration
tool with [`<latency-test>`](https://github.com/idsinge/latency-test)
(`@adasp/latency-test`), which measures round-trip audio latency using an MLS
signal and cross-correlation.

It also fixes a formula bug in the original calibration: the old implementation
subtracted `outputLatency` from the measured round-trip, leaving a residual
offset in recorded tracks. This fork uses the full round-trip value directly,
which is what `SampleRegionRecorder` needs to align a recording correctly.

The fork is a research deliverable of the [Hi-Audio project](https://hiaudio.fr)
and serves as a working showcase for a proposed upstream PR to
[Brotherta/wam-studio](https://github.com/Brotherta/wam-studio).

## What changed

| File | Change |
|---|---|
| `public/src/Controllers/LatencyController.ts` | Rewritten — uses `<latency-test>` instead of the original threshold-based AudioWorklet peak-detector |
| `public/src/Audio/LatencyProcessor.js` | Deleted — replaced by the component |
| `public/package.json` | Added `@adasp/latency-test@1.2.0` |

Key behaviour changes:
- 3-run MLS/cross-correlation measurement instead of single-shot threshold detection
- Reliability gate: prior calibration left untouched if any run is unreliable or the user stops early
- Race-safe: stale-element guards prevent a rejected async operation from clobbering a newer calibration run
- Uses the main `AudioContext` (not a fresh one) — same output sink, sample rate, and `outputLatency` as the recording pipeline

## Quick start

```bash
cd public
npm install
cp /dev/null .env   # empty .env is fine — defaults apply
npm start           # → http://localhost:5002
```

Requires a microphone and headphones. The backend bank/plugin server is **not
needed** for the latency calibration demo. One failed login fetch at startup is
expected and harmless.

## Browser requirements

This app uses `SharedArrayBuffer`, which requires [cross-origin isolation](https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated).
The server **must** respond with:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

The webpack dev server (`npm start`) sets these automatically. For production,
the `netlify.toml` in this repo configures them for Netlify deployments, or run
`node server.js` (which sets them via Express).

You can verify cross-origin isolation is active in the browser console:

```js
crossOriginIsolated // must be true
```

## Live demo

https://charming-paletas-c95a3f.netlify.app

## Research record

- Implementation notes and findings:
  [`demos/wam-studio/NOTES.md`](https://github.com/idsinge/latency-test-examples/blob/main/demos/wam-studio/NOTES.md)
  in [idsinge/latency-test-examples](https://github.com/idsinge/latency-test-examples)
- `<latency-test>` component source and API docs:
  [idsinge/latency-test](https://github.com/idsinge/latency-test)

## Attribution

This fork builds on [WAM Studio](https://github.com/Brotherta/wam-studio) by
Michel Buffa and Antoine Vidal-Mazuy (Université Côte d'Azur / Inria). Please
cite the upstream work if you use this:

```
@inproceedings{buffa2023wam,
  title={WAM-studio, a Digital Audio Workstation (DAW) for the Web},
  author={Buffa, Michel and Vidal-Mazuy, Antoine},
  booktitle={Companion Proceedings of the ACM Web Conference 2023},
  pages={543--548},
  year={2023}
}
```

This fork is part of *Hybrid and Interpretable Deep Neural Audio Machines*
(Hi-Audio), funded by the European Research Council (ERC) under Horizon Europe
(grant agreement No. 101052978).
