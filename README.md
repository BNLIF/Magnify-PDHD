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

### (Experimental feature) Channel Scan
```
./channelscan.sh path/to/rootfile
```
This is a wrapper for looping over channels where the channel list can be predefined in the `bad tree` or a text file. 

See detailed usage via `./channelscan.sh -h`.
