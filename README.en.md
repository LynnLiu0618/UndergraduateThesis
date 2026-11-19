# Study on Vibration Characteristics of Pantograph-Catenary System for High-Speed EMUs

[简体中文](./README.md) · **English**

Undergraduate thesis in Vehicle Engineering · Beijing Jiaotong University · May 2026

How does vibration from an uneven track change the contact force between a pantograph and the overhead wire? This thesis uses SIMPACK to calculate vertical carbody motion, then feeds the displacement and velocity into a Simulink model of a three-mass DSA380 pantograph and a catenary with position-dependent stiffness. It examines how train speed, track quality and pantograph suspension parameters affect current collection.

## Simulation results

At 250 km/h, moving from no track disturbance to high disturbance reduced the mean contact force by only about 2 N, while the minimum fell by about 20 N. The average masks the moments when the pantograph head presses most lightly against the wire. The figures below are from [Table 3-2 of the thesis](./高速动车组弓网系统振动特性研究.pdf#page=31).

| Contact force | No disturbance | Low disturbance | High disturbance |
| --- | ---: | ---: | ---: |
| Mean (N) | 155.86 | 156.82 | 153.78 |
| Standard deviation (N) | 48.62 | 50.56 | 55.85 |
| Minimum (N) | 73.01 | 61.67 | 52.56 |

Chapter 4 also compares six speeds from 225 to 350 km/h. An L9 orthogonal experiment ranked three suspension parameters by their effect on the standard deviation of contact force. Over the ranges tested, lower-frame damping `c3` had the largest effect, followed by head stiffness `k1` and upper-frame stiffness `k2`.

## Repository contents

| File | Contents |
| --- | --- |
| [Thesis (PDF, in Chinese)](./高速动车组弓网系统振动特性研究.pdf) | Chapter 2 covers model development and validation, Chapter 3 examines carbody excitation, and Chapter 4 compares speeds, track conditions and suspension parameters. |
| [PC_model.slx](./PC_model.slx) | Simulink pantograph–catenary model. It takes vertical carbody motion as input and records contact force, head displacement and catenary stiffness. |
| [MATLAB workspace notes](./matlab工作区代码.md) | Three code snippets for initializing parameters, importing SIMPACK output and exporting results to Excel. |

## Run the Simulink model

1. Set the repository folder as the current folder in MATLAB. The model was saved with Simulink R2026a. The zero-input example below was tested with that release.
2. Paste the first block from the [MATLAB workspace notes](./matlab工作区代码.md) into the MATLAB Command Window and run it. The example sets the train speed to `v_kmh = 250` km/h and the static uplift force to `F0 = 70` N.
3. For a first run, set vertical carbody velocity and displacement to zero. These inputs cover the model’s default 10 s simulation.

   ```matlab
   body_vel = [0 0; 10 0];  % Columns: time (s), vertical velocity (m/s)
   body_pos = [0 0; 10 0];  % Columns: time (s), vertical displacement (m)
   out = sim('PC_model');
   ```

4. Inspect `out.contact_force` for contact force (N), `out.x1` for head displacement (m), `out.k_t` for catenary stiffness (N/m) and `out.dis` for distance travelled (m).

**To reproduce a track-excitation case from the thesis**, supply the SIMPACK carbody velocity and displacement histories for that case. The SIMPACK vehicle project and its exported data are not included in this repository.

1. In the second block of the workspace notes, point the two `readmatrix('.txt')` calls to the velocity and displacement files, respectively. Each file uses time in seconds in the first column and velocity in m/s or displacement in m in the second.
2. Run `clear body_vel body_pos`, then execute the second block to load the time histories into the two model inputs. Rerun `out = sim('PC_model')`.

**Exporting to Excel.** The model records contact force, head displacement and stiffness every 1 ms, while `out.tout` and `out.dis` use the solver time points. In the third code block, replace the assignments to `time` and `s` with the two lines below so that each force sample is paired with the distance at the same instant. Here `v` is the model’s constant train speed in m/s.

```matlab
time = (0:numel(out.contact_force)-1)' * 1e-3;
s = v * time;
```
