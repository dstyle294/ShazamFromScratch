# Shazam from Scratch: Facilitator Guide (60 min)

**The idea**: Shazam turns audio into a constellation of spectrogram peaks, hashes pairs of peaks into fingerprints, and identifies a song by finding which track's fingerprints agree on one consistent time offset.

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

Everything uses only numpy, scipy, and matplotlib, all preinstalled in Colab. The whole solution runs in under 10 s. By default the notebook generates its own synthetic songs. To use real music instead, see [Using your own songs](#using-your-own-songs).

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

**Buffer:** if you're behind, tell everyone to open the *Stuck?* box for TODO 3 and move straight to the aha plot. If you're ahead, use the "if you finish early" prompts at the end of Part 4.

### Hook talking points (not in the notebook)

| Idea | Why it breaks |
|---|---|
| Compare raw **waveforms** | Unknown time shift; the speaker and room change phase and amplitude; you'd have to slide the clip across every sample of every song. |
| Compare the **whole-clip FFT** | Throws away time. Most songs have similar broad spectra. |
| Compare the full **spectrogram** | Every pixel is affected by noise, and it's a huge amount of data to search. |
| ✅ Keep only the **loudest time-frequency points** | They survive noise, cheap speakers, MP3 compression, and room echo. |

## Expected results (synthesized library)

The main query picks a **random song and start time** on every run, so its exact numbers change each time. Everything else reproduces exactly.

- Song constellation: about 500 peaks (~17/s). Query: about 70 peaks, roughly 600 hashes.
- Noise slider at 0 dB: only **~35–50%** of clean peaks survive. That's fine, and Part 4 matches at −10 dB. "We only need enough survivors to agree" is the lesson.
- Main query: the correct song and start time, typically with a score around 30–150 while every other song stays in single digits.
- Stress test: clean, phone, and −10 dB are all identified (scores 39–202). The not-in-library and 5% speed-up clips both get a top score of 1, which suggests a "no match" rule of score ≥ ~10.
- Fan-out trade-off (presenter fact; this cell was cut from the notebook): at −10 dB, fan_out 1 gives 30% accuracy, 5 gives 93%, 10 gives 97%, and 20 gives 100%, with 50× the database size of fan_out 1.

## Prep checklist

- [ ] Upload `shazam_student.ipynb` to Drive / GitHub and make a shareable Colab link (or a QR code on the first slide).
- [ ] Do a dry run in a fresh Colab runtime: Runtime → Run all on the solution notebook.
- [ ] Have `shazam_solution_executed.ipynb` open in a second tab as a fallback.
- [ ] Check that laptop audio reaches the room's speakers (the hook depends on it).
- [ ] Optional: to use real music, follow [Using your own songs](#using-your-own-songs) and do a full dry run with your tracks.
- [ ] If you share songs through Google Drive, set `DRIVE_FOLDER_URL` (and `USE_REAL_AUDIO = True`) in `build_notebooks.py`, rebuild, and test the student link in a fresh Colab runtime ([Sharing songs through Google Drive](#sharing-songs-through-google-drive-colab)).

## Using your own songs

The notebook can fingerprint real music instead of the synthetic library.

1. **Add the files.** Create a folder named `songs/` next to the notebook and put the tracks in it.
   - **Supported formats:** `.wav`, `.mp3`, `.flac`, `.ogg`. Any other file in the folder, including `.m4a`, is silently skipped.
   - **Song names** come from the file names: `Martin Garrix - Animals.wav` appears as `Martin Garrix - Animals`.
   - **In Colab:** open the 📁 sidebar, create a `songs` folder, and upload into it. Uploads are deleted when the runtime resets, so re-upload after a restart.
2. **Turn it on.** In the first cell under the setup cell, set `USE_REAL_AUDIO = True` (`AUDIO_DIR` sets the folder name). Re-run from that cell down.
3. **Install the decoder locally.** Loading uses `librosa`, and Drive downloads use `gdown`. Colab already has both; locally, run `uv sync --extra audio`.
   - **For Colab students,** share the songs through Google Drive instead of having everyone upload them: see [Sharing songs through Google Drive](#sharing-songs-through-google-drive-colab).

How the files are processed:
- Only the **first 30 s** of each track is loaded (`SONG_SECONDS`), converted to mono at 11,025 Hz. Queries come from that window, so intros with silence or slow fades make weaker fingerprints.
- Use tracks that are **at least 30 s long**. The query and the noise-slider cell take clips from up to 30 s into a song.

Things to know:
- **Use at least 7 tracks.** The Part 4 stress test uses songs 1, 2, 4 and 6 by index, so it raises an `IndexError` with fewer. With 8–10 tracks you'll also get a convincing "wrong song" in the aha plot.
- The expected numbers above are for the synthetic library. Real tracks give different peak counts and scores, so do a dry run first.
- To load more than the first 30 s of each track, raise `SONG_SECONDS` in the settings cell. Loading and fingerprinting take longer.

## Sharing songs through Google Drive (Colab)

Students in Colab don't have your `songs/` folder, so the notebook can download it from a shared Google Drive folder with `gdown`. Colab normally has `gdown` and `librosa` preinstalled; if it doesn't, the notebook installs them.

**1. Make small copies of the songs.** Students only need what the notebook uses: the first 30 s of each track, in mono, at 11,025 Hz. The full tracks in `songs/` are 397 MB, and the trimmed copies are 7 MB in total. Small files also matter because Drive blocks a file with *"Too many users have viewed or downloaded this file recently"* when a whole room downloads large files at once.

```bash
mkdir -p drive_upload/songs drive_upload/samples
for f in songs/*; do n=$(basename "${f%.*}")
  ffmpeg -loglevel error -y -i "$f" -t 30 -ac 1 -ar 11025 -c:a pcm_s16le "drive_upload/songs/$n.wav"
done
cp samples/*.wav drive_upload/samples/      # optional: recordings for recording mode
```

`drive_upload/` is already built from the current `songs/` and `samples/`. With these copies, the samples match the same songs at the same offsets as with the full tracks. If you raise `SONG_SECONDS`, change `-t 30` to match.

**2. Upload and share.** Upload the **contents** of `drive_upload/` (the `songs` and `samples` folders) into one Drive folder. Then use Share → General access → **Anyone with the link** (Viewer), and copy the link.

```
<your Drive folder>/          ← share this folder's link
├── songs/      9 × .wav
└── samples/    recordings (optional)
```

The folder must contain `songs/` and `samples/` as subfolders. `gdown` copies the folder's contents into the notebook's working directory (`/content` in Colab). If the audio files sit directly in the shared folder instead of in `songs/`, the notebook won't find them. `gdown` downloads at most 50 files per folder.

**3. Point the notebook at it.** In the settings cell at the top of Part 0:

```python
USE_REAL_AUDIO = True
DRIVE_FOLDER_URL = "https://drive.google.com/drive/folders/<id>?usp=sharing"
RECORDING = "samples/human_nature_sample.wav"   # optional
```

To give these defaults to every student, make the change in `build_notebooks.py` and rebuild. Otherwise students paste the link in themselves. Any standard folder-link format works, with or without `?usp=sharing`.

The download runs on the first run of the "Load audio" cell. It's skipped when `songs/` already has audio in it, so re-running that cell doesn't download again. After a Colab runtime reset, everything is downloaded again. If `USE_REAL_AUDIO = True`, there's no link, and `songs/` is empty, the cell stops with an error message explaining what to do.

## Recording mode: identify a phone recording (bonus, not a dependency)

Colab can't easily capture the mic, so record on a phone and upload the file. The recording then replaces the generated query, and the notebook identifies it.

1. **Make sure the song is in the library.** Either use real music ([Using your own songs](#using-your-own-songs)) with the song in `songs/`, or play one of the synthetic songs from the notebook through the speakers.
2. **Record about 5–10 s** with the phone's voice memo app. The clip has to come from the part of the song that's loaded (the first `SONG_SECONDS`, 30 s by default).
3. **Upload it.** Locally, put it in `samples/`. In Colab, create a `samples` folder in the 📁 sidebar and upload it there, or include it in the shared Drive folder's `samples/` ([Sharing songs through Google Drive](#sharing-songs-through-google-drive-colab)).
4. **Point the notebook at it.** In the settings cell at the top of Part 0, set `RECORDING = "samples/your_clip.wav"`. `RECORDING_SECONDS` (default 5) sets how much of the recording to use, starting from its beginning. Then run all cells.

What changes with a recording:
- There's no known answer, so the "truth" line isn't printed. The aha plot compares the **best match** with the runner-up and draws the diagonal at the matched offset.
- The Part 1 plots and the noise slider show song 0 next to the query, because the notebook doesn't know which song it is yet.
- A score in the single digits means no match (see the "no match" threshold above).

To just check a file without re-running everything, run `identify(load_audio("your_clip.wav", seconds=10))` in any cell after Part 3.

**File formats:** `.wav` is safest. `load_audio` reads `.m4a` voice memos through ffmpeg, which Colab has. Locally you may need to install ffmpeg or convert the memo first: `ffmpeg -i "<pathname>" -acodec pcm_s16le -ar 44100 "<pathname to convert to>"`.

**Tested** with the 9 tracks in `songs/`: `human_nature_sample.wav` matched Human Nature (score 31) and `cc_mall_sample.wav` matched Coconut Mall (score 32). `human_nature_sample_noisy.wav` did **not** match (top score 2), so a quiet room matters.

## Discussion answers, quick reference

- **Why are peaks robust?** Noise spreads its energy across all bins; a note concentrates its energy in a few. Loud local maxima stay put.
- **Why pairs?** A single (f, t) isn't distinctive, and its t depends on where the clip starts. (f1, f2, Δt) is distinctive and shift-invariant.
- **Why fan-out instead of all pairs?** It's a trade-off: robustness (more hashes survive) versus db size and lookup cost. At −10 dB: fan_out 1 → 30%, 5 → 93%, 10 → 97%.
- **Why don't coincidental hits matter?** They don't agree on an offset. The evidence is alignment, not hit count.
- **Why does a speed-up break it?** Every frequency bin and every Δt shifts, so no hash matches exactly. A fix is to hash ratios (f2/f1, Δt₂/Δt₁) instead.
- **Scaling:** pack each hash into 32 bits (9 + 9 + 8 bits fits here), shard the inverted index across machines, keep it in RAM.
