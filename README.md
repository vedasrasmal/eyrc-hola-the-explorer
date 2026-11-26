# Hola the Explorer — e-Yantra Robotics Competition 2026-27

This repository documents the full Hola the Explorer theme, task by task
(Tasks 0–3C) — from setup through perception, kinematics and control, planning
and teamwork, to the integration and cooperative-carry missions and the video
viva. Task 3A remains the deepest section, because it was the integration task:
find a payload from a single overhead camera frame, plan a collision-free route
around randomly placed pillars, dock precisely on the payload, attach it, carry
it to the centre of a hexagon, release it, and return to the start — against an
8-second target.

- **Competition:** e-Yantra Robotics Competition (eYRC) 2026-27, IIT Bombay
- **Theme:** Theme 2 — "Hola the Explorer"
- **Team:** eYRC #6982 — Team leader: Vedas Rasmal
- **Stage:** Stage 1, Tasks 0–3 (Sep–Nov 2026)
- **Stack:** ROS 2 Humble, Python, OpenCV, A* planning, MuJoCo, Ubuntu 22.04

> **This repository is a write-up only.** It describes the design, the system and the
> measured results. It contains **no competition source code**: the Task 3A planner and
> follower carry the e-Yantra copyright header — the simulation files and source are
> e-Yantra, IIT Bombay intellectual property provided for eYRC 2026-27 and are not
> redistributable.

Full project report (9 pages): [docs/eyrc-hola-explorer-project-report.pdf](docs/eyrc-hola-explorer-project-report.pdf)

## The Task 3A mission

The robot starts at (1.2192, 2.2616) m in a 2.4384 m square arena. A payload crate
(`payload_beta`) sits on a circular pad among pillars placed randomly at each launch.

| Phase | What happens |
| --- | --- |
| Approach | Drive to the payload, align to yaw π (facing left), dock on its right side so the solenoid tip makes contact, and call attach. |
| Carry | Carry the attached crate to the destination (1.2192, 1.2192) m — the centre of the hexagon — holding yaw π throughout, then call the detach service. |
| Return | Retreat clear of the placed crate, then drive back and park at the start pose. |

Scoring rewards speed: the target is 8 seconds, with marks deducted per second over.
Docking also has tight geometric conditions — the solenoid tip must touch the crate,
within about ±20 degrees of the required orientation — so the final approach is a
precision problem, not just a speed problem.

## How the theme built up to it

| Task | Skill | What was built |
| --- | --- | --- |
| Task 0 | Setup | Ubuntu 22.04, ROS 2 Humble and MuJoCo simulation environment. |
| Task 1A | Perception | Checkpoint detection from one overhead camera frame with OpenCV colour masking and contours. |
| Task 1B | Kinematics | Inverse kinematics for the 3-wheel holonomic drive: body velocity to wheel speeds. |
| Task 1C | Control | PID pose control following waypoints and shapes without corner-cutting or overshoot. |
| Task 2A–2C | Planning and teamwork | Occupancy-grid A* planning, planner / follower split, multi-robot coordination through a single gap. |
| Task 3A | Integration | Camera-planned docking, attach / carry / detach and return under a time target (this report). |
| Task 3B | Cooperative carry | Three robots carrying one payload as a rigid body, each docked on its own side with matched offsets and headings. |
| Task 3C | Video viva | Recorded viva on the portal covering Tasks 1–3; submitted, awaiting grading when checked. |

## The whole theme, task by task

This repo covers the full theme, task by task, and Task 3A remains the deepest
section because it was the integration task. The sections below give the detailed
account of Task 3A; this section completes the picture for the rest of the theme,
using only verified results. Where the verified record for a task is brief, that is
stated plainly rather than filled in.

### Task 0 — Setup | 100 / 100

Task 0 set up the simulation environment used for the theme: Ubuntu 22.04, ROS 2
Humble and MuJoCo. It scored 100/100. Detail beyond the environment and the score
is not part of the verified record summarised here, so none is added.

### Task 1A — Perception

Task 1 as a whole scored 99/100 (subtasks 30/30, 20/20, 49/50). Task 1A covered
perception: checkpoint detection from one overhead camera frame, using OpenCV
colour masking and contours. No further verified detail for Task 1A is recorded
here beyond the method above and the Task 1 scores.

### Task 1B — Kinematics

Task 1B covered inverse kinematics for the 3-wheel holonomic drive, converting
body velocity to wheel speeds. This is the same conversion the Task 3A follower
relies on. No further verified detail for Task 1B is recorded here beyond the
description above.

### Task 1C — Control

Task 1C covered PID pose control, following waypoints and shapes without
corner-cutting or overshoot. No further verified detail for Task 1C is recorded
here beyond the description above.

### Tasks 2A–2C — Planning and teamwork | 100 / 100

Tasks 2A–2C together scored 100/100 (subtasks 20/20, 30/30, 50/50). The verified
work for this stage covers three elements: occupancy-grid A* planning; the
planner / follower split — the separation between planning a path and following
it, which the Task 3A system also uses; and multi-robot coordination through a
single gap. Detail beyond these three elements and the scores above is not part
of the verified record summarised here, so none is added.

### Task 3A — Integration | Best 27.37 / 30

Task 3A was the integration task: camera-planned docking, attach, carry and
detach, and return, under the 8-second target. It is covered in depth in the
sections below. Best portal score: 27.37/30, from the submission on 16 Nov,
11:21 pm; earlier submissions were 23.03, 23.02 and 22.19. The portal marks
the best submission "Late, not eligible", so the counted value after the late
penalty may differ. Best verified run: 9.54 s. A later
run of 7.88 s was reported by the team and is marked as reported, not verified,
because its files and telemetry were not available for independent checking.

### Task 3B — Cooperative carry | Best 46.68 / 50

Task 3B extended the work from one robot to three: three robots carrying one
payload as a single rigid body, each robot docked on its own side, with matched
offsets and headings. Best portal score: 46.68/50. No further verified detail
for Task 3B — such as timings or run-by-run results — is recorded here, so none
is added.

### Task 3C — Video viva

Task 3C was a recorded viva on the portal, covering Tasks 1–3. It was submitted
on 17 November 2026 and was awaiting grading when last checked. No score for
Task 3C is recorded here, because grading was still awaited at the time of the
last check.

### The theme as a whole

The progression is from setup, through perception, kinematics and control, to
planning and teamwork, and then to integration and cooperative carry, with the
viva covering Tasks 1–3. Task 3A remains the most fully documented task in this
repo because it brought those earlier skills together in one timed mission.

## Architecture: one planner, one follower

The delivered solution is a two-node ROS 2 system. A camera-based path planner
detects the payload and obstacles from one fresh frame, builds an occupancy grid, and
produces timed paths with A* search. A path follower tracks those paths at 50 Hz using
velocity feedforward, converts body velocity to wheel speeds through inverse
kinematics, and calls the attach and detach services at the docking and drop-off
points. No crate coordinates or routes are hardcoded, and no grader-only state is
read — every launch plans from the camera, because the pillar layout changes every
time.

Interfaces: `/r2/plan` (approach), `/carry_plan` (crate centres to destination),
`/r2/return_plan` (return), `/r2/odom` feedback, and the attach / detach services
between phases.

### Perception and path planning

The planner fetches one 1280 × 720 JPEG frame from the camera stream and crops the
671 × 671 px arena region. The payload is found by its teal top: colour segmentation
and contour analysis locate the crate, its circular pad and every pillar obstacle.
Height correction is calibrated from the pad, and the corrected frame is converted to
a 488 × 488 occupancy grid (about 5 mm per cell) over the 2.4384 m arena.

Planning uses the right footprint for each phase. Approach obstacles are inflated by a
conservative robot disc (84 mm circumscribed radius, which already includes the 70 mm
front solenoid at every yaw), plus a 20 mm obstacle margin. Dock and undock are
special: they are straight axial contact segments, so the target crate is checked
analytically against the 70 mm front tip instead of the full robot disc — otherwise
the planner would correctly, but uselessly, declare intentional contact impossible.
Carry uses the union of the rectangular crate and the offset robot disc, because the
robot drives beside the crate it is carrying. The return plan also treats the newly
placed crate as an obstacle.

Routes are found with A* search, simplified to waypoints, then smoothed and
time-parameterised with speed, acceleration and lateral-acceleration limits, plus
slower dock-entry and retreat-exit speeds.

Key planning parameters (v12.0):

| Parameter | Approach / return | Carry |
| --- | --- | --- |
| Plan speed | 1.04 m/s | 0.90 m/s |
| Acceleration / deceleration | 2.40 / 2.25 m/s² | 2.10 / 2.00 m/s² |
| Lateral acceleration limit | 2.10 m/s² | 1.58 m/s² |
| Dock entry speed | 0.340 m/s | — |
| Retreat exit speed | 0.440 m/s | — |
| Docking slot offset | (+0.1798, 0) m from the destination; dock yaw held at π | — |

Earlier verified runs used the slightly cooler v10.9 tuning (cruise at or below
1.0 m/s). Version differences and test outcomes are reported honestly in the results
below — hotter tuning was trialled, not assumed better.

### Trajectory following and docking control

The robot is a 3-wheel holonomic drive (wheels at 30°, 150° and 270°; wheel radius
25.5 mm; chassis radius 64.12 mm). An inverse-kinematics matrix converts a commanded
body twist (vx, vy, yaw rate) into the three wheel speeds. Omni wheels are what make
the docking strategy possible: the robot can translate sideways without turning, so it
can hold yaw π — facing left — while sliding into the dock on the payload's right side.

- **Saturation handling:** wheel commands are limited to 30 rad/s. When a command
  would saturate, yaw authority is preserved first and translation is scaled down
  uniformly, so the robot slows rather than spinning off heading.
- **Yaw wrap-around:** the error is always the short way round at ±π, which matters
  because the dock yaw sits exactly on the wrap boundary.
- **Tracking:** each plan is converted to a trajectory whose pose timestamps give
  relative time. The follower tracks it with velocity feedforward plus pose feedback,
  using an adaptive path clock that can slow time rather than lose the path. During
  the empty approach the robot rotates towards yaw π while translating, instead of
  stopping to turn.
- **Settling:** the follower brakes to a controlled stop only where precision
  matters — at docking, at drop-off and at final parking. Everywhere else it keeps
  tracking through.
- **The docking gate:** before entering the final contact corridor, the follower
  requires yaw π to be genuinely established — not just touched once. The gate checks
  yaw error, spin speed and lateral offset, and holds at the pre-dock point until they
  are satisfied.
- **Attach and detach:** payload pose during carry is inferred from the detected
  initial centre and the rigid weld — the follower never subscribes to grader-only
  arena or payload topics. Attach and detach service calls are asynchronous: a
  blocking call inside the control callback would stall the 50 Hz loop at the exact
  moment stability matters most.
- Every run logs per-phase diagnostic lines and an automatic telemetry CSV (pose, yaw
  error, spin speed, mode), which is what made the evidence-based debugging below
  possible.

## Results

### Verified Task 3A runs (v10.9, team's Humble / Ubuntu 22.04 machine)

| Run | Approach | Carry | Return | Total | Outcome |
| --- | --- | --- | --- | --- | --- |
| Run 1 | 3.40 s | 3.08 s | 3.78 s | 10.26 s | Complete, attach / detach OK |
| Run 2 | 3.18 s | 2.83 s | 3.51 s | **9.54 s** | Complete, attach / detach OK |

Both runs completed the full mission. Pre-dock hold time was 0.00 s in all six
phases, with zero hold rows in the telemetry CSV — the docking gate passed first try
(yaw error 0.038 rad at 50 mm out in the analysed run).

**Where the extra time sits:** splitting the best run by distance showed the problem
is not cruising speed. About 5.44 s is spent reaching the final 100 mm of each leg,
and about 4.08 s in the three final-100 mm tails (1.10–1.58 s per leg, against a
0.50–0.70 s target). Telemetry showed the carry and return legs switching to direct
terminal mode at 68 mm and nearly stalling around 50 mm, while the approach leg —
which stays in tracking mode longer, entering terminal mode at 50 mm — passes the same
zone at 0.39 m/s. That single difference explains most of the gap to the 8-second
target.

### Later tuning trial (v12.0, VM-tested)

v12.0 raised plan speed, acceleration and lateral limits above v10.9 and added new
terminal-mode logic. It was tested in a ROS 2 Humble chroot environment on a throttled
VM: 4 of 4 missions completed, with zero aborts, zero holds, zero attach refusals and
zero planner warnings. However, overshoot past the goal deepened compared with v10.9
(true minimum along-track distance past the goal reached −5.0 mm or beyond on 6 of 12
legs, worst −8.9 mm, with the return leg overshooting in all 4 runs).

Verdict recorded at the time: safe to trial on the team's machine under a strict rule
(median under 9.54 s, no abort / hold / refusal, overshoot not routinely past −5.4 mm)
— **not pre-approved for submission**. VM timings (15–17 s) are throttled and are not
comparable to the team's machine.

### Portal scores (Stage 1, checked 19 Nov 2026)

| Task | Score | Detail |
| --- | --- | --- |
| Task 0 | 100 / 100 | Setup and theme selection |
| Task 1 | 99 / 100 | 1A 30/30, 1B 20/20, 1C 49/50 |
| Task 2 | 100 / 100 | 2A 20/20, 2B 30/30, 2C 50/50 |
| Task 3A | 27.37 / 30 (best) | Single-robot docking mission |
| Task 3B | 46.68 / 50 (best) | Three-robot cooperative carry |
| Task 3C | Submitted | Video viva submitted 17 Nov 2026, awaiting grading when checked |
| Task 3 total | 74.05 / 100 | 27.37 + 46.68; Task 3C not yet graded |

**Reported, not verified:** the team later reported a 7.88 s Task 3A run (believed to
be a v12.2 revision). Its files, diagnostic logs and CSV were not available when the
report was written, so **9.54 s remains the best verified time** used throughout this
repo.

## Problems found, by evidence — not by guessing

- **The pre-dock latch (latent risk):** code analysis found that the docking gate
  tightens after a first failed entry — yaw tolerance narrows from 0.075 rad to
  0.048 rad and extra spin-speed and lateral conditions are added, without relaxing
  again during that approach. A robot arriving with yaw error between 0.048 and
  0.075 rad could therefore hold until timeout. The verified runs prove it did not
  trigger there (0.00 s hold), so it is recorded as a latent risk with a known
  signature in the CSV, not as the cause of the slow runs.
- **The terminal-mode stall (measured cause):** the slow tails were traced to carry
  and return entering direct terminal mode at 68 mm, versus 50 mm for approach —
  found by comparing mode, speed and position columns in the run CSV, not by raising
  speeds blindly.
- **Actuator limits:** approach wheel speeds sat at the 30 rad/s limit for 21+
  control ticks mid-leg while rotating and translating, and braking produced 6–8 mm
  of recoil with zero command. The lesson: faster commands can make the robot
  slower; cruise speed was deliberately not pushed past the verified envelope
  without evidence.
- **Environment discipline:** the Humble simulation binaries do not run under
  ROS 2 Jazzy (missing library symbols), so all timing claims come from the team's
  Humble / Ubuntu 22.04 machine. A separate Humble chroot environment was used only
  for completion and safety checks, never for timing.

## Skills demonstrated

- ROS 2 node design: topics, latched QoS, services, launch and timing contracts
  between nodes
- Computer vision with OpenCV: colour segmentation, contours, pixel-to-world
  calibration
- Grid path planning: occupancy grids, footprint inflation, A* search, path
  smoothing and time parameterisation
- Holonomic kinematics and control: inverse kinematics, feedforward trajectory
  tracking, saturation-aware wheel allocation, angle wrap handling
- Evidence-based debugging: per-phase diagnostics, telemetry CSV analysis,
  median-of-runs comparison across random layouts

## Limitations

- All results are in simulation (MuJoCo / e-Yantra arena); no physical-robot results
  are claimed.
- The best verified time, 9.54 s, is above the 8-second full-marks target; this repo
  identifies where the remaining time sits rather than claiming the target was met on
  verified evidence.
- The 7.88 s team-reported run and any v12.2 revision are unverified here because
  their artefacts were not available.
- Task 3C viva grading was still awaited when the portal was last checked.

## Repository layout

```
eyrc-hola-the-explorer/
├── README.md
└── docs/
    └── eyrc-hola-explorer-project-report.pdf   # Full 9-page project report
```

Deliverables per submission: the planner and follower Python files, ROS bag
recordings, and an unlisted run video, submitted through the e-Yantra portal by the
team leader (best attempt counts).

## License

MIT — see [LICENSE](LICENSE). The write-up and results in this repository are the
author's own; the e-Yantra simulation and competition source are e-Yantra, IIT Bombay
intellectual property and are not included here.

## Contributors

- Vedas Rasmal — https://github.com/vedasrasmal
- Vedant Vijay Gadekar — https://github.com/vedant04-08
