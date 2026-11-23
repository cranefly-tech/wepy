# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
with modifications from [salotz RFC 002](https://github.com/salotz/rfcs/tree/master/rfcs/salotz.002_semantic-changelog)
(Growth Versioning `B.R.G`).
Audience is end users.

History for 1.x releases remains in [`info/changelog.org`](info/changelog.org).

## [Unreleased]

## [2.0.0b0] - 2026-04-17

Beta of wepy 2. This is a breakage version: scripts written against
1.x will need updates. The on-disk HDF5 layout is largely unchanged.

The short-term goals were to make REVO and OpenMM work mapping robust
enough to finish simulations, and to make the core components
testable. Maintenance of core wepy is now focused on molecular
dynamics with OpenMM.

A realistic script-style example is
`tests/integration/test_openmm/test_realistic.py`.

### Improvements

- Python 3.11 through 3.14, NumPy 1 and 2, and h5py 3 are supported.
- Simulation components have explicit typed interfaces. `Manager` is a
  documented state machine over walker state, with `CycleReportDict`
  handed to reporters and `RunSegmentData` produced by runners.
- Runners, resamplers, and work mappers are constructed from
  serializable factories. `Manager` owns the runtime objects, so
  callers no longer need to `deepcopy` those inputs.
- OpenMM walker state is a plain-data `OpenMMState` (all fields are
  `openmm.unit.Quantity`). `OpenMMStateWrapper` wraps a live
  `openmm.State`. States can be built without an `openmm.Context` via
  State XML (`from_dwim`, `from_xml`, `state_to_xml`).
- OpenMM simulations can attach logging reporters. Included:
  `HeartBeatLoggingReporter`, `EnergyLoggingReporter`, and
  `UnitCellLoggingReporter`.
- REVO all-to-all distances use `spawn` worker processes, bounded
  tasks per child, and integrated subprocess logging. This removes
  several deadlock and CUDA-reinit failure modes.
- OpenMM work mapping is purpose-built: `OpenMMSerialWorkMapper` and
  `OpenMMProcPoolWorkMapper` (the production mapper). Device and
  platform settings are part of the mapper configuration. Only
  `spawn` is supported.
- Central `wepy.util.multiprocessing` helpers forward worker logs into
  the parent logging configuration.
- `Walker` is generic over state. `WalkerState` is a protocol.
  `WalkerStateBox` is the dynamically typed black-box state.
- Decision records are real types (`DecisionRecord` /
  `BaseDecisionRecord`) with `to_dict` for reporters.
- Distances and resamplers are generic protocols, with `DistanceABC`
  and `ResamplerABC` for the previous default implementations.
- `wepy.storage` defines shared record-field interfaces for producers
  and reporters, as a step toward storage backends other than HDF5.
- `WepyHDF5Reporter` infers units from initial walker fields when none
  are given. Alternate-representation topologies and sparse trajectory
  fields behave correctly. `h5py.File` handles are closed properly.
- WepyHDF5 dtype serialization follows current NumPy conventions, and
  several h5py 3 string/encoding bugs are fixed.
- `Monitor` interface for cycle metrics (Prometheus extra still
  optional).
- Common symbols are exported from the top-level `wepy` package.
- Test systems under `wepy_tools` include Lennard-Jones pair and
  alanine dipeptide. `MockRunner` is a reference runner for tests.
- Core components have a substantial unit and integration test suite.

### Breakages

- Python older than 3.11 is not supported. h5py 2 is not supported.
- `Manager` takes `runner_factory`, `resampler_factory`, and
  `work_mapper_factory` instead of live runner/resampler/mapper
  instances. Boundary conditions, reporters, and monitors are still
  passed as objects (factories for those are not in this beta).
- Generic `WorkerMapper` and `TaskMapper` are removed. Use
  `OpenMMProcPoolWorkMapper` / `OpenMMSerialWorkMapper` for OpenMM, or
  `SerialMapper` for in-process mapping.
- The old `OpenMMState` (wrapper around `openmm.State`) is now
  `OpenMMStateWrapper`. `OpenMMState` is a pure-data class. Use
  `.dict()` rather than `.to_dict()` on `OpenMMState`.
- OpenMM runner code lives under `wepy.runners.openmm` (`runner`,
  `state`, `logger`, `reporter`).
- `WepyHDF5Reporter` has an explicit constructor. The old
  “pass me the components” convenience is
  `WepyHDF5Reporter.from_components`.
- `Walker` clone/merge methods are standalone functions.
- `Decision.parents` is no longer on the base class; each decision
  type implements it. `NoDecision` moved to
  `wepy.resampling.decisions.no_decision`. `NoResampler` moved to
  `wepy.resampling.resamplers.noresampler`.
- `Resampler` no longer takes `debug_mode`. `REVOResampler` no longer
  accepts `**kwargs`; previously optional constructor arguments are
  required and typed.
- The `orchestration` package, `util/kv.py`, `pytest_wepy`, and the
  random-walk runner/distance modules are removed. The orchestration
  CLI is gone.
- `wepy_tools` test-system APIs changed and are not under the same
  stability guarantee as `wepy`.

### Regressions

- `OpenMMProcPoolWorkMapper` is slower than the old `TaskMapper`. Each
  worker process runs one segment then exits, which is the robustness
  tradeoff (OpenMM context corruption and leaks).
- Logging is much more verbose. Configure loggers if you need quiet
  runs.
- Dashboard reporters were only partially updated for the new
  interfaces.
- Analysis modules (`ContigTree`, parents, networks, profiles) were
  not fully retested; some methods are known incomplete or slated for
  removal.
- Types are annotations only; the project is not type-checked yet.

[Unreleased]: https://github.com/cranefly-tech/wepy/compare/wepy2...HEAD
[2.0.0b0]: https://github.com/cranefly-tech/wepy/compare/c6697f3ac1b82c9c0146da955b7b8f20fe6bdff4...wepy2
