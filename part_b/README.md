## Prediction, before training the wide-range policy (Oct 16, 1:31am)
I widened `lin_vel_x` and `lin_vel_y`, which are forward/backward and sideways inputs, respectively, from (-1, 1) to (-2, 2) m/s. `ang_vel_z` is left unchanged.

**Physical feasibility:** According to Unitree's specs ([unitree.com/go2](https://www.unitree.com/go2/)), the Go2's top forward speed is 2.5-3.7 m/s depending on the model, so +-2 m/s forward is physically plausible. However, I could not verify if the sideways speed can reach +-2 m/s as well. Although possible, I expect the robot to undershoot fast `lin_vel_y` commands because moving sideways for a quadruped relies on hip joints with a smaller range of motion.

**What I expect:**
1. `error_vel_xy` will increase.
2. `success_rate` will decrease.
3. `base_contact` will remain around the same.

**Why:**
The policy's reward weights and tracking tolerance were chosen for the default +-1 range. Given that the tracking reward is `exp(-error^2 / 0.25)`, when the robot is far from a fast command, the reward is almost 0, meaning that the policy receives almost no feedback about how to go faster. At the same time, the penalties give a clear feedback regardless, which I expect to result in the policy preferring balance over speed (hence, I expect the `base_contact` to remain around the default's value). I expect the `success_rate` to drop because the threshold (0.5 m/s) stays fixed while the range of commands becomes twice as large -> stricter tolerance

I expect the 3x iteration time to help improve the `error_vel_xy` and `success_rate` for wider-range policy, but I'm skeptical about it performing the same or better than the default unless the physical-plausibility of the sideways motion of (-2, 2) is proven and the reward weights and tracking tolerance are adjusted.

## Results (wide-range vs default)
![TensorBoard comparison of default and wide-range runs](tensorboard_default_vs_wide.png)

| Metric (iteration 299) | Default (±1 m/s) | Wide (±2 m/s) |
|---|---|---|
| `error_vel_xy` (m/s) | 0.107 | 1.539 |
| `success_rate` | 1.0 | 0.0 |
| `base_contact` | 0.0061 | 0.0068 |
| `error_vel_yaw` (rad/s) | 0.188 | 0.170 |
| `track_lin_vel_xy_exp` reward | 1.408 | 0.092 |
| `track_ang_vel_z_exp` reward | 0.625 | 0.643 |

Values are the raw (unsmoothed) values at the final iteration.

## Comparison against my predictions
As expected, `error_vel_xy` increased, `success_rate` decreased, and `base_contact` remained about the same as default.

However, I expected the policy to undershoot fast commands, but what actually ended up happening is that the policy did not learn to move at all (the video shows robots not moving even for slow commands, which the default policy tracked well). This is shown by `success_rate` staying at 0 for all iterations, and by the `error_vel_xy` curve staying flat at ~1.5 from the first to last iteration. The `track_lin_vel_xy_exp` also stayed flat at ~0.1, while the default's rose to 1.4, for the whole run, which supports my prediction that the policy receives almost no feedback about how to go faster.

Not covered in my prediction: `error_vel_yaw` and `track_ang_vel_z_exp` performed as well as the default, so the policy learned to turn.

Overall, the policy ended up preferring balance over speed, and Go2 stood and turned in place.

Also, I had attributed the undershooting to the possible physical limitations of the Go2's sideways motion, but the policy failed at slow commands as well, so I think that the weak reward signal is a better explanation for this performance.

The video confirms that the robots do not move: the blue arrows (velocity) stay near zero while the green arrows (commands) are long.
