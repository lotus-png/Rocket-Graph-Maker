# 1D Model Rocket Flight Simulator

A lightweight Python simulation of a model rocket's vertical (1D) flight from launch to apogee. It uses a tabulated motor thrust curve, accounts for propellant mass loss and quadratic aerodynamic drag, and integrates the equations of motion with a 4th-order Runge-Kutta (RK4) solver.

## Features

- Tabulated thrust curve with linear interpolation
- Time-varying mass, based on the fraction of total impulse delivered
- Quadratic drag that always opposes velocity
- RK4 integration of height and velocity
- Reports burnout time, max velocity, apogee height, time to apogee, and total impulse
- Saves a 4-panel plot of height, velocity, acceleration, and mass vs. time

## Requirements

- Python 3.9+
- NumPy **2.0 or newer** (the script uses `np.trapezoid`, which does not exist in older versions; on NumPy 1.x, replace it with `np.trapz`)
- Matplotlib

```bash
pip install numpy matplotlib
```

## Usage

```bash
python rocket_sim.py
```

(Replace `rocket_sim.py` with whatever you named the file.)

The script prints a summary to the console and writes `rocket_flight.png` to the working directory.

```
Burnout time:   ...
Max velocity:   ...
Apogee height:  ...
Time to apogee: ...
Total impulse:  ...
```

## Configuring Your Rocket

Edit the constants near the top of the script.

| Parameter | Default | Units | Description |
|---|---|---|---|
| `MASS_EMPTY` | 0.030 | kg | Rocket structure plus motor casing, no propellant |
| `MASS_PROPELLANT` | 0.015 | kg | Propellant burned during flight |
| `DRAG_COEFFICIENT` | 0.75 | – | Drag coefficient (Cd) |
| `FRONTAL_AREA` | 0.0015 | m² | Cross-sectional area of the rocket |
| `AIR_DENSITY` | 1.225 | kg/m³ | Air density at sea level |
| `G` | 9.81 | m/s² | Gravitational acceleration |
| `DT` | 0.01 | s | Integration time step |

### Thrust curve

`THRUST_CURVE` is an array of `[time (s), thrust (N)]` pairs. Replace it with data for your motor (manufacturer data sheets and the [ThrustCurve.org](https://www.thrustcurve.org) database list these). The first point should be at `t = 0` and the last point should have zero thrust, since the last time value is treated as burn time.

## How It Works

**State:** `[height, velocity]`

**Forces:**

```
F_net = F_thrust(t) - F_drag(v) - m(t) * g
F_drag = 0.5 * rho * v * |v| * Cd * A
a = F_net / m(t)
```

**Mass model:** Propellant is assumed to burn in proportion to the impulse delivered so far:

```
m(t) = m_empty + m_propellant * (1 - cumulative_impulse(t) / total_impulse)
```

**Integration:** Each step of size `DT` uses classic RK4 on `d/dt [h, v] = [v, a(t, v)]`.

**Stopping condition:** The simulation ends at apogee, defined as the first time after burnout when velocity drops to zero or below. There is also a 60 s safety cutoff.

## Output

`rocket_flight.png` contains four stacked plots sharing a time axis:

1. Height (m), with apogee in the title
2. Velocity (m/s)
3. Acceleration (m/s²)
4. Mass (kg)

A dashed green line marks motor burnout on each plot.

## Assumptions and Limitations

- **1D only.** Vertical flight with no wind, weathercocking, or angle of attack.
- **Constant Cd and air density.** No variation with Mach number or altitude, which is reasonable for low-altitude model rockets.
- **Constant frontal area and no parachute.** The simulation stops at apogee and does not model descent or recovery.
- **No launch rail or lug friction.**
- **Linear impulse-based propellant burn.** Real motors may deviate.
- Results are only as good as your inputs. Validate against a tool such as [OpenRocket](https://openrocket.info) or real flight data before relying on them.

## Possible Improvements

- Descent and parachute deployment phases
- Altitude-dependent air density (ISA atmosphere model)
- Mach-dependent drag coefficient
- Load thrust curves directly from `.eng` files
- Command-line arguments for rocket parameters
- Comparison against OpenRocket or flight-computer data

## License

Add your license here (e.g., MIT).
