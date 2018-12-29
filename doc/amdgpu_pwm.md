# AMDGPU PWM fan speed control algorithm

## Inputs / Outputs
1. fan1_input - > Fan (RPM)
2. in0_input -> ASIC Voltage (mV)
3. power1_average -> ASIC power (mW)
4. power1_cap -> ASIC powerlimit currently configured (mW)
5. power1_cap_max -> ASIC powerlimit hardware max (mw)
6. power1_cap_min -> ASIC powerlimit minimum (mW)  --- Why does this exist?
7. pwm1 -> Current fan pwm value
8. pwm1_enable - > PWM state (0, 1, 2)
9. pwm1_max -> PWM Max value (255) (u_8int)
10. temp1_crit -> Hardware max temperature
11. temp1_crit_hyst -> Hysterisis value (broken in 4.19.12 large negative value)
12. temp1_input -> Current ASIC temperature

## Critical values
* fan1_input - Use this to approximate noise value
* in0_input - Use this to determine heating coefficient of ASIC
* power1_average - average power dissipation (Use this to calculate vector vs current temperature to pre-emptively spin up fans ahead of thermal load)
    * Thermal curve lags behind power curve
* power1_cap - useful indicator of how hot the card can potentially get (use together with in0_input (voltage))


* pwm1_enable - we write 1 here to enable manual fan speed control
* pwm1 - We write here to set the current fan speed

* temp1_crit - use this as the do-not exceed temperature
    * if we reach this temperature 
        1. check power1_average and voltage(in0_input) to determine if load is present
        2. calculate temperature vector (is temperature increasing, decreasing or stable)
        3. check that fan rpm is > 0
        4. check that fan rpm is experiencing microfluctuations (useful indicator if fans are still spinning)
            * If we fail these checks alert the system operator / user - report the specific GPU and it's state
            * If we determine that fans are at 0 or not fluctating at high temperature - report a possible fan failure
* temp1_input
    * Average out short burst loads using power1_average to prevent unncessary fan speed changes for temporary temperature peaks that will dissipate easily
    * Use this temperature to determine thermal change vectors (decreasing/increasing)
        * Increase fan speeds proportional to vector
        * Increase fan speeds using smoothed curve (non-linear) so it doesn't sound so annoying.

## Control variables

* temperature_target (C)
* noise range (rpm)
    Array calibrated by user to determine which fan speeds are quiet and from what speed it gets progressively louder
* bias target (noise, temperature, both) - in future perhaps power also
* temperature_leeway (precision of temperature target in C) 

## Calculated variables

* power Vector
* temperature Vector
* voltage state (increasing/decreasing/stable)
* fan_pwm
* fan_pwm_target
