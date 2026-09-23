# Biondi Protocol Package (v1.7)

This is an executable, configuration-driven implementation of the **Generic Biondi Subaperture Tomography Protocol v1.7**. It is designed for Ed to hand to a coding agent together with a complex SAR SLC and an acquisition-specific run configuration.

It enforces the processing order:

1. verify the input file hash and axis convention;
2. construct matched Reference/Offset filter pairs in the Doppler-conjugate axis;
3. enforce the maximum 5% successive Offset-edge overlap;
4. reconstruct each matched complex R/O pair;
5. run registration controls before target processing;
6. measure same-pair target and nuisance vectors;
7. form `q_arc = Y - G_arc` as the canonical depth-focus input;
8. fit modal windows and focus the nominal-depth grid with acquisition-specific Kz;
9. save arrays, tables, configuration, and a manifest.

The package deliberately does **not** claim that a registration vector proves physical motion or that a focused depth proves a void, penetration, or absolute depth.

## What Ed uploads

Upload the entire `biondi_protocol_package` folder, the complex SAR file, and a completed copy of `configs/example_run.json` to the agent. The agent should run the commands below from the package folder.

```bash
python -m venv .venv
source .venv/bin/activate                # Windows: .venv\\Scripts\\activate
pip install -e .
pytest
biondi-validate configs/my_run.json
biondi-run configs/my_run.json
```

The run creates a new output directory containing:

- `results.npz`: every reconstructed pair, target/nuisance vectors, `q_arc`, modal table, Kz, and depth-score arrays;
- `manifest.json`: pair table, achieved overlap, validation result, input hash, and recurrence estimate;
- `config_used.json`: immutable copy of the exact configuration.

## What Ed must still do

### 1. Make the SAR file readable as a two-dimensional complex SLC

Supported direct input modes are:

- `npy`: a 2-D NumPy complex array;
- `tif`/`tiff`: a 2-D complex TIFF;
- `h5`/`hdf5`: provide `input.dataset_path` for the 2-D complex dataset.

For Capella/ICEYE proprietary containers, Ed must identify the actual complex dataset and place that path in the configuration—or export it once to a `.npy` complex SLC. He must not substitute an intensity image or a Quicklook.

### 2. Fill the acquisition-specific facts before looking at the target

The values that must come from the **current acquisition**, rather than another sensor/run, are:

- SHA-256, array dimensions, and row/column convention;
- which axis is azimuth/Doppler-conjugate;
- processed Doppler support and frequency convention;
- passband width, B-shift, pair count, and edge placement;
- slant range, incidence angle, and the LOS-orthogonal virtual baselines;
- operational wavelength/velocity model and nominal depth grid;
- target track and two or more nuisance arcs in native `[row, column]` coordinates.

For `baseline_perp_m`, the current version accepts either one scalar aperture span (it creates evenly spaced look baselines) or one value per pair. Use a real per-pair geometry vector whenever available.

### 3. Check every gate

Do not bypass a failure. In particular, stop when: the hash does not match; the axis mapping is uncertain; a pair extends beyond spectral support; the realized Offset-edge overlap exceeds 5%; registration controls fail; target/arc patches exceed the image; or the geometry/wavelength model is missing.

## Configuration notes

`maximum_successive_overlap_fraction` defaults to 0.05. With `b_shift_hz = 404`, the permitted overlap is 20.2 Hz and the next Offset-edge span must advance at least 383.8 Hz. The package writes the realized pair table to the manifest.

`target_track` and every nuisance arc are arrays of native SLC `[row, column]` points. Nuisance samples are combined with a median per side and the average of both sides. The complete nuisance arrays and the identity `Y = G_arc + q_arc` are retained.

## Important implementation status

This is a runnable reference package and includes synthetic tests for the overlap gate, registration controls, and Kz-vector formation. It is intentionally strict about invalid/missing inputs. Before publication-grade use, Ed should validate its product adapter against the vendor metadata and against known planted translations on the actual SLC; replace the scalar baseline approximation with state-vector-derived per-pair `B_perp`; and review the registration estimator for sub-pixel fidelity on the sensor in use.

## Agent instruction to paste with the files

> Treat `README.md` and the run JSON as the operational contract. Do not infer missing SAR geometry, axis conventions, mask frequencies, or depth-scale assumptions. First obtain a verified 2-D complex SLC, complete the configuration from the current acquisition metadata, run `pytest`, then `biondi-validate`, and stop on any failed gate. Run `biondi-run` only after those checks pass. Preserve every generated output and do not interpret depth scores as proof of a void or physical motion.
