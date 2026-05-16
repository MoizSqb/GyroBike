# GyroBike
Attitude Control &amp; Gyroscopic Stabilization of a Self-Balancing Bicycle, using an Adaptive PID algorithm.

A self-balancing bicycle robot built as a Final Year Project at FAST-NU, Lahore (Department of Electrical Engineering). The system uses a novel hybrid control architecture combining cascaded PID control with an immune-inspired adaptive gain law and a sliding-mode-inspired disturbance compensator.

---

## The Problem

A two-wheeled bicycle is inherently unstable — gravity constantly tries to tip it over. Unlike a Segway-type robot, a bicycle has additional degrees of freedom (steering and forward speed), making balance significantly harder to achieve and maintain. The challenge is to keep the bike upright in real time, even when external disturbances like a lateral push are applied.

---

## Our Solution

We designed and implemented a **three-loop cascaded control architecture** running at 200 Hz on an STM32 Cortex-M4 microcontroller. The controller combines:

- A **cascaded PID baseline** with three bandwidth-separated loops
- An **immune-inspired adaptive gain law** that modulates controller aggressiveness in real time based on error magnitude
- An **event-triggered sliding-mode-inspired disturbance compensator** that activates only when disturbances exceed a meaningful threshold

---

## How It Works

### The Three Control Loops

The system decomposes the control problem into three nested loops, each operating at a different speed:

| Loop | What it controls | Speed |
|------|-----------------|-------|
| Inner (Gyro) | Flywheel roll rate | Fastest |
| Middle (Attitude) | Roll angle | Medium |
| Outer (Velocity) | Forward/backward drift | Slowest |

Each loop feeds its output as the setpoint for the loop inside it. This cascade ensures that fast dynamics (flywheel rate) are handled independently of slow dynamics (forward velocity), preventing interference between control objectives.

### Why the Bike Stays Upright

At low speeds, the bicycle behaves like an inverted pendulum. The linearised roll dynamics are:

```
Ix * φ'' = mgh * φ − τc + τd
```

Where `φ` is the lean angle, `τc` is the corrective torque from the controller, and `τd` is external disturbance. The controller's job is to keep generating `τc` faster than the bike can fall.

### Adaptive Gain — The Immune Analogy

Standard PID uses fixed gains, which degrade when conditions change. Our controller updates the proportional gain every 5ms:

```
Kp_new = Kp_old + η * e²
```

The larger the lean error, the faster the gain increases — analogous to how the immune system ramps up antibody production when antigen load is high. Saturation bounds prevent the gain from going unstable.

### Disturbance Compensator — Smooth SMC

Classical Sliding Mode Control uses a hard sign() switching function, causing rapid motor oscillation (chattering). We replace it with a hyperbolic tangent (tanh), giving smooth disturbance rejection without mechanical wear. The compensator activates in three tiers based on disturbance severity:

| Roll Error | Response Level |
|-----------|---------------|
| ≤ 8° | Mild (Ksmc = 7.5) |
| 8° – 15° | Moderate (Ksmc = 15.0) |
| > 15° | Full mobilisation (Ksmc = 45.0) |

---

## Hardware

| Component | Role |
|-----------|------|
| STM32 Blue Pill (Cortex-M4, 72 MHz) | Main controller |
| MPU-6050 IMU | Roll angle and roll rate sensing |
| 24V Brushed DC Motor + Encoder | Flywheel actuation |
| N20 DC Gear Motor | Rear wheel propulsion |
| MG995 Servo | Steering control |

---

## Results

Tested under repeated lateral impulse disturbances (~0.8 N·s hand push):

| Metric | Fixed PID | Our Controller | Improvement |
|--------|-----------|---------------|-------------|
| Peak roll deviation | 3.1° | 1.9° | 38.7% |
| Settling time | 510 ms | 295 ms | 42.2% |
| Steady-state error | 0.87° | 0.57° | 34.5% |
| Actuator effort | 612 | 598 | 2.3% less |

The controller runs entirely within a 5ms interrupt cycle on the STM32 — no additional hardware required beyond the base platform.

---

## Tech Stack

- **Microcontroller:** STM32 (ARM Cortex-M4)
- **Sensors:** MPU-6050 IMU, DC encoder
- **Control:** Cascaded PID + Adaptive Immune Law + SMC-inspired compensator
- **Language:** Embedded C
- **Domain:** Control Systems, Embedded Systems, Robotics

---

## Photos

<!-- Add your hardware photos here -->
![Hardware Setup](assets/hardware.jpg)

---

## What I Learned

- How cascaded PID works and why bandwidth separation matters in multi-loop control
- The inverted pendulum model and its application to real unstable physical systems
- How adaptive control laws can outperform fixed-gain controllers under varying conditions
- Sliding Mode Control concepts and why tanh is preferred over sign() in practice
- Real-time embedded systems programming on ARM Cortex-M4
- The gap between theoretical control design and physical hardware implementation

---

## Department
Electrical Engineering, FAST-NU Lahore
