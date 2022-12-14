# Low cost shunt regulator
This repository contains the design of a shunt regulator that's designed to be easy to manufacture and mount onto the front of my power supply. This readme will briefly describe the design and it's properties.

## Rationale
Most low-cost (lab/bench) power supplies are only able to supply power, they are not able to sink it. This becomes a problem when powering motor controllers, where the energy generated by "braking" cannot be dissipated elsewhere. I'd like to connect a motor controller to my power supply, as far as I know, there wasn't a cheap/simple solution that would allow me to do that safely. So I decided to try to develop my own solution.

## Requirements
 - Mount right on the binding-posts of my power supply
 - Inherently safe, no chance of burning my house down!
   - Maximum component temperature of <100 degC.
   - Fuses to limit further overloading.
 - Voltage range: 24V - 55V
 - Peak current dissipation: >5A
 - Continuous power dissipation: >5W, for example: 
   - Brake a 1000 gcm^2 rotor, from 3.000 RPM to 0 RPM every second.
   - Brake a 33 gcm^2 rotor from 16.000 RPM to 0 RPM every second.
 - Maximum (one time) energy pulse: 25 Joule
   - Enough to brake a 5000 gcm^2 rotor from 3.000 RPM to 0 RPM.
 - I'd like to use as many "jellybean" components as possible, to keep the cost down and to enable cheap manufacturing services.
 

## Design
The global design can be summarized by a comperator, a MOSFET, a polyfuse and some resistors. The picture below illustrates the circuit.

![](docs/design.png)

The idea is that the comperator simply enables the MOSFET when the bus voltage is higher than a set limit. There is some positive feedback for a bit of hysteresis: To prevent fast fluttering of the MOSFET. The voltage bus has significant capacitance, so the voltage doesn't instantly drop to 0, it "depletes" the capacitors until the voltage level is acceptable again.

The picture below shows the PCB design, it's shaped to fit around my power supply (Owon SPE-series).

![](docs/design_3d.png)

## Simulation
An LTSpice simulation is made to validate the design before prototyping. A constant-current source simulates the regenerative braking of a motor. 

![](docs/ltspice_model.png)

The behaviour is as expected, the voltage is constrained, it looks like a saw-tooth or triangular wave depending on the brake current. The image below shows the bus voltage in green and the current through the resistors in blue.

![](docs/ltspice_result.png)

The image below shows a zoomed-in view.

![](docs/ltspice_result_zoomed.png)


## Usage
 1. Identify the maximum safe voltage, this is the voltage that shall not be exceeded: Vmax.
 2. Set the voltage of the power supply <B>3 volts below Vmax</B>.
 3. Set the current limit of the power supply to 50mA.
 4. Enable the power supply.
 5. Turn the potentiometer until the LED is off, but it's right below turning on.
 6. Increase the power supply voltage, make sure the LED turns on. Keep slowly increasing the voltage, verify that the current increases rapidly before Vmax is reached.
 7. Disable the power supply voltage to 3 volts below Vmax again. Lower is even better, if your application allows.
 8. Increase the power supply current limit again to whatever your application requires.


## Experiments
WARNING: This device is not fully verified yet

### Power supply test
In this test, the shunt regulator is connected to a power supply. The power supply has a current limit of 500mA, the shunt regulator should limit the voltage. The picture below shows the resulting waveform. The yellow trace represents the power supply voltage, the blue trace represents the current from the power supply to the shunt regulator.

![](docs/voltage_limit.png)


### Braking a motor from 1200 RPM to 0 RPM
The main purpose of this shunt regulator is to be able to connect motor-controllers to this power supply, so this test verifies that it will effectively limit the voltage when and ESC is (regeneratively) braking a motor.

The yellow trace represents the system voltage, the blue trace represents the current between the ESC and the PSU + Shunt regulator.

![](docs/1200rpm_brake.png)

The screenshot below shows the same waveform, but zoomed in.

![](docs/1200rpm_brake_zoomed.png)


### Braking a motor from 2500 RPM to 0 RPM
This test is identical to the previous one, but with 4 times the amount of energy.

![](docs/2500rpm_brake.png)

The screenshot below shows the same waveform, but zoomed in.

![](docs/2500rpm_brake_zoomed.png)


### Conclusion
It looks like the shunt regulator works, the bus voltage is effectively limited.