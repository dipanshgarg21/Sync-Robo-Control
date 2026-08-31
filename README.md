# Sync Robo Control

A beginner-level Arduino project demonstrating **wireless robot control using a transmitter and receiver pair**.

The project uses the Arduino Serial Monitor as the control interface. Movement commands entered through the transmitter are converted into short wireless messages and sent to a separate Arduino receiver. The receiver interprets each message and controls two motor channels to move the robot forward, backward, left, right, or stop.

The project demonstrates the fundamental concepts behind wireless communication in robotics:

* Serial communication
* Digital output control
* Wireless data transmission
* Wireless data reception
* Motor direction control
* Transmitter and receiver architecture
* Basic master-slave communication

## Project Overview

The system is divided into two independent Arduino programs:

```text
                 Serial Monitor
                       |
                       v
              +----------------+
              |   Transmitter  |
              |    Arduino     |
              +----------------+
                       |
                       | Wireless
                       | VirtualWire
                       v
              +----------------+
              |    Receiver    |
              |    Arduino     |
              +----------------+
                       |
                       v
                 Motor Driver
                       |
                +------+------+
                |             |
             Left Motor   Right Motor
```

The **transmitter** acts as the controller. It receives a movement command through the Arduino Serial Monitor and transmits the corresponding command wirelessly.

The **receiver** acts as the robot controller. It receives the wireless command and changes its digital outputs to control the motors.

## Features

* Wireless communication between two Arduino boards.
* Serial Monitor-based robot control.
* Forward, backward, left, right, and stop commands.
* Separate transmitter and receiver programs.
* Motor control using four digital output signals.
* Uses the `VirtualWire` library for wireless communication.
* 2000 bits-per-second wireless communication configuration.
* Simple command-based architecture suitable for learning basic robotics and IoT concepts.

## Hardware Requirements

The exact hardware configuration can vary depending on the robot chassis and motor driver being used.

### Core Components

* 2 × Arduino-compatible boards
* 1 × RF transmitter module
* 1 × RF receiver module
* 2 × DC motors
* 1 × Motor driver
* Robot chassis
* Appropriate power supply or battery
* Connecting wires

### Recommended Supporting Components

A motor driver should be used between the Arduino and DC motors. Arduino digital pins should generally not be connected directly to motors because motors require more current than an Arduino GPIO pin is designed to provide.

## Software Requirements

The project uses the Arduino IDE and the `VirtualWire` library.

### Arduino IDE

Install the Arduino IDE and select the appropriate board and port for each Arduino.

### VirtualWire Library

The sketches include:

```cpp
#include <VirtualWire.h>
```

Install the `VirtualWire` library through the Arduino IDE's library management system or add the library manually if it is not already available in your development environment.

> **Note:** `VirtualWire` is an older Arduino library. It is retained in this project because the implementation is designed around it.

## Repository Structure

The repository is organized into separate transmitter and receiver sketches:

```text
Sync-Robo-Control/
│
├── README.md
│
├── VW_Transmitter/
│   └── VW_Transmitter.ino
│
└── VW_Reciever/
    └── VW_Reciever.ino
```

### `VW_Transmitter`

Contains the Arduino program responsible for:

1. Reading commands from the Serial Monitor.
2. Converting commands into wireless messages.
3. Transmitting those messages using the RF transmitter.
4. Controlling the transmitter-side motor output pins.

### `VW_Reciever`

Contains the Arduino program responsible for:

1. Listening for wireless messages.
2. Reading the received command.
3. Interpreting the command.
4. Setting the motor-control outputs accordingly.

## Communication Flow

The communication process is intentionally simple.

For example, when the user enters:

```text
F
```

the transmitter interprets `F` as the forward command.

It sends:

```text
f
```

through the wireless transmitter.

The receiver receives the message and checks the first character:

```cpp
if (msg[0] == 'f')
```

It then activates the appropriate motor outputs.

The same process is used for the other movement commands.

```text
User Input
    |
    v
Serial Monitor
    |
    v
Transmitter Arduino
    |
    | Convert command
    v
Wireless Message
    |
    v
RF Receiver
    |
    v
Receiver Arduino
    |
    | Interpret command
    v
Motor Outputs
```

## Command Reference

The transmitter accepts the following commands:

| Input | Wireless Message | Action        |
| ----- | ---------------- | ------------- |
| `F`   | `f`              | Move forward  |
| `B`   | `b`              | Move backward |
| `L`   | `l`              | Turn left     |
| `R`   | `r`              | Turn right    |
| `S`   | `s`              | Stop          |

Commands are case-sensitive.

Use uppercase letters in the Serial Monitor because the transmitter checks for uppercase input:

```cpp
if (input == 'F')
```

while the corresponding wireless messages are lowercase.

## Transmitter

The transmitter configures four digital pins for motor control:

```cpp
#define L1 5
#define L2 6
#define R1 9
#define R2 11
```

It configures digital pin `10` as the VirtualWire transmission pin:

```cpp
vw_set_tx_pin(10);
```

The wireless communication speed is configured as:

```cpp
vw_setup(2000);
```

The transmitter also initializes Serial communication at:

```cpp
Serial.begin(9600);
```

### Transmitter Command Processing

The program continuously checks whether Serial data is available:

```cpp
if (Serial.available())
{
    input = Serial.read();
}
```

When a recognized command is received, the program:

1. Creates the corresponding wireless command.
2. Sends it using `vw_send()`.
3. Waits for transmission to complete.
4. Sets the local motor-control pins.

For example, the forward command sends `f` and activates both motor channels in the forward direction.

## Receiver

The receiver uses four digital output pins for motor control:

```cpp
#define L1 10
#define L2 11
#define R1 12
#define R2 13
```

The RF receiver is configured on pin `9`:

```cpp
vw_set_rx_pin(9);
```

The same VirtualWire communication speed is used:

```cpp
vw_setup(2000);
```

The receiver starts listening for incoming messages with:

```cpp
vw_rx_start();
```

## Receiver Command Processing

The receiver continuously checks whether a wireless message has been received:

```cpp
if (vw_get_message(msg, &msglen))
```

When a message arrives, the receiver examines its first character.

For example:

```cpp
if (msg[0] == 'f')
```

means that the receiver has received the forward command.

The receiver then changes the motor-control pins to produce the required movement.

## Motor Control Logic

The project controls two motor channels:

* Left motor
* Right motor

Each motor has two control signals.

Conceptually:

```text
             Left Motor          Right Motor
             ----------          -----------
              L1 / L2            R1 / R2
                 |                   |
                 v                   v
            Motor Driver        Motor Driver
                 |                   |
                 v                   v
             Left Motor          Right Motor
```

The direction of each motor is determined by the combination of its two control signals.

### Forward

```text
L1 = HIGH
L2 = LOW

R1 = HIGH
R2 = LOW
```

Both motors operate in the forward direction.

### Backward

```text
L1 = LOW
L2 = HIGH

R1 = LOW
R2 = HIGH
```

Both motors operate in the reverse direction.

### Left

```text
L1 = HIGH
L2 = LOW

R1 = LOW
R2 = LOW
```

The left-side motor is activated while the right-side motor is stopped.

### Right

```text
L1 = LOW
L2 = LOW

R1 = HIGH
R2 = LOW
```

The right-side motor is activated while the left-side motor is stopped.

### Stop

```text
L1 = LOW
L2 = LOW

R1 = LOW
R2 = LOW
```

Both motor channels are turned off.

## Setting Up the Project

### 1. Connect the Hardware

Build two Arduino systems:

```text
Transmitter Arduino
        |
        +-- RF Transmitter
        |
        +-- Serial Monitor


Receiver Arduino
        |
        +-- RF Receiver
        |
        +-- Motor Driver
              |
              +-- Left Motor
              +-- Right Motor
```

The RF transmitter and receiver must be connected to the pins configured in their respective sketches.

### 2. Open the Transmitter Sketch

Open:

```text
VW_Transmitter/VW_Transmitter.ino
```

Select the Arduino board connected to the transmitter and select its corresponding serial port.

Upload the sketch.

### 3. Open the Receiver Sketch

Open:

```text
VW_Reciever/VW_Reciever.ino
```

Select the Arduino board connected to the receiver and select its corresponding serial port.

Upload the sketch.

### 4. Open the Serial Monitor

Open the Serial Monitor for the transmitter Arduino.

Set the baud rate to:

```text
9600
```

Enter one of the supported commands:

```text
F
B
L
R
S
```

The transmitter should send the corresponding wireless command to the receiver.

## Example

To move the robot forward:

```text
Serial Monitor
      |
      | F
      v
Transmitter
      |
      | f
      v
RF Receiver
      |
      v
Receiver Arduino
      |
      v
Both motors forward
```

To stop the robot:

```text
Serial Monitor
      |
      | S
      v
Transmitter
      |
      | s
      v
RF Receiver
      |
      v
Receiver Arduino
      |
      v
Both motors stopped
```

## Important Pin Configuration

The current sketches use the following pins.

### Transmitter

| Component             | Arduino Pin |
| --------------------- | ----------: |
| Left motor control 1  |           5 |
| Left motor control 2  |           6 |
| Right motor control 1 |           9 |
| Right motor control 2 |          11 |
| RF transmitter        |          10 |

### Receiver

| Component             | Arduino Pin |
| --------------------- | ----------: |
| Left motor control 1  |          10 |
| Left motor control 2  |          11 |
| Right motor control 1 |          12 |
| Right motor control 2 |          13 |
| RF receiver           |           9 |

The pin numbers are defined directly in the respective `.ino` files. If the physical wiring differs, update the definitions before uploading the sketches.

## Understanding the Master-Slave Model

This project demonstrates a simple master-slave communication model.

The **transmitter** behaves as the master because it determines which command should be executed.

The **receiver** behaves as the slave because it waits for a command and performs the corresponding action.

```text
MASTER
Transmitter
     |
     | Command
     v
SLAVE
Receiver
     |
     | Motor control
     v
Robot movement
```

This architecture is useful for understanding how a control device can communicate with a separate embedded system.

## Troubleshooting

### Robot does not move

Check:

* Both Arduino boards are powered.
* The RF transmitter and receiver are connected correctly.
* The receiver is connected to the configured pin.
* The motor driver is powered.
* The motors are connected to the motor driver.
* The correct sketch has been uploaded to each Arduino.
* The transmitter and receiver use the same VirtualWire speed.

### Serial commands do not work

Verify that:

* The Serial Monitor is connected to the transmitter Arduino.
* The baud rate is set to `9600`.
* Commands are entered using uppercase letters.
* The entered command is one of `F`, `B`, `L`, `R`, or `S`.

### Wireless communication does not work

Check:

* RF module wiring.
* RF transmitter and receiver compatibility.
* Configured transmitter and receiver pins.
* `vw_setup(2000)` is configured on both devices.
* The RF modules have an appropriate power supply.
* The modules are within a suitable operating range.

### Motors rotate in the wrong direction

Motor direction depends on the motor driver's input configuration and the physical motor wiring.

If forward and backward are reversed, either swap the relevant motor connections or modify the HIGH/LOW combinations in the sketches.

If left and right are reversed, check the left and right motor connections and the corresponding pin definitions.

## Limitations

This project is intentionally simple and focuses on demonstrating wireless command transmission rather than implementing a complete production-ready robot controller.

The current implementation:

* Uses single-character commands.
* Does not implement command authentication.
* Does not provide feedback from the receiver to the transmitter.
* Does not include an acknowledgement mechanism.
* Does not include obstacle detection.
* Does not include automatic error recovery.
* Depends on the range and reliability of the RF hardware.
* Uses the older `VirtualWire` communication library.

These limitations make the project suitable as a starting point for learning wireless robotics and embedded communication.

## Possible Extensions

The project can be extended by adding:

* Joystick-based control.
* Bluetooth or Wi-Fi communication.
* Smartphone control.
* Ultrasonic obstacle detection.
* Automatic obstacle avoidance.
* Speed control using PWM.
* Battery-level monitoring.
* Receiver-to-transmitter acknowledgements.
* Multiple robot control.
* More structured wireless messages.
* A web-based or mobile control interface.

A natural next step would be replacing individual character commands with a structured message format containing command, speed, and optional sensor information.

## Learning Objectives

This project provides practical exposure to several fundamental IoT and embedded-systems concepts:

### Serial Communication

The transmitter receives commands through the Arduino Serial interface.

### Digital I/O

The Arduino uses digital output pins to control the motor driver.

### Wireless Communication

The `VirtualWire` library provides the communication layer between the transmitter and receiver.

### Embedded Control

The receiver converts a received software command into a physical action.

### System Architecture

Separating the transmitter and receiver demonstrates how an embedded system can be divided into independent components that communicate with each other.
