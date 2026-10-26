# FYP-Block-Diagram
Block Diagram Description: Grid-Supporting Inverter with Pre-Synchronization and P-f/Q-V Droop Control
Below is a structured block diagram description you can directly translate into Visio, draw.io, PowerPoint, or Simulink. I've organized it hierarchically so each subsystem is clearly delineated.

1. High-Level System Block Diagram
text
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    POWER STAGE                                              │
│                                                                                             │
│  ┌──────────┐    ┌──────────────┐    ┌────────────┐    ┌───────────┐    ┌───────────────┐  │
│  │   DC     │    │  3-Phase     │    │    LCL     │    │    PCC    │    │   Area EPS    │  │
│  │ Source   │───▶│  VSI         │───▶│  Filter    │───▶│  Breaker  │───▶│   (Grid)      │  │
│  │ (PV/BESS)│    │ (Inverter)   │    │            │    │           │    │               │  │
│  └──────────┘    └──────┬───────┘    └─────┬──────┘    └─────┬─────┘    └───────────────┘  │
│                         │                  │                 │                              │
│                    Gate Signals       V_pcc, I_inv     Breaker Status                      │
│                         │                  │                 │                              │
└─────────────────────────│──────────────────│─────────────────│──────────────────────────────┘
                          │                  │                 │
                          │                  ▼                 │
                          │    ┌─────────────────────────┐     │
                          │    │  MEASUREMENT & PLL      │     │
                          │    │                         │     │
                          │    │  V_pcc(abc) ──▶ [PLL] ──┼──▶ θ_g, ω_g, V_g_mag
                          │    │                         │     │
                          │    │  V_pcc(abc) ──┐         │     │
                          │    │  I_inv(abc) ──┴▶ [P/Q   │     │
                          │    │                 Calc] ──┼──▶ P, Q
                          │    └────────────┬────────────┘     │
                          │                 │                  │
                          │                 ▼                  ▼
                          │    ┌──────────────────────────────────────────┐
                          │    │           MODE SELECTOR                  │
                          │    │                                          │
                          │    │  ┌────────────────┐  ┌────────────────┐  │
                          │    │  │  PRE-SYNC      │  │  DROOP         │  │
                          │    │  │  CONTROLLER    │  │  CONTROLLER    │  │
                          │    │  │                │  │                │  │
                          │    │  │ Inputs:        │  │ Inputs:        │  │
                          │    │  │  V_inv_mag     │  │  P, Q          │  │
                          │    │  │  θ_inv, ω_inv  │  │  f_g, V_g      │  │
                          │    │  │  V_g_mag (PLL) │  │                │  │
                          │    │  │  θ_g, ω_g (PLL)│  │ P-f Droop:     │  │
                          │    │  │                │  │  f_ref = f_0   │  │
                          │    │  │ PI Controllers │  │   − m_p(P−P_0) │  │
                          │    │  │  ΔV → V_ref    │  │                │  │
                          │    │  │  Δf → f_ref    │  │ Q-V Droop:     │  │
                          │    │  │  Δθ → θ_ref    │  │  V_ref = V_0   │  │
                          │    │  │                │  │   − n_q(Q−Q_0) │  │
                          │    │  └───────┬────────┘  └───────┬────────┘  │
                          │    │          │                   │           │
                          │    │          └─────────┬─────────┘           │
                          │    │                    ▼                     │
                          │    │            ┌──────────────┐              │
                          │    │            │   SWITCH     │◀── Breaker   │
                          │    │            │  (SPDT)      │    Status    │
                          │    │            └──────┬───────┘              │
                          │    └───────────────────┼──────────────────────┘
                          │                        │
                          │              V_ref, f_ref, θ_ref
                          │                        │
                          │                        ▼
                          │    ┌──────────────────────────────────────┐
                          │    │       INNER CONTROL LOOPS            │
                          │    │                                      │
                          │    │  θ_ref ──▶ [Integrator] ──▶ θ        │
                          │    │                                      │
                          │    │  V_ref ──▶ [Voltage PI] ──▶ I_dq_ref │
                          │    │                                      │
                          │    │  I_dq_ref ──▶ [Current PI] ──▶ V_dq* │
                          │    │                                      │
                          │    │  V_dq* ──▶ [dq/abc] ──▶ [PWM]       │
                          │    │                                      │
                          │    └──────────────────┬───────────────────┘
                          │                       │
                          └───────────────────────┘
                                    Gate Signals to VSI
2. Detailed Block-by-Block Description
Block 1: Power Stage
Component	Description
DC Source	PV array, battery, or DC bus
3-Phase VSI	Voltage source inverter (2-level or 3-level)
LCL Filter	Attenuates switching harmonics
PCC Breaker	Point of common coupling circuit breaker
Area EPS	Local electric power system (grid)
Outputs to control: V_pcc(abc), I_inv(abc), Breaker Status (0/1)

Block 2: Measurement & PLL
Sub-block	Input	Output	Purpose
PLL	V_pcc(abc)	θ_g, ω_g, V_g_mag	Extract grid phase, frequency, magnitude
abc/dq Transform	V_pcc, I_inv	V_dq, I_dq	Park transformation
P/Q Calculator	V_dq, I_dq	P, Q	Instantaneous power (filtered)
Block 3: Pre-Synchronization Controller (Active when Breaker = OPEN)
Element	Signal	Description
ΔV PI	V_g_mag − V_inv_mag → V_ref_pre	Voltage magnitude matching
Δf PI	ω_g − ω_inv → f_ref_pre	Frequency matching
Δθ PI	θ_g − θ_inv → θ_ref_pre	Phase angle matching
Enable	Breaker Status = 0	Active before PCC closes
Purpose: Align inverter voltage vector with grid voltage vector so that closing the PCC breaker produces zero transient.

Block 4: Droop Controller (Active when Breaker = CLOSED)
Element	Equation	Description
P-f Droop	f_ref = f_0 − m_p · (P − P_0)	Frequency decreases as P increases (grid-supporting)
Q-V Droop	V_ref = V_0 − n_q · (Q − Q_0)	Voltage decreases as Q increases (grid-supporting)
Inputs	P, Q (measured), f_g, V_g (from PLL)	—
Outputs	f_ref_droop, V_ref_droop	—
Enable	Breaker Status = 1	Active after PCC closes
IEEE 1547-2018 Clause 5 compliance: The inverter responds to grid frequency and voltage deviations by adjusting P and Q according to the droop characteristics, providing grid support (frequency regulation, voltage regulation).

Block 5: Mode Selector (SPDT Switch)
State	Breaker Status	Selected Reference
Pre-Sync Mode	0 (Open)	V_ref_pre, f_ref_pre, θ_ref_pre
Grid-Connected Mode	1 (Closed)	V_ref_droop, f_ref_droop, θ_ref = ∫2πf_ref dt
Seamless transition: The switch changes state upon breaker closure. Because pre-sync has already aligned V, f, and θ with the grid, the transition produces no transient.

Block 6: Inner Control Loops
Loop	Input	Output	Description
Phase Generator	f_ref	θ_ref	Integrator: θ_ref = ∫2πf_ref dt
Voltage Outer Loop	V_ref, V_dq	I_dq_ref	PI controller generates current reference
Current Inner Loop	I_dq_ref, I_dq	V_dq*	PI controller with feedforward decoupling
dq/abc	V_dq*, θ	V_abc*	Inverse Park transform
PWM Modulator	V_abc*	Gate Signals	SPWM/SVPWM
3. Signal Flow Summary
text
                    ┌──────────────────────────────────────────────┐
                    │              CONTROL MODES                    │
                    │                                              │
  Breaker Open ────▶│  PRE-SYNC:  PLL tracks grid → align V, f, θ │
                    │                                              │
  Breaker Closed ──▶│  DROOP:     P-f and Q-V respond to grid     │
                    │             deviations (IEEE 1547-2018)      │
                    └──────────────────────────────────────────────┘
Key signal paths:

V_pcc(abc) → PLL → θ_g, ω_g, V_g_mag → Pre-Sync Controller

V_pcc(abc), I_inv(abc) → P/Q Calc → P, Q → Droop Controller

Breaker Status → Mode Selector → selects V_ref, f_ref, θ_ref

V_ref, f_ref, θ_ref → Inner Loops → PWM → Gate Signals → VSI

4. Drawing Tips (for Visio/draw.io/PowerPoint)
Layout: Power stage on top row (left→right), control system below in hierarchical layers.

Color coding:

Red/Orange: Power path (DC → VSI → Filter → PCC → Grid)

Blue: Measurement signals (V_pcc, I_inv, P, Q, θ_g, ω_g, V_g)

Green: Control references (V_ref, f_ref, θ_ref)

Gray: Gate signals and breaker status

Dashed lines: Use for mode-selector switching logic (breaker status).

Label every signal with its variable name (e.g., V_ref_pre, f_ref_droop).

Annotate the droop equations directly on the Droop Controller block for clarity.

Mark the PCC breaker with a clear open/close symbol and a status signal arrow going to the Mode Selector.

5. Optional: Add These Annotations to Your Diagram
"Pre-sync active: Breaker OPEN" next to the Pre-Sync Controller

"Droop active: Breaker CLOSED (IEEE 1547-2018 Clause 5)" next to the Droop Controller

"Seamless transition: V, f, θ aligned before closure" at the Mode Selector

"Grid-supporting: P-f / Q-V droop" as the overall control philosophy label

This structure directly maps to your described system: PLL-based pre-synchronization before PCC closure, and P-f/Q-V droop-based grid support after closure, with a clean mode switch between the two.
