# Wedge: design proposal

**Status:** Proposal, September 2026. No implementation or performance claims yet.

## Problem statement

The North American Mesoscale (NAM) forecast system is scheduled for retirement as NOAA transitions to RRFS. Forecasters in the Carolinas and Mid-Atlantic have found NAM guidance useful in some cold-air damming (CAD) or wedge setups. A shallow layer of cold air against the Appalachians can make precipitation type and surface temperatures particularly sensitive to model behavior. We want to preserve and test that useful guidance rather than assume a successor produces equivalent results in these cases.

Running NOAA's complete operational NAM suite independently would be a much larger undertaking than our immediate goal. Wedge proposes a **smaller geographic domain at approximately 2 km grid spacing**, centered on the Carolinas and including enough of the Appalachians, Mid-Atlantic, and upstream environment to represent CAD formation and evolution. The exact boundaries, resolution, forecast length, and driving data are research decisions, not settled specifications.

Public sources now provide a concrete starting point: NOAA's operational NAM v4.2.13 code archive and Matthew Pyle's NAM repository. The core, workflow, configuration, and fixed files need to be inventoried and reproduced on accessible hardware. A regional simulation requires valid initial conditions and lateral boundary conditions even when its footprint is small. Getting a model to run is only the first step; its forecasts must be evaluated against historical events and observations.

## Goals

1. **Preserve a reproducible baseline.** Record exact source revisions, build dependencies, NAM configuration, data inputs, and a historical case that can be reproduced. Attribute upstream material and track its licensing separately.
2. **Explore a CAD-focused regional configuration.** Start from the NAM/NMMB code and investigate a smaller, approximately 2 km domain. Choose boundaries with an atmospheric modeler so the model retains the conditions responsible for the wedge.
3. **Run and validate historical forecasts first.** Compare forecasts with observations, original NAM output where available, and RRFS or other relevant guidance. Assess near-surface temperature, cold-layer depth, precipitation type, timing, and event-specific biases. Publish methods and unfavorable results too.
4. **Make the engineering accessible.** Document a portable build, a repeatable input and run pipeline, resource use, forecast latency, and failure modes. Measure compute cost after a real run; no cost estimate is a commitment.
5. **Plan a path to a modern implementation.** Evaluate Rust and Go for portable tooling, workflow orchestration, data preparation, and eventually model components. Port incrementally behind reference-output tests so modernization does not silently change the forecast.
6. **Keep the project open.** Develop in public, accept scientific and engineering contributions, and publish original project work under MIT where we have the rights to do so.
7. **Offer a simple interface once forecasts are trustworthy.** Make selected fields and run metadata easy to inspect, alongside raw output and enough provenance to interpret it.

## Non-goals

- Reproduce the entire operational NAM suite, its nationwide coverage, all nests, or every forecast product.
- Claim that 2 km resolution by itself improves CAD forecasts or preserves NAM's specific strengths. That is a hypothesis to test.
- Present an experimental run as official NOAA/NWS guidance or as a replacement for warnings, forecasts, and safety decisions.
- Promise live forecasts for the coming winter before a portable build, input pipeline, and historical validation exist.
- Perform a big-bang rewrite of atmospheric dynamics before the original system has a reproducible baseline and numerical parity tests.
- Put an AI model in the forecast physics. AI tools may help with engineering; scientific results require reproducible, human-reviewed validation.
- Assume that an MIT license on this repository changes the terms of NOAA or third-party code, dependencies, data, or redistributed assets.

## Proposed first milestone

Reproduce **one historical CAD event** with the existing NAM code and known inputs, on a supported or ported build. Archive the configuration and compare model output against the original NAM run and observations. This establishes a reference before changing geography or grid spacing.

Only then attempt a smaller domain near 2 km and quantify the effects of the changed initialization, boundaries, grid, and physics settings. A successful experiment should finish within a useful forecast window, generate inspectable output, and have documented validation. An initial forecast may use a parent model to supply boundary conditions; which model and fields are appropriate remains open.

In parallel with the regional experiment, inventory the legacy code by responsibility: numerical kernels and physics, input and boundary preparation, workflow control, postprocessing, and user-facing services. Capture reference outputs at component boundaries and record acceptable numerical tolerances. This test corpus becomes the compatibility contract for modernization.

Begin the port with components that can change without altering atmospheric behavior, such as configuration, workflow orchestration, data acquisition, run metadata, and selected postprocessing. Prototype those boundaries in both Rust and Go before choosing a primary language. Evaluate numerical performance, array and scientific-data libraries, Fortran/C interoperability, operational simplicity, and contributor experience rather than selecting a language on preference alone.

Retain validated legacy kernels through a narrow foreign-function or process boundary while replacements are developed. Port numerical components one at a time only when each replacement can be compared with the baseline across the historical CAD case and smaller deterministic fixtures. A modern release should build reproducibly on supported Linux systems, run in containers and common batch environments, and produce documented diagnostics when output diverges from the reference.

## Key open questions

- Which historical wedge cases best test the behavior forecasters value, and which observed datasets should serve as the reference?
- What domain captures the Appalachian cold-air source and upstream forcing while keeping the 2 km run affordable?
- Can the archived NAM/NMMB code and its dependencies be built outside NOAA's operational HPC environment? Which versions are needed?
- Which fixed datasets can be regenerated for the new grid, and how do we produce consistent initial and lateral boundary conditions after NAM retirement?
- Is a 2 km standalone domain sound, or should a coarser parent and a 2 km nest be run together?
- Should Rust or Go be the primary implementation language, and which scientific libraries and interoperability constraints decide that choice?
- Which legacy components should remain in Fortran initially, and what stable interfaces let them be replaced without coupling the whole system to the original build?
- What numerical tolerances, deterministic fixtures, and historical cases are sufficient to declare a ported component equivalent?
- Which modern targets must be supported first: containerized Linux workstations, cloud machines, or HPC schedulers and accelerators?
- What does the existing code and its components permit us to redistribute? Review source provenance and terms before importing upstream files into this MIT-licensed repository.
- What verification threshold would justify publishing experimental live runs, with clear caveats and provenance?

## Source material

- [NOAA NAM v4.2.13 operational code archive](https://www.nco.ncep.noaa.gov/pmb/codes/nwprod/nam.v4.2.13/): model source under `sorc/nam_nems_nmmb.fd`, plus scripts, parameters, fixed files, and version information.
- [Matthew Pyle's NAM repository](https://github.com/MatthewPyle-NOAA/NAM): GitHub snapshot of NAM source and workflow; inspect its history and provenance before selecting a baseline.
- [NOAA NAM system description](https://www.emc.ncep.noaa.gov/emc/pages/numerical_forecast_systems/nam.php): operational domains, nesting, assimilation, and physics differences.
- [NOAA GSI](https://github.com/NOAA-EMC/GSI): data assimilation component.
- [NOAA UPP](https://github.com/NOAA-EMC/UPP): postprocessing component.
- [BSC MONARCH](https://gitlab.earth.bsc.es/es/monarch): separate NMMB-derived system with regional build examples; useful portability reference, not an interchangeable NAM release.

These sources are references, not copied dependencies of this repository. Their availability, revision, and terms should be checked when implementation begins.
