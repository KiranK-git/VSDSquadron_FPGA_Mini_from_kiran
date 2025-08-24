VSDSquadron FPGA Project: Technical Documentation
This document outlines the functionality of the provided Verilog code, the corresponding hardware pin configuration, and the process for integrating the design with the VSDSquadron FPGA Mini board.
1. Verilog Code Functional Overview
The top-level Verilog module is designed to drive the VSDSquadron board's onboard RGB LED. It utilizes two primary Lattice primitives: an internal high-frequency oscillator (SB_HFOSC) and an RGB LED driver (SB_RGBA_DRV).
Internal Oscillator and Counter
●	SB_HFOSC Primitive: This block generates a stable internal clock without requiring external components. With the CLKHF_DIV parameter set to 2, it produces a 12 MHz clock (int_osc) from the base 48 MHz oscillator.
●	Frequency Counter: A 28-bit counter (frequency_counter_i) increments on every rising edge of the 12 MHz clock. The higher-order bits of this counter toggle at a much slower, human-visible rate, making them ideal for creating LED blink patterns.
RGB LED Driver (SB_RGBA_DRV)
The module generates Pulse Width Modulation (PWM) signals for the RGB LED by logically combining the slow-toggling counter bits. The SB_RGBA_DRV primitive is instantiated to control the LED channels based on these signals.
●	Green Channel (led_green): Activated when frequency_counter_i[24] AND frequency_counter_i[23] are high.
●	Blue Channel (led_blue): Activated when frequency_counter_i[24] is high AND frequency_counter_i[23] is low.
●	Red Channel (led_red): Activated when frequency_counter_i[24] is low AND frequency_counter_i[23] is high.
This logic causes the red, green, and blue LEDs to illuminate in a distinct sequence as the counter cycles, creating a simple, repeating blink pattern.
2. PCF Pin Mapping and Hardware Correlation
The Physical Constraints File (PCF) maps the logical signals from the Verilog design to the physical pins of the FPGA, as specified in the VSDSquadron board datasheet.
●	led_red -> Pin 41 (Corresponds to RGB2 on the board)
●	led_blue -> Pin 40 (Corresponds to RGB1 on the board)
●	led_green -> Pin 39 (Corresponds to RGB0 on the board)
●	hw_clk -> Pin 20 (Input for the external hardware oscillator)
●	testwire -> Pin 17 (General-purpose test output)
Each mapping was verified against the board's official schematic to ensure the Verilog outputs correctly drive the intended physical components.
3. Integration and Deployment Workflow
The design is built and flashed onto the FPGA using a toolchain within the provided VirtualBox VM.
Software and Board Setup
1.	VM Preparation: The VSDSquadron FM VirtualBox image is configured and launched.
2.	Board Connection: The FPGA board is connected to the host PC via USB-C, and the FTDI interface is passed through to the VM. The lsusb command is used within the VM to verify a successful connection.
Build and Flash Process
The provided Makefile automates the entire synthesis and programming flow. The process is executed from the project directory in the VM's terminal:
1.	make clean: Clears any artifacts from previous builds.
2.	make build: Synthesizes the Verilog code into a bitstream file using Yosys and NextPNR.
3.	sudo make flash: Programs the FPGA's SRAM with the generated bitstream.
Upon successful flashing, the RGB LED on the board immediately began blinking in the red, green, and blue sequence, confirming that the hardware, Verilog code, and pin constraints were all functioning correctly together.
4. Challenges and Solutions
The integration process was largely smooth, but required careful attention to the following areas:
●	Pin Assignment Verification: The primary challenge was ensuring the PCF pin assignments for led_red, led_green, and led_blue perfectly matched the board's datasheet. Relying solely on the official datasheet for the RGB0/1/2 to pin mapping was crucial for success.
●	VM Device Connection: The FTDI USB device can sometimes disconnect from the VM. It was important to ensure the device was correctly captured by VirtualBox before running the make flash command to avoid programming errors.
●	Oscillator Configuration: The design relies on the CLKHF_DIV parameter to slow the internal oscillator to 12 MHz. Verifying this parameter was set correctly was essential for achieving the intended blink rate.
By systematically verifying these key points, the design was implemented successfully without modifications to the source code.
