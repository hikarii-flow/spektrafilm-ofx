# spektrafilm OFX

spektrafilm OFX is a native macOS OpenFX plugin project built from the
`spektrafilm` film-simulation codebase. It is intended for host applications
such as DaVinci Resolve and provides Metal-accelerated film, print, scan, grain,
halation, diffusion, color-management, and LUT-export workflows inside an OFX
plugin. The project tries to match Andrea Volpato's original implementation as 
closely as possible and uses the python codebase as scientific reference as best as possible.

## Relationship to Andrea Volpato's spektrafilm

This project is an expansion of
[spektrafilm by Andrea Volpato](https://github.com/andreavolpato/spektrafilm).
The original project provides the research foundation, Python implementation,
profile-generation workflow, and much of the film-density modeling direction.

The OFX work in this directory ports and extends that idea into a native plugin
for video applications. The main goals are:

1. Stay true to the spectral and density-based character of the original model.
2. Expand upon the controls for photochemical developemnt simulation (e.g. adding push/pull)
3. Make the pipeline usable in professional video and finishing workflows.
4. Keep spacial effects resolution-independant (1080p and 2160p sample the same virtual film negative, just at different densities)
5. Leverage Apple's Metal GPU stack for performance-sensitive stages (and CUDA for future Windows workflows).
6. Provide ready to use OFX binaries for non developers.
7. Keep tools in active development and research controls in a separate dev
   build.

## Naming

While browsing this repository, you may come across different terms when referring to the software.
The naming is intentionally split between the underlying framework and the built plugin 
"flavors":

| Name | Meaning |
| --- | --- |
| `spektrafilm` | The code framework and reference film-simulation project by Andrea Volpato. |
| `spektrafilm flow` | A streamlined OFX plugin flavor focused on the main creative controls and a simpler grading workflow. Intended for beginner users. |
| `spektrafilm` | A fuller OFX plugin flavor with more professional controls exposed while still hiding unstable/in-development tools. |
| `spektrafilm dev` | The development/reference OFX flavor used for research and development, verification, and tuning internal controls. It is built locally, but it is not packaged by the normal public download target. |

## Binary Downloads and Product Page

Prebuilt binaries, release information, and product-facing documentation are available at:

<https://spektrafilm.114c.de>

The public binary packages are intended for users who only want to install the
OFX plugin. Building from source is mainly useful for development, verification,
custom resources, or local experimental builds.

## Current Platform Status

The project currently targets macOS only.

The renderer is Metal-first, and the build requires Apple's Metal toolchain.
Windows and Linux builds are not currently supported by this OFX project, however
they are in early stages of development and scheduled for a future release.

## Main Features

The OFX plugin adds several workflow and model extensions on top of the
base `spektrafilm` framework. For a general project overview visit 
[Andrea Volpato's page](https://github.com/andreavolpato/spektrafilm) or read the documentation in [`documentation/`](documentation/).

### Color Management

The plugin includes explicit input and output color-management controls for
common camera, scene-linear, scene-log, SDR display, and HDR output paths. The
pipeline decodes input into a scene-referred working representation, runs the
film and print model, then encodes the selected output format.

The available output formats are:

| Format | Purpose |
| --- | --- |
| `Display Out SDR` | A finished display-referred SDR output. |
| `Scene Handoff` | A scene-referred handoff for continued grading downstream. (Experimental) |
| `Display Out HDR` | A finished display-referred HDR output using Rec.2100 PQ or HLG controls. |

### LUT Export

The `Manage` group includes a LUT export path for display-referred SDR looks.
Exports are rendered through the native Metal color pipeline as `.cube` LUTs. 
Spatial and stochastic effects such as grain, halation, diffusion, scanner blur, 
and similar image-dependent operations are excluded from the exported LUT
as they go beyond the scope of LUT capabilites.

The LUTs are designed to easily work for dailies, on-set monitoring, cross-platform editing and many other workflows.

### Push and Pull Modes

The film-development controls include two push/pull approaches:

| Mode | Description |
| --- | --- |
| `Standard` | The more straightforward gamma/timing-style push/pull model. It is intended to be stable and predictable. |
| `Experimental` | A layer- and tone-region-dependent warp that often achieves looks closer to real push/pull references. |

Print/paper push and pull is separate from film push and pull. Film push/pull
acts around negative development. Print push/pull acts around paper development.

### Printer Lights and ST 2065-2

The project contains a printer-light system for APD-based print timing comparable to motion picture printers. 
It uses printer points where one point equals `1/12` stop of light, matching the common lab timing
unit.

This feature depends on SMPTE ST 2065-2 Academy Printing Density data. The relevant
CSV files are licensed standards material and are not redistributed in this
public repository.

Public source builds without those CSV files still build successfully, but the
printer-density mode and printer-point controls will be disabled and the build
prints a clear notice when the files are missing.

If you have your own licensed copies of the CSV files attached to ST 2065-2, place them here
before configuring the build:

```text
Resources/data/standards/smpte_st_2065_2/st2065-2a-2020.csv
Resources/data/standards/smpte_st_2065_2/st2065-2b-2020.csv
```

Then re-run CMake from a clean build directory. When both files are present, the
generated profile counts header enables the Academy Printer Density path and the
printer-light controls become available in the relevant plugin flavors.

### Bleach Bypass Controls (Experimental)

The native renderer has experimental negative and print bleach-bypass controls
available in the `spektrafilm dev` build. They are an attempt to model retained silver in the
film or print path, but this area is not yet backed by enough stock- and/or
process-specific measured data.

For that reason, these controls should not be treated as representative lab
controls but as a playful first attempt at modeling this process. They are not exposed 
in the normal public `spektrafilm flow` and `spektrafilm` builds.

## Project Layout

Important paths in this directory:

| Path | Purpose |
| --- | --- |
| `CMakeLists.txt` | Main build definition for the OFX bundles, Metal library, generated data, harnesses, and download ZIP targets. |
| `build_macos.sh` | Convenience build script for local macOS builds and public package ZIP generation. |
| `src/SpektraFilmPlugin.cpp` | OFX entry points, parameter definitions, flavor visibility, render dispatch, defaults, clipboard handling, and LUT export wiring. |
| `src/SpektraMetalRenderer.mm` | Objective-C++ Metal renderer implementation and CPU-side render orchestration. |
| `src/SpektraMetalRenderer.h` | Renderer API used by the OFX host side and the local harnesses. |
| `src/SpektraParameters.h` | Shared render parameter types and enums. |
| `src/SpektraProfileCurves.h` | Declarations for generated stock/profile tables. |
| `src/SpektraTooltips.h` | User-facing control help text. |
| `shaders/SpektraFilm.metal` | Metal kernels for the film, print, scan, grain, halation, diffusion, and utility passes. |
| `Resources/data/profiles/` | Self-contained film and paper profile JSON files used by the OFX build. |
| `Resources/data/filters/` | Filter data used for enlarger, print, neutral filters, heat absorption, and lens transmission. |
| `Resources/data/luts/` | Spectral upsampling LUT resources used during native table generation. |
| `Resources/data/standards/` | Optional standards-derived data. Licensed ST 2065-2 CSVs belong here. |
| `Resources/icons/` | SVG and PNG plugin icons. |
| `Resources/Info.plist.in` | macOS bundle plist template. |
| `Resources/plugin_manifest.json.in` | Plugin manifest template copied into each bundle. |
| `tools/generate_profile_curves.py` | Generates native C++ profile tables, color-space tables, APD tables, and the Hanatos LUT resource. |
| `tools/ofx_stock_lists.py` | Film and paper stock ordering for OFX plugins. |
| `tools/export_reference_cases.py` | Exports reference cases from the Python model for comparison work. |
| `tools/SpektraMetalPerfHarness.mm` | Synthetic Metal performance harness for debugging and performance hunting. |
| `tools/SpektraMetalEvaluationHarness.mm` | Native evaluation harness. |
| `tools/SpektraVariantGenerator.mm` | Generates rendered variants for stock/look inspection (used for generating images of stocks for product website). |
| `tests/` | Python tests for build wiring, resource generation, parameter metadata, and source invariants. |
| `third_party/openfx/` | Vendored OpenFX SDK headers and support code. (OFX_Release_1.5.1)|
| `Legal/` | Binary distribution notices, exported LUT license terms, and third-party notices. |
| `documentation/` | Manual and user-facing documentation for the OFX plugin. |

## Build Requirements

The source build expects:

1. macOS.
2. Xcode Command Line Tools or Xcode.
3. Apple's Metal toolchain available through `xcrun`.
4. CMake `3.24` or newer.
5. Python `3.13` with the repository's Python dependencies installed.
6. libpng discoverable by CMake for the variant generator target.
7. OpenFX SDK headers (OFX_Release_1.5.1).

The OFX build prefers the repository virtual environment at `../../.venv/bin/python`
when it exists. Otherwise CMake falls back to the Python interpreter found by
`find_package(Python3)`.

## Setup From a Fresh Checkout
For ease of use, I developed this project from within Andrea's spektrafilm repository root. To follow the below instructions, pull the latest version of spektrafilm and place the contents of this repo at OFX/SpektraFilm.

From the spektrafilm root, create or sync the Python environment first. This
project uses the Python package for build-time table generation.

Using `uv`:

```sh
uv sync --extra dev
```

Or with a manually managed Python 3.13 environment:

```sh
python -m pip install -e ".[dev]"
```

Then build the OFX project:

```sh
cd OFX/SpektraFilm
./build_macos.sh
```

The script configures CMake, builds the plugin targets, and produces ZIP
packages for `spektrafilm flow` and `spektrafilm`.

## Manual CMake Build

For a more explicit build:

```sh
cd OFX/SpektraFilm
cmake -S . -B build
cmake --build build --parallel
```

To package the download ZIPs:

```sh
cmake --build build --target spektrafilmDownloadZip spektrafilm_flowDownloadZip --parallel
```

To install the built OFX bundles into the system OFX plugin directory:

```sh
cmake --install build
```

The install destination is:

```text
/Library/OFX/Plugins
```

Depending on your system permissions, installation may require elevated rights.

## Build Outputs

Normal local build outputs are written under:

```text
OFX/SpektraFilm/build/
```

The ZIP targets write:

```text
website/public/downloads/spektrafilm_flow-OFX-macOS.zip
website/public/downloads/spektrafilm-OFX-macOS.zip
```

Each ZIP contains the OFX bundle, `install_instructions.txt`, a root-level
`manual.pdf`, and the distribution notices under `Legal/`.

The `spektrafilm dev` development flavor is built as a local target, but the default
download packaging script does not create a `spektrafilm_dev` ZIP by default as it is not intended for distribution but internal validation only.

## Resource Generation

The OFX project is designed to be self-contained at build time. It does not read
profile data from the root Python package (spektrafilm) at runtime. Instead, CMake runs the
generation tools and bundles the generated resources into each OFX bundle.

Generated resources include:

| Resource | Source |
| --- | --- |
| `SpektraGeneratedProfileCurves.cpp` | `Resources/data/profiles/`, filters, standards data, LUT data, and `tools/ofx_stock_lists.py`. |
| `SpektraGeneratedProfileCounts.h` | Film/paper counts, default indices, and whether the ST 2065-2 APD path is enabled. |
| `SpektraHanatos2025Spectra.f32` | `Resources/data/luts/spectral_upsampling/irradiance_xy_tc.npy`. |
| `SpektraFilmData.spkdata` | Compiled profile/resource payload generated by `tools/compile_ofx_data.py`. |
| `SpektraFilm.metallib` | Compiled Metal kernels from `shaders/SpektraFilm.metal`. |

## Plugin Flavors

The build defines three OFX bundle targets:

| Target | Artifact name | Bundle label | Plugin identifier | Public package |
| --- | --- | --- | --- | --- |
| `spektrafilm_flow` | `spektrafilm_flow` | `spektrafilm flow` | `org.spektrafilm.flow` | Yes |
| `spektrafilm` | `spektrafilm` | `spektrafilm` | `org.spektrafilm` | Yes |
| `spektrafilm_dev` | `spektrafilm_dev` | `spektrafilm dev` | `org.spektrafilm.dev` | No |

All three are compiled from the same source. Flavor-specific behavior is
controlled through compile definitions and parameter visibility rules in
`src/SpektraFilmPlugin.cpp`.

## Testing

The OFX tests are Python tests that inspect source, generated wiring, resource
expectations, and build invariants. From this directory:

```sh
python -m pytest tests -q
```

## Legal and Redistribution Notes

The source code in this repository is licensed separately from the official
binary distribution notices. See:

```text
LICENSE.txt
Legal/SPEKTRAFILM_OFX_LICENSE.txt
Legal/SPEKTRAFILM_OFX_LUT_LICENSE.txt
Legal/THIRD_PARTY_NOTICES.txt
```

Important practical points:

1. The public source tree does not redistribute licensed SMPTE ST 2065-2 CSV
   files.
2. Official binary distributions may include bundled resources covered by the
   notices in `Legal/`.
3. LUT files exported from the plugin are governed by
   `Legal/SPEKTRAFILM_OFX_LUT_LICENSE.txt`.
4. The vendored OpenFX SDK carries its own notices under `third_party/openfx/`.

## Development Notes

The plugin is still an active development project. The public flavors prioritize
controls that are useful and reasonably defensible in grading workflows. The dev
flavor keeps deeper controls available so that modeling decisions can be tested
without committing every experiment to the public UI.

Areas that are intentionally treated as research rather than finished product
behavior include advanced grain synthesis via monte carlo raytracing, some scanner refinements, and the
bleach-bypass model. Improvements in those areas depend on measured data or clear references, any contributions are very welcome

Hope you enjoy and make some awesome art!
