OpenPLC ESP32 Start/Stop Flasher
PLC Programming | OpenPLC | ESP32 | Ladder Logic

This project demonstrates a PLC-style start/stop control system built with an ESP32 and OpenPLC. The system uses a seal-in circuit with Stop priority and two TON timers to control an LED that flashes 500 ms on and 500 ms off.
Hardware

The hardware used for this project includes an ESP32, two pushbuttons, two 10 kΩ pull-down resistors, an LED, a resistor for the LED, and jumper wires.

Pin Mapping

The Start button is connected to GPIO 18, the Stop button is connected to GPIO 19, and the LED is connected to GPIO 5.

<img width="1462" height="237" alt="PinMapping1" src="https://github.com/user-attachments/assets/f0f3d519-7f7a-4a17-8b10-a6abc5c03a1b" />

I/O Table
Device	Type	Pin	Address/Alias
Start	Digital Input	GPIO 18	%IX0.0 / start
Stop	Digital Input	GPIO 19	%IX0.1 / stop
LED	Digital Output	GPIO 5	%QX0.0 / light
Run	Internal BOOL	—	RUN
Timer 1 Done	Internal BOOL	—	T1_DONE
Timer 2 Done	Internal BOOL	—	T2_DONE

<img width="1527" height="396" alt="VariableTable1" src="https://github.com/user-attachments/assets/fa34f1e9-3781-40a8-8788-04f4d0b074f9" />

How It Works

The program uses four ladder logic rungs.

Rung 1: Start/Stop Seal-In

The first rung controls the RUN bit. When the Start button is pressed, RUN turns on. The RUN contact then keeps the circuit on after the Start button is released. The Stop contact is in series with the circuit, so pressing Stop turns RUN off. This gives the Stop button priority if both buttons are pressed at the same time.

Rung 2: Timer 1

The second rung uses the T1 TON timer. When RUN is active and T2_DONE is off, Timer 1 starts. Its preset time is 500 ms. When the timer finishes, T1_DONE turns on.
Rung 3: Timer 2

The third rung uses the T2 TON timer. When T1_DONE turns on, Timer 2 starts. It also has a preset time of 500 ms. When Timer 2 finishes, T2_DONE turns on.

Rung 4: LED

The fourth rung controls the LED using T1_DONE. When T1_DONE is on, the LED turns on. When T1_DONE turns off, the LED turns off. The two timers work together to make the LED flash 500 ms on and 500 ms off.

<img width="982" height="922" alt="PLCcode1" src="https://github.com/user-attachments/assets/e888f366-b156-4cc9-9554-8adb335c428f" />


Problems and Fixes
Floating Inputs

One problem I had was that the inputs were floating when the buttons were not being pressed. Touching the board could cause the LED to change because the ESP32 input could pick up electrical noise. I fixed this by adding a 10 kΩ pull-down resistor to each pushbutton input. The pull-down resistors keep the inputs at a known LOW state when the buttons are not pressed.

<img width="3024" height="4032" alt="IMG_4897" src="https://github.com/user-attachments/assets/0f6e3fe9-4a45-4ee9-880c-e12ea65c9e19" />



DINT Location Error

I also had an error when trying to use the wrong variable type for an I/O address. Physical digital inputs and outputs use addresses such as %IX0.0 and %QX0.0. A BOOL is used with an X address because an X address represents one digital bit that can be either 0 or 1. Internal variables such as RUN, T1_DONE, and T2_DONE are BOOL variables and do not need a physical I/O address.

TON Block Error

The TON timer initially showed a red outline because the timer needed its own variable. I fixed this by creating separate timer variables for T1 and T2. Each timer then has its own memory for keeping track of its timing.

Testing

I tested the project in three different ways.

First, I pressed the Start button. The LED started flashing 500 ms on and 500 ms off.

Second, I pressed the Stop button. The LED stopped flashing and went dark.

Third, I held both the Start and Stop buttons at the same time. Nothing happened because the Stop button has priority over Start.



https://github.com/user-attachments/assets/cd3c86bc-f57a-4897-b2e9-d6895bd95f69



Honest Limitations

This project uses an ESP32 running OpenPLC, so it is not the same as a real industrial PLC system. The Stop button used in this project is normally open and is not a fail-safe emergency stop. In a real industrial application, an emergency stop would use a hardwired safety circuit and safety-rated equipment. This project is meant to demonstrate PLC programming concepts using an ESP32 and OpenPLC.

Conclusion

This project demonstrates how ladder logic can be used to create a start/stop seal-in circuit and control a flashing LED. It also helped me understand digital inputs and outputs, pull-down resistors, BOOL variables, TON timers, and how PLC programs interact with physical hardware.
