# A Blurred Line's pipeline

```text
$GAME_DIR (.lmu .ldb .lmt)    --ditto unpack (just dump)-->  data/ (JSON)
data/ + patches/manual/
+ images, MIDI and WAV            --the converter, rm2k-snes-convert--> build/<rom>/
  ($GAME_DIR, $RTP_DIR)     (its stages compiled in: wally,   mapdata.inc, musicdata.inc,
                                     events, screens, images,
                                     battle, mapdata, patches, audio)
                                                                  abl.msu (battle backdrops),
                                                                  patches/ (decisions)
the runtime + those .inc          --the assembling---------------> abl.sfc + abl.sfc.sym
                                    (bank packing, ca65 + ld65)
abl.sfc + MIDI or replacements    --the MSU-1 tracks-------------> abl-N.pcm
                                    (fluidsynth, ffmpeg)
every cartridge                   --at the end-------------------> build/advocate.json

just cartridges release  advocate build release: all of it, with the MSU-1 tracks,
                         in build/release/
just cartridges release --write-patches
                         and copies release's decisions to patches/auto/
```

From the dump on it is one process, `advocate build`, which calls the
converter, the assembling and the tracks as libraries; until 2026-11-25
it was `rm2k-snes-cartridges`, which ran `rm2k-snes-convert` and
`rm2k-snes-audio msu`, and the patches were copied by
`rm2k-snes-patches build/release`.

The tools of each arrow and the runtime belong to the tools' repository,
the-advocate, and [Internals](../../internals/README.md) says how each one
works; the same road with generic names is [the
pipeline](../../internals/pipeline.md).
