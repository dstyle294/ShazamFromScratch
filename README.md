# Shazam from Scratch: Facilitator Guide (60 min)

**Audience:** HKN inductees (EE/CS undergrads who know the FFT and Python).
**The one sentence:** Shazam turns audio into a constellation of spectrogram peaks, hashes pairs of peaks into fingerprints, and identifies a song by finding which track's fingerprints agree on one consistent time offset.

## Files

| File | Purpose |
|---|---|
| `shazam_student.ipynb` | Hand this out. It has 3 TODOs (`find_peaks`, `fingerprint`, `vote`), each followed by a ✅ check cell and a collapsed "Stuck?" solution box. |
| `shazam_solution.ipynb` | Complete version with no outputs. |
| `shazam_solution_executed.ipynb` | Complete version with every plot and audio clip already rendered. Project this if live code breaks. |
| `build_notebooks.py` | The single source for both notebooks. Edit this file, then run `python build_notebooks.py`. Don't hand-edit the notebooks. |

**Running locally** (instead of Colab), with [uv](https://docs.astral.sh/uv/):

```bash
uv sync                                    # creates .venv (add --extra audio for real-audio support)
uv run python -m ipykernel install --user --name shazam-from-scratch --display-name "Python (Shazam from Scratch)"
uv run python build_notebooks.py           # regenerate notebooks after editing the source
```

Everything uses only numpy, scipy, and matplotlib, all preinstalled in Colab. The whole solution runs in under 10 s. **No dataset download is needed:** the notebook synthesizes 8 original songs with fixed seeds, so every student sees the same results.

## Timeline (~25 min of hands-on coding inside the hour)

| Min | Block | Mode | Notes |
|---|---|---|---|
| 0–8 | Hook + why naive approaches fail | Talk | Run the setup cells, play `query` out loud, ask "how would you name this?" Walk through the table below. |
| 8–12 | Spectrograms | Talk + run | Run the Part 1 spectrogram cell. Point out the bright spots surviving in the noisy query. |
| 12–22 | **TODO 1** `find_peaks` + noise slider | **Code (~10 min)** | Payoff: the constellation map. Have people try `SNR_DB` = −10 and notice that the weak, high-frequency peaks are the ones that die. |
| 22–30 | **TODO 2** `fingerprint` | Talk 2 min, **code (~6 min)** | Explain why (f1, f2, Δt) is shift-invariant and why fan-out exists (see numbers below). |
| 30–40 | **TODO 3** `vote` | Talk 3 min, **code (~6 min)** | Explain the offset idea *before* they code it. |
| 40–48 | The aha plot | Talk | Scatter diagonal → histogram spike. Spend time here. |
| 48–60 | Stress test + wrap-up | Run + discuss | Discuss the "no match" threshold and the speed-up failure. Point to Wang 2003. |

**Buffer:** if you're behind at minute 30, tell everyone to open the *Stuck?* box for TODO 3 and move straight to the aha plot. If you're ahead, use the "if you finish early" prompts at the end of Part 4.

### Hook talking points (not in the notebook)

| Idea | Why it breaks |
|---|---|
| Compare raw **waveforms** | Unknown time shift; the speaker and room change phase and amplitude; you'd have to slide the clip across every sample of every song. |
| Compare the **whole-clip FFT** | Throws away time. Most songs have similar broad spectra. |
| Compare the full **spectrogram** | Every pixel is affected by noise, and it's a huge amount of data to search. |
| ✅ Keep only the **loudest time-frequency points** | They survive noise, cheap speakers, MP3 compression, and room echo. |

## Expected results (synthesized library, so these reproduce exactly)

- Song constellation: about 500 peaks (~17/s). Query: about 70 peaks, 634 hashes.
- Noise slider at 0 dB: only **~35–50%** of clean peaks survive. That's fine, and Part 4 matches at −10 dB. "We only need enough survivors to agree" is the lesson.
- Main query: Phase-Locked Love at 12.3 s, score 28 vs runner-up 2.
- Stress test: clean, phone, and −10 dB are all identified (scores 39–202). The not-in-library and 5% speed-up clips both get a top score of 1, which suggests a "no match" rule of score ≥ ~10.
- Fan-out trade-off (presenter fact; this cell was cut from the notebook): at −10 dB, fan_out 1 gives 30% accuracy, 5 gives 93%, 10 gives 97%, and 20 gives 100%, with 50× the database size of fan_out 1.

## Prep checklist

- [ ] Upload `shazam_student.ipynb` to Drive / GitHub and make a shareable Colab link (or a QR code on the first slide).
- [ ] Do a dry run in a fresh Colab runtime: Runtime → Run all on the solution notebook.
- [ ] Have `shazam_solution_executed.ipynb` open in a second tab as a fallback.
- [ ] Check that laptop audio reaches the room's speakers (the hook depends on it).
- [ ] Optional: to use real music, set `USE_REAL_AUDIO = True` and put 8–10 royalty-free tracks in `songs/` (e.g. from Free Music Archive). Then re-check the expected numbers above, since they will change.

## Live phone demo (bonus, not a dependency)

Colab can't easily capture the mic. Play a song from the notebook through the speakers, record 10 s on a phone's voice memo app, upload the file through the 📁 sidebar, and run `identify(load_audio("file.m4a", seconds=10))`. Room noise is unpredictable, so do this only if there's time.

## Discussion answers, quick reference

- **Why are peaks robust?** Noise spreads its energy across all bins; a note concentrates its energy in a few. Loud local maxima stay put.
- **Why pairs?** A single (f, t) isn't distinctive, and its t depends on where the clip starts. (f1, f2, Δt) is distinctive and shift-invariant.
- **Why fan-out instead of all pairs?** It's a trade-off: robustness (more hashes survive) versus db size and lookup cost. At −10 dB: fan_out 1 → 30%, 5 → 93%, 10 → 97%.
- **Why don't coincidental hits matter?** They don't agree on an offset. The evidence is alignment, not hit count.
- **Why does a speed-up break it?** Every frequency bin and every Δt shifts, so no hash matches exactly. A fix is to hash ratios (f2/f1, Δt₂/Δt₁) instead.
- **Scaling:** pack each hash into 32 bits (9 + 9 + 8 bits fits here), shard the inverted index across machines, keep it in RAM.
