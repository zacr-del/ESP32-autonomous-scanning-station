# Ultrasonic Scanner

An ultrasonic distance measurement system built with an Arduino Mega 2560 and HC-SR04, being developed into a motorized scanner.

The project focuses on embedded programming, sensor characterization, and reproducible testing. The current prototype measures distance and streams timestamped readings to a computer over USB.

## Current Status

- [x] Connect the HC-SR04 to the Arduino Mega 2560.
- [x] Implement distance measurement with echo timeout handling.
- [x] Stream measurements in CSV format.
- [x] Confirm that readings change with target distance.
- [ ] Complete controlled accuracy and repeatability testing.
- [ ] Add automated data collection and analysis.
- [ ] Integrate servo-driven scanning.
- [ ] Visualize measurements as a two-dimensional map.

## Hardware

| Component | Quantity | Purpose |
|---|---:|---|
| Arduino Mega 2560 | 1 | Sensor control and measurement |
| HC-SR04 ultrasonic sensor | 1 | Distance measurement |
| Jumper wires | 4 | Electrical connections |
| USB cable | 1 | Power, programming, and serial communication |
| Flat target and ruler | 1 each | Reference measurements |

### Wiring

Disconnect power before changing connections.

| HC-SR04 Pin | Arduino Mega 2560 Pin |
|---|---|
| VCC | 5V |
| GND | GND |
| TRIG | Digital 9 |
| ECHO | Digital 10 |

The Mega 2560 uses 5 V logic, so the HC-SR04 ECHO signal connects directly to its digital input.

**This wiring is specific to the Mega 2560.** A 3.3 V controller such as an ESP32 requires voltage reduction on the ECHO signal.

## How It Works

1. The Arduino sends a 10-microsecond HIGH pulse to TRIG.
2. The HC-SR04 emits an ultrasonic burst.
3. The sensor produces an ECHO pulse whose duration represents the sound’s round-trip travel time.
4. The Arduino measures that duration and calculates distance:

   ```text
   distance_mm = echo_duration_us × 0.343 / 2
   ```

The calculation assumes a speed of sound of approximately 343 m/s. Division by two accounts for travel to the target and back.

If no complete echo is measured within 30 milliseconds, the firmware reports a timeout.

## Getting Started

### Requirements

- Arduino IDE
- Arduino AVR Boards package
- Arduino Mega 2560 and HC-SR04 wired as shown above

The firmware uses built-in Arduino functions and requires no additional sensor libraries.

### Upload and Run

1. Open `firmware/mega2560/distance_logger/distance_logger.ino`.
2. Select **Tools → Board → Arduino AVR Boards → Arduino Mega or Mega 2560**.
3. Select the board’s serial port under **Tools → Port**.
4. Upload the sketch.
5. Open Serial Monitor.
6. Set the baud rate to **115200**.
7. Point the sensor toward a flat target perpendicular to its measurement direction.

## Serial Output

The firmware produces CSV-formatted output:

```csv
time_ms,distance_mm,status
103,157.1,OK
204,156.8,OK
335,NA,TIMEOUT
```

*The rows above illustrate the output format; they are not a recorded test dataset.*

| Field | Meaning |
|---|---|
| `time_ms` | Milliseconds since startup, recorded after the echo measurement attempt |
| `distance_mm` | Calculated distance in millimeters, or `NA` after a timeout |
| `status` | `OK` when an echo pulse is measured; otherwise `TIMEOUT` |

`OK` indicates that an echo was measured. It does not guarantee that the echo came from the intended target or that the distance is accurate.

The firmware waits 100 milliseconds after each measurement attempt. The actual sampling interval also includes measurement and serial output time.

## Validation Plan

Secure the sensor and a flat target so that neither moves during each test. Measure the reference distance from the front faces of the sensor transducers to the target.

Collect at least 100 measurement attempts at each reference distance:

| Test Position | Reference Distance |
|---|---:|
| 1 | 150 mm |
| 2 | 300 mm |
| 3 | 500 mm |
| 4 | 1,000 mm |

For each test, record:

- Reference distance and measurement method
- Target material, size, and orientation
- Ambient temperature, if available
- Firmware commit used
- Raw readings, including timeouts

Calculate:

- **Bias:** mean valid reading minus reference distance
- **Mean absolute error:** average absolute difference from the reference distance
- **Standard deviation:** variation among valid readings
- **Timeout rate:** timed-out attempts divided by total attempts

Keep raw data unchanged. Store any corrected or filtered measurements separately.

## Preliminary Observation

During an initial manual check, a target placed at a nominal distance of 6 inches (152.4 mm) produced a reported reading of approximately 157 mm.

This corresponds to an apparent error of approximately +4.6 mm, or 3%.

This observation is not a controlled accuracy result. Reference placement, alignment, and repeated measurements must be verified before drawing conclusions or applying calibration.

## Limitations

- Target angle, shape, and material can affect the returned echo.
- The ultrasonic beam covers an area rather than a single point.
- Nearby surfaces may produce unintended reflections.
- Temperature affects the speed of sound.
- `pulseIn()` blocks execution while waiting for the echo or timeout.
- Displaying one decimal place does not establish 0.1 mm measurement accuracy.
- The current prototype measures distance only; scanning and tracking are not yet implemented.

## Repository Structure

```text
firmware/    Arduino source code
hardware/    Wiring documentation and bill of materials
tests/       Repeatable test procedures
data/raw/    Unmodified measurement logs
analysis/    Data processing and plotting scripts
docs/        Results, photographs, and engineering notes
```

Directories will be added as their contents are developed.

## Next Milestone

Complete baseline distance testing and publish raw measurements with an analysis of accuracy, repeatability, and timeouts. Use those results to guide calibration before adding motorized scanning.
