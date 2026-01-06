# Creating Custom Engines for Engine Simulator

This guide explains how to create new engine definitions for the engine simulator.

## Units

The simulator accepts both metric and imperial units. Use the `units` helper:

| Metric | Imperial |
|--------|----------|
| `units.mm` | `units.inch` |
| `units.g` | `units.lb` |
| `units.kg` | `units.lb` |
| `units.cc` | `units.L` |
| `units.deg` | `units.deg` |
| `units.rpm` | `units.rpm` |

Example: `84 * units.mm` or `4.25 * units.inch`

Pick one system and stay consistent within your engine file for readability.

## File Location

Engine files are stored in `assets/engines/<manufacturer>/` with the `.mr` extension.

Examples:
- `assets/engines/kohler/kohler_ch750.mr`
- `assets/engines/chevrolet/chev_truck_454.mr`
- `assets/engines/bmw/M52B28.mr`

## Required Information

### Basic Engine Specifications

| Parameter | Description | Example Values |
|-----------|-------------|----------------|
| Engine name | Display name | "BMW M52B28", "Chev. 454 V8" |
| Redline | Maximum safe RPM | 5500-7000 rpm |
| Rev limiter | Cut-off RPM | Slightly above redline |
| Starter torque | Torque for starting | 50-200 lb_ft |
| Starter speed | Initial cranking speed | 500 rpm |

### Crankshaft Data

| Parameter | Description | Metric | Imperial |
|-----------|-------------|--------|----------|
| Stroke | Piston travel (throw = stroke/2) | 69-100 mm | 2.7-4.0 inches |
| Flywheel mass | Weight of flywheel | 2.5-14 kg | 5-30 lb |
| Crankshaft mass | Weight of crankshaft | 2.5-35 kg | 5-75 lb |
| Friction torque | Internal friction | 13.5 N·m | 10 lb_ft |
| TDC angle | Top dead center reference | Depends on config | Depends on config |

### Piston Parameters

| Parameter | Description | Metric | Imperial |
|-----------|-------------|--------|----------|
| Piston mass | Weight per piston | 280-880 g | 0.6-2.0 lb |
| Compression height | Pin to crown distance | 30-45 mm | 1.2-1.8 inches |
| Blowby | Combustion gas leakage | k_28inH2O(0.1-0.6) | k_28inH2O(0.1-0.6) |

### Connecting Rod Parameters

| Parameter | Description | Metric | Imperial |
|-----------|-------------|--------|----------|
| Rod length | Center to center | 100-160 mm | 4.0-6.5 inches |
| Rod mass | Weight per rod | 300-800 g | 0.7-1.8 lb |

### Cylinder Bank Parameters

| Parameter | Description | Metric | Imperial |
|-----------|-------------|--------|----------|
| Bore | Cylinder diameter | 83-110 mm | 3.3-4.3 inches |
| Deck height | Block height | Calculated from geometry | Calculated from geometry |
| Bank angle | V-angle (0 for inline) | 0°, 45°, 60°, 90° | 0°, 45°, 60°, 90° |

### Intake System

| Parameter | Description | Example Values |
|-----------|-------------|----------------|
| Carburetor CFM | Airflow capacity | 500-650 CFM |
| Idle flow rate | Flow at idle | 0.007-0.1 CFM |
| Idle throttle position | Throttle plate angle | 0.99-0.996 |
| Plenum volume | Intake manifold volume | 1-5 L |

### Exhaust System

| Parameter | Description | Example Values |
|-----------|-------------|----------------|
| Primary tube length | Header length | 10-20 inches |
| Primary flow rate | Header CFM | 100-200 CFM |
| Outlet flow rate | Total exhaust CFM | 300-1000 CFM |
| Audio volume | Sound intensity | 0.5-5.5 |

### Camshaft Profile

| Parameter | Description | Example Values |
|-----------|-------------|----------------|
| Duration at 50 thou | Cam duration | 160-260 degrees |
| Lift | Valve lift | 200-400 thou (5-10 mm) |
| Lobe separation | Angle between lobes | 105-114 degrees |
| Gamma | Lobe shape factor | 0.8-1.1 |

### Cylinder Head

| Parameter | Description | Example Values |
|-----------|-------------|----------------|
| Chamber volume | Combustion chamber size | 34-75 cc |
| Intake runner volume | Runner size | 100-150 cc |
| Flow curves | Airflow vs lift | See examples |

### Ignition Timing

| RPM | Typical Advance |
|-----|-----------------|
| 0-1000 | 10-12° |
| 2000 | 20-30° |
| 3000 | 30° |
| 4000+ | 30-38° |

## Engine Configurations

### Firing Orders by Engine Type

| Engine Type | Typical Firing Order | Rod Journal Angles |
|-------------|---------------------|-------------------|
| V-Twin 90° | 1-2 | 0° (shared journal) |
| Inline-4 | 1-3-4-2 | 0°, 180°, 180°, 0° |
| Inline-6 | 1-5-3-6-2-4 | 0°, 120°, 240°, 240°, 120°, 0° |
| V8 (cross-plane) | 1-8-4-3-6-5-7-2 | 0°, 90°, 270°, 180° |

## File Structure Template

```
import "engine_sim.mr"

units units()
constants constants()
impulse_response_library ir_lib()

// 1. Define ignition wires (one per cylinder)
private node wires {
    output wire1: ignition_wire();
    output wire2: ignition_wire();
    // ... add more for each cylinder
}

// 2. Define distributor with firing order
public node my_distributor {
    input wires;
    input timing_curve;
    input rev_limit: 6000 * units.rpm;
    alias output __out:
        ignition_module(timing_curve: timing_curve, rev_limit: rev_limit)
            .connect_wire(wires.wire1, (0.0/N) * cycle)
            .connect_wire(wires.wire2, (1.0/N) * cycle);
            // ... continue for firing order
}

// 3. Define main engine node
public node my_engine {
    alias output __out: engine;

    engine engine(
        name: "My Engine",
        starter_torque: 150 * units.lb_ft,
        starter_speed: 500 * units.rpm,
        redline: 6000 * units.rpm
    )

    // Define crankshaft, pistons, rods, banks, etc.
    // See existing engines for complete examples
}
```

## Activating Your Engine

Edit `assets/main.mr` to use your engine:

```
import "engine_sim.mr"
import "themes/default.mr"
import "engines/manufacturer/your_engine.mr"

use_default_theme()
set_engine(
    your_engine()
)
```

## Reference Examples

- **Simple V-Twin**: `engines/kohler/kohler_ch750.mr` - Small 2-cylinder
- **V8**: `engines/chevrolet/chev_truck_454.mr` - Classic American V8
- **Inline-6**: `engines/bmw/M52B28.mr` - Modern inline-6

## Tips

1. Start by copying an existing engine similar to what you want to create
2. Use consistent units (prefer metric: mm, kg, cc)
3. The `k_carb()` function converts CFM to internal flow units
4. The `k_28inH2O()` function is used for blowby calculations
5. Audio volume affects how loud the engine sounds in simulation
6. Primary tube length affects exhaust tone characteristics
