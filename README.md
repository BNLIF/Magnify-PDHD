# Magnify

## A magnifier to investigate raw and deconvoluted waveforms.

### Preprocess
```
./preprocess.sh /path/to/your/magnify.root
```
This is a wrapper for merging magnify histograms from all APAs. See more detailed description through "./preprocess.sh -h".


### Usage

```
cd scripts/
root -l loadClasses.C 'Magnify.C("path/to/rootfile")'
# or
root -l loadClasses.C 'Magnify.C("path/to/rootfile", <threshold>, "<frame>", <rebin>)'
```

The second argument is the default threshold for showing a box.

The third, optional argument names which output from the signal processing to display.  Likely names are:

- `decon` produced by the Wire Cell prototype (default).
- `wiener` produced by the Wire Cell toolkit, used to define ROI or "hits".
- `gauss` produced by the Wire Cell toolkit, used for charge measurement.

The call to ROOT can be be called somewhat more easily via a shell
script wrapper.  It assumes to stay in the source directory:

```
/path/to/magnify/magnify.sh /path/to/wcp-rootfile.root
# or
/path/to/magnify/magnify.sh /path/to/wct-rootfile.root 500 gauss 4
```

### Example files

An example ROOT file of waveforms can be found at twister:/home/wgu/Event27.root

If one omits the file name, a dialog will open to let user select the file:
```
cd scripts/
root -l loadClasses.C Magnify.C
```

### Event / APA Navigation

The control window has a second row with a **Navigation** group for switching between events and APAs without restarting ROOT:

| Widget | Action |
| --- | --- |
| `apa` combo | Switch to a different APA (0–3) of the current event |
| `event` combo | Jump directly to any discovered event |
| `<` button | Previous event (same APA) |
| `>` button | Next event (same APA) |

Events are auto-discovered at startup by scanning the parent directory of the opened file for subdirectories matching `<run>_<event>` (e.g. `027409_1`) that contain at least one `magnify-run<run>-evt<event>-apa0.root` file. Directories with extra suffixes (e.g. `027409_1_sel1`) are skipped. The list is sorted by `(run, event)`.

Switching tears down the active `Data`, opens the new file, and redraws all 9 pads in place; Region Sum / RMS Analysis sub-windows are hidden and their caches are reset.

### RMS & FFT noise analysis (batch)

Compute per-channel noise RMS and frequency spectra for one or more Magnify files:

```bash
./scripts/run_rms_analysis.sh input_data/magnify-run.root
# Produces: input_data/magnify-run.root.rms.root
```

The cache file contains per-plane RMS TTrees (`rms_u/v/w`) and FFT TH2Fs
(`fft_u/v/w`, channel × frequency in MHz).  It is loaded automatically by the
viewer when you click **RMS Analysis** in the control bar.  See `docs/RMS_FFT.md`
for the full workflow and panel description.

### Resampling Check Window

Click **resamp check** in the General group of the control window to open a 1×3 canvas comparing the original (512 ns/tick) and Wire-Cell resampled (500 ns/tick) waveforms for the active channel, one pad per plane (U/V/W).

Each pad shows both waveforms as TGraphs on a common **time-in-µs** x-axis:

| Trace | Source histogram | Tick width | Style |
| --- | --- | --- | --- |
| Blue open circles | `hu_orig` / `hv_orig` / `hw_orig` (raw ADC, baseline-subtracted) | 512 ns | `TGraph "PL"` |
| Red filled triangles | `hu_raw` / `hv_raw` / `hw_raw` (Wire-Cell resampled) | 500 ns | `TGraph "PL"` |

Because both traces are drawn as discrete sample markers connected by a thin line (not stair-step histograms), you can directly check whether each resampled point lands on the curve interpolated through the original samples.

**Sync with the main panel:**

- **Channel:** changing the channel entry redraws the matching plane's pad automatically; the other two pads retain their last-drawn channel (same behavior as the bottom three waveform pads in the main canvas).
- **X range:** editing the `x range` entries and pressing Enter applies the range to both the main panel and the resamp-check canvas (converted to µs: ticks × 0.500).
- **Y range:** editing the `y range` entries and pressing Enter applies to both. Setting both entries to 0 restores auto-range from the waveform data.

Switching to a different event or APA closes the resamp-check canvas automatically to prevent access to freed waveform data.

### (Experimental feature) Channel Scan
```
./channelscan.sh path/to/rootfile
```
This is a wrapper for looping over channels where the channel list can be predefined in the `bad tree` or a text file. 

See detailed usage via `./channelscan.sh -h`.
