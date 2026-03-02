# EDF Recording Issue - Root Cause Analysis

## Problem
Recording is not writing EDF files to disk.

## Root Cause
**No EEG stream is available on the LSL network.**

The `lsl_connect.py` subprocess waits up to 20 seconds for an EEG stream to appear. If no EEG stream is found, it exits without recording anything.

### Verification
Run this command to check for available LSL streams:
```bash
python check_lsl_streams.py
```

This should show you any available EEG streams. Currently, it reports:
```
❌ No LSL streams found!
```

## Solution
You need to **start an EEG data source** that publishes LSL streams before starting the calibration.

### For Testing
You can test with a simulated EEG stream:
```bash
python -m pylsl.examples.SendData
```

### For Production
Use your actual EEG device's LSL streaming interface (e.g., ActiCHamp LSL driver, OpenBCI LSL bridge, etc.)

The device should publish a stream with:
- **Type**: "EEG" or similar
- **Channels**: 7+ channels (F3, F4, C3, Cz, C4, P3, P4)
- **Sampling Rate**: Typically 250 Hz

## Fixed Issues
In addition to finding the root cause, I've also fixed some bugs that were preventing recording:

1. ✅ **EDF Header Finalization** - Added `update_header()` before closing EDF files to ensure proper file format
2. ✅ **Periodic Header Updates** - Added periodic header updates during recording to recover data if interrupted
3. ✅ **Auto-Recording** - Added `--auto-record` flag to start recording immediately when EEG stream connects
4. ✅ **Absolute File Paths** - Changed to use absolute paths for output files
5. ✅ **Stream Diagnostics** - Added diagnostics output to show available streams

## Next Steps
1. Start your EEG device/stream
2. Run `python check_lsl_streams.py` to verify it appears
3. Then run the calibration - recording should work automatically
