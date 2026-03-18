# cattle_climate_simulator_env

`cattle_climate_simulator_env` is a lightweight C++ cattle-building climate simulator package that preserves the original mechanical simulation core of [cattle_climate_simulator](https://github.com/nn44nniri/cattle_climate_simulator)
 and adds a new **reinforcement-learning-oriented environment layer** for stepwise control. It is close to a Gym-style RL environment core, but it is not yet a complete Gym library by itself in the usual sense. It is a C++ Gym-like environment wrapper around the cattle climate simulator, designed so a reinforcement learning agent can interact with the cattle hall climate through reset/step style transitions.
The package now supports two complementary modes:

1. **Original simulation mode** for deterministic scenario execution over a full time horizon.
2. **RL environment mode** for closed-loop control experiments in which an agent adjusts environmental stimuli at each control step to keep the cattle-hall vital climate variables inside configurable optimal ranges.

The goal of this update is to prepare the simulator for future Gym-style integration **without disturbing the original core equations, transition formulas, or configuration flow**.

---

## 1. Project goal

The original `cattle_climate_simulator` models the indoor climate of a cattle building using a compact mechanical formalism. It already computes the hall variables needed for control-oriented studies, such as indoor temperature, humidity, airflow, and actuator effects.

This update extends that simulator so it can be used as a **control environment**.

The practical objective is to support experiments where an intelligent controller or reinforcement learning agent uses environmental stimuli such as:

- dampers
- fans
- heaters

in order to keep hall vital conditions within target bands, especially:

- indoor temperature
- indoor relative humidity
- airflow

This makes the package suitable for future work on policy learning, control benchmarking, random-action rollouts, reward shaping.

---

## 2. Design approach

The update follows a low-risk design:

- the original climate simulator core is preserved
- the original mechanical formulas are reused as-is
- the existing batch simulator remains available
- the new RL layer is added **on top of** the original simulator rather than replacing it

This means the package was not rewritten into a new physics model. Instead, the simulator was extended with a stateful interface that supports the control pattern required by RL frameworks:

- `reset()`
- `step(action)`
- observation
- reward
- done flags
- rollout history

This approach keeps the package scientifically stable while making it easier to connect to Gym-style later.

---

## 3. What was added to the original library

The following additions were introduced on top of the original `cattle_climate_simulator` package.

### New RL environment layer

A new stateful environment API was added:

- `include/cattle_climate/env.hpp`
- `src/env.cpp`

Main class:

- `cattle_climate::CattleVitalEnv`

New RL-oriented data structures:

- `RLAction`
- `RLObservation`
- `VitalStatus`
- `RLStepInfo`
- `RLStepResult`
- `RLEnvironmentOptions`

### New random test executable

A configurable random-action test was added:

- `tests/random_env_test.cpp`

This test can now:

- set environment input parameters from the command line
- run one or more random-action episodes
- export rollout data to CSV
- export the last episode as an SVG graph

### New rollout reporting helpers

The RL layer also includes report-writing helpers for the episode history:

- `write_rl_csv_report(...)`
- `write_rl_svg_graph_report(...)`

These additions make the package usable for fast RL-style experimentation and for visual inspection of the environment response after testing.

---

## 4. What was intentionally preserved

The following parts of the original package remain the reference computational core:

- formula implementation files
- simulation settings flow
- single-zone simulator logic
- original CLI scenario execution path

In particular, this update does **not** replace or redesign the original mechanical climate formalisms. The RL layer only wraps and reuses the simulator outputs to create observations, reward values, and stepwise interaction.

This separation is important because it allows future control research without changing the baseline simulator equations.

---

## 5. RL environment concept

The RL environment in this package should currently be understood as a:

**cattle-hall climate control environment**

and not yet as a full direct cattle physiology simulator.

At this stage, the environment optimizes hall climate variables that are important for animal comfort and operational control.

### Action space

The action is continuous and actuator-based:

- `damper_percent` in `[0, 100]`
- `fan_percent` in `[0, 100]`
- `heater_percent` in `[0, 100]`

These percentages are converted internally into actuator commands for one control interval.

### Observation space

A typical observation includes:

- indoor temperature
- indoor relative humidity
- indoor humidity ratio
- current airflow
- average airflow
- outdoor temperature
- outdoor relative humidity
- outdoor wind speed
- outdoor solar radiation
- actuator powers
- in-range / out-of-range status indicators

### Reward logic

The reward combines:

- penalties for temperature outside the target band
- penalties for humidity outside the target band
- penalties for airflow outside the target band
- penalties for actuator energy use
- penalties for abrupt control changes
- bonus for maintaining the target climate envelope

These target bands and weights are configurable through the environment options or test command-line parameters.

---

## 6. Why this update is useful

This update prepares the package for several future workflows:

- policy-gradient or actor-critic control experiments
- MPC vs RL comparison studies
- reward design experiments
- stochastic rollout analysis
- stress-testing of environmental control limits
- later coupling with cattle thermal-comfort or physiological models

Because the simulator remains in C++, it can also serve as a fast backend for repeated control rollouts.

---

## 7. Project structure

Typical important files in the package are:

- `include/cattle_climate/env.hpp`
- `src/env.cpp`
- `src/formulas.cpp`
- `src/settings.cpp`
- `src/simulator.cpp`
- `src/main.cpp`
- `tests/random_env_test.cpp`
- `config/settings.json`
- `README.md`

Build artifacts may also appear in `build/` after compilation.

---

## 8. Build instructions

From the project root:

```bash
cmake -S . -B build
cmake --build build
```

This should produce at least:

- `build/cattle_climate_example`
- `build/cattle_climate_random_env_test`

---

## 9. Running the original simulator

The original batch simulator remains available.

Example:

```bash
./build/cattle_climate_example \
  --settings config/settings.json \
  --duration-minutes 300 \
  --outside-temp-c 35 \
  --dampers off \
  --fans on \
  --fan-driver 42 \
  --fans-active-minutes 300 \
  --heaters off \
  --csv build/fan_only_hot_300m.csv \
  --graph build/vital_params_0_300m.svg
```

This mode is appropriate for deterministic scenario analysis over a full horizon.

---

## 10. Running the RL random-action test

A default test run:

```bash
./build/cattle_climate_random_env_test --settings config/settings.json
```

A parameterized run with explicit environment settings, CSV export, and graph export:

```bash
./build/cattle_climate_random_env_test \
  --settings config/settings.json \
  --episodes 2 \
  --steps 30 \
  --seed 77 \
  --outside-temp-c 32 \
  --outside-rh 0.72 \
  --outside-wind-speed-m-s 4.5 \
  --initial-indoor-temp-c 18 \
  --initial-indoor-rh 0.58 \
  --temp-min 10 --temp-max 22 \
  --rh-min 0.45 --rh-max 0.80 \
  --airflow-min 0.05 --airflow-max 2.0 \
  --damper-max 60 --fan-max 85 --heater-max 30 \
  --csv build/rl_rollout.csv \
  --graph build/rl_rollout.svg
```

Example text output:

```text
random_env_test=PASS
episodes=...
steps_per_episode=...
history_size_last_episode=...
last_episode_reward=...
total_reward=...
```

---

## 11. RL test outputs

The random test can produce two useful artifacts.

### CSV rollout file

The CSV file stores one row per control step for the last episode.

Typical use:

- numerical inspection
- external plotting
- offline analysis
- validation of reward behavior

### SVG graph

The SVG graph visualizes the last rollout and typically plots:

- indoor temperature
- indoor relative humidity
- airflow
- reward

This makes it easier to inspect how the randomized actuator sequence affects the hall climate through time.

---

## 12. Running tests with CTest

After building:

```bash
cd build
ctest --output-on-failure
```

This is the recommended way to validate that the package still builds and that the random RL environment test runs successfully.

---

## 13. Minimal C++ usage example

```cpp
#include "cattle_climate/env.hpp"
#include "cattle_climate/settings.hpp"

using namespace cattle_climate;

SimulationSettings settings = load_settings_from_json("config/settings.json");
RLEnvironmentOptions options;
options.max_episode_steps = 24;

CattleVitalEnv env(settings, options);
RLObservation obs = env.reset();

RLAction action;
action.damper_percent = 15.0;
action.fan_percent = 35.0;
action.heater_percent = 0.0;

RLStepResult result = env.step(action);
```

---

## 14. Future direction

This package is now structurally ready for a Gym-style wrapper because the C++ environment exposes the correct control pattern.

A future binding layer can be added with tools such as:

- `pybind11`
- C API wrappers
- other native bindings

Later extensions may also add:

- THI-based rewards
- direct cattle thermal-comfort penalties
- thermoregulation indices
- behavior-oriented reward terms
- stochastic weather disturbances
- multi-zone or multi-pen control

---

## 15. Notes and current limits

- The current RL environment optimizes **hall climate vital variables**, not direct cattle body-state variables.
- It should therefore be used as a climate-control testbed for cattle-building conditions.
- The original simulator core has been preserved intentionally for low-risk extension.
- The RL layer is designed as an add-on and not as a replacement for the original simulator.

---

## 16. Summary

This updated package extends `cattle_climate_simulator` into a reinforcement-learning-ready C++ environment while preserving the original simulation core.

In practical terms, the package now provides:

- the original cattle climate simulator
- a stateful RL-style environment interface
- configurable random testing
- CSV rollout export
- SVG graph export
- a clean base for future Gym integration

This makes it suitable for the next development stage: mounting a Gym-style layer or training a controller directly against the preserved C++ simulator backend.
