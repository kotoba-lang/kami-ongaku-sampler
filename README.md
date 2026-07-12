# kami-ongaku-sampler

Sample zone/mapping, round-robin variation cycling, velocity-layer lookup,
and a streaming-lifecycle state machine for portable sample-playback
engines (the Kontakt-equivalent mapping model), per
[ADR-2607121400](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607121400-kami-ongaku-eizo-commercial-grade-cljs-stack.md).

Part of the `kotoba-lang` `ongaku` (music production) stack, alongside
[`kami-ongaku-notation`](https://github.com/kotoba-lang/kami-ongaku-notation),
[`kami-ongaku-sequencer`](https://github.com/kotoba-lang/kami-ongaku-sequencer),
and [`kami-ongaku-project`](https://github.com/kotoba-lang/kami-ongaku-project).

## Scope (v0)

- Zone/layer data model: key range x velocity range -> a round-robin group
  of sample variations, with per-layer pitch offset, gain, and pan.
- Deterministic round-robin cycling (trigger-count modulo group size, not
  `rand` — testable and reproducible).
- Velocity-layer + key-range lookup with correct inclusive-boundary
  handling.
- Overlap validation: a sample map with two layers overlapping in both key
  range and velocity range is rejected. Layers sharing a key range at
  disjoint velocity bands (the normal multi-velocity-layer case) are not
  an overlap.
- Streaming lifecycle state machine (`:unloaded`/`:loading`/`:ready`/
  `:streaming`/`:error`) with an explicit valid-transition table.

**Not in v0**: no audio decoding, no disk/network I/O, no playback. This is
purely the mapping/lookup data model and lifecycle state machine that a
real sampler engine would sit on top of.

## Usage

```clojure
(require '[kami.ongaku.sampler :as sampler])

(def layer
  (sampler/make-layer
   {:key-low 60 :key-high 60 :vel-low 64 :vel-high 127
    :variations [(sampler/make-variation :snare-hard-1)
                 (sampler/make-variation :snare-hard-2)]}))

(sampler/trigger-sample [layer] 60 100 0)
;; => {:sample-ref :snare-hard-1, :pitch-offset 0.0, :gain 1.0, :pan 0.0}
```

## Test

```
clojure -M:test
```
