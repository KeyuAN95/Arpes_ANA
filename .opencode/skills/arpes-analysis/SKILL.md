---
name: arpes-analysis
description: Use when loading ARPES data from h5/nxy files, extracting EDC or MDC curves, or analyzing band structure and Fermi surfaces
---

# ARPES Analysis

## Overview

Standardized ARPES (Angle-Resolved Photoemission Spectroscopy) analysis workflow using **pyarpes** for reproducible, publication-quality results.

## When to Use

- Loading `.h5`, `.nxy` ARPES data files
- Extracting **EDC** (vertical cut: fixed momentum, varying energy)
- Extracting **MDC** (horizontal cut: fixed energy, varying momentum)
- Calibrating Fermi energy, momentum axis, or background subtraction
- Plotting E-K slices, Fermi surfaces, band dispersion
- Exporting processed data or 300 DPI publication figures

## Quick Reference

### Data Loading
```python
from pyarpes.io import load_h5, load_nxy
data = load_h5("sample.h5")  # or load_nxy("sample.nxy")
```

### Preprocessing Pipeline (order matters)
1. **Fermi calibration**: `find_fermi_edge()` → `calibrate_fermi()`
2. **Background subtraction**: `subtract_background(kind='shirley')`
3. **Momentum calibration**: `calibration_from_lattice(a=, b=, alpha=)`

### Visualization Colormaps
| Plot Type | Colormap | Never |
|-----------|----------|-------|
| E-K slice, EDC, MDC | `magma` | rainbow/jet |
| Fermi surface | `viridis` | rainbow/jet |
| Difference maps | `RdBu` | - |

### Publication Figures
```python
plt.rcParams.update({'figure.dpi': 300, 'savefig.dpi': 300, 'font.size': 12})
```

## Lattice Parameters

Calibrate momentum axis using crystal structure:

```python
from pyarpes.io import calibration_from_lattice

cal = calibration_from_lattice(a=3.9, b=3.9, alpha=90)  # square lattice (Å)
cal = calibration_from_lattice(a=3.8, b=3.9, alpha=90)  # orthorhombic
cal = calibration_from_lattice(a=4.5, b=4.5, alpha=60)   # hexagonal (e.g., graphene)
data_cal = data_bg.calibrate(cal)
```

**Common materials:** Bi2212 (a≈3.8 Å), cuprates (a≈3.8-4.0 Å), FeSe (a≈3.8 Å), graphene (a≈2.46 Å)

## EDC vs MDC

| Curve | Fixed Axis | Varying Axis | Use Case |
|-------|-----------|--------------|----------|
| **EDC** | momentum k | binding energy | Peak positions, binding energies |
| **MDC** | energy E | momentum k | Band dispersion, Fermi velocity |

```python
# EDC at k=0 (Gamma point)
edc = data.sel(k=0.0, method='nearest').sum('phi')

# MDC at Fermi level
mdc = data.sel(energy=0.0, method='nearest').sum('energy')
```

## File Naming

**Raw data:** `{sample}_{mp_id}_{lattice_params}_{beamline}_{temp}K_{energy}eV_{polarization}.h5`

**Analyzed:** `{original}_analysis_{step}.h5`

Suffixes: `_fermi_cal`, `_bg_sub`, `_k_cal`, `_processed`

Example: `Bi2212_mp-2194_a3.8_b3.8_alpha90_BLS_30K_21.2eV_L.h5`

**Format:**
- `mp_id`: Materials Project ID (e.g., mp-2194)
- `lattice_params`: `a{val}_b{val}_alpha{val}` (Angstroms, degrees)
- Manual entry required — resolve mp-* IDs at materialsproject.org

## Axis Labels (Required Units)

| Axis | Unit | Label |
|------|------|-------|
| Energy | eV | "Binding Energy (eV)" or "E - E_F (eV)" |
| Momentum | Å⁻¹ | "Momentum (Å⁻¹)" or "k (Å⁻¹)" |
| Intensity | arb. units | "Intensity (arb. units)" |

## Common Mistakes

### ❌ EDC/MDC axis confusion
- EDC = vertical cut (constant k, energy on x-axis)
- MDC = horizontal cut (constant E, momentum on x-axis)

### ❌ Skipping background subtraction
- Raw data contains secondary electrons
- Subtract before quantification

### ❌ Wrong Fermi reference
- Verify with Au reference or known gap
- Check at multiple k-points

### ❌ Wrong colormap
- Always use `magma` for spectral intensity
- Use `viridis` for Fermi surfaces
- **Never** use rainbow/jet for publication

## Checklist

Before exporting/publishing:
- [ ] Fermi edge calibrated with reference
- [ ] Background subtracted (Shirley or Tougaard)
- [ ] Momentum axis calibrated with lattice parameters
- [ ] `magma` or `viridis` colormap applied
- [ ] All axes labeled with units
- [ ] Figure at 300+ DPI
- [ ] Data saved as `.h5`
- [ ] Code committed to git
