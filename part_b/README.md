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
