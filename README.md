# NUC140-PROJECT

# Question 1: UART0 Echo (PC Terminal ↔ NUC140)

Description:
Use a USB-UART adapter as a bridge between the PC Terminal and the NUC140 board. Configure UART0 so any character sent from the Terminal is received by the NUC140 and immediately transmitted back (echo).
- CPU clock: 50 MHz (PLL from HXT)
- UART clock source: 22.1184 MHz
- UART channel: UART0 (TX PB1, RX PB0)
- Frame format: 8N1 (1 start, 8 data, no parity, 1 stop)
- Baud rate: 115200 bps

Solution:
- Use interrupt to detect any changes in the RX.
- If the RX is not empty, this means there's a pending message.
- Receive that message through the RBR register, and send through THR register.

# Question 2: ADC7 Voltage Monitor + LCD Display + Conditional SPI2 TX

Description:
Use ADC channel 7 to continuously sample an analog voltage on PA7, convert it into a 12-bit digital value, calculate the input voltage (Vref = 3.3V), and display both values on the LCD. When the measured voltage is above a threshold, transmit a short message via SPI.
- CPU clock: 50 MHz
- ADC: channel 7, 12-bit, continuous scan mode, Vref = 3.3 V
- Voltage conversion: Vin = ADC_value × 3.3 / (2^12 − 1)
- LCD uses SPI3 for display updates
- SPI2 output: 1 MHz, idle high, transmit on rising edge, LSB first, 8-bit per transfer
- SPI2 pins: PD0, PD1, PD3
- Threshold used in code: Vin > 2.0 V

Solution:
- We're going to calculate the Voltage input as we already know the Reference Voltage (3.3V), Resolution.
- From this we're going to send data through SPI3 to display on the LCD
- If Vin > 2.0 V, transmit “HEHE” over SPI2 (0x48 0x45 0x48 0x45) and show a small indicator text on LCD.
- Otherwise, clear the indicator line.

# Question 3: Battleship Mini-Game

Description:
Implement a Battleship-style game on NUC140 using an 8x8 grid shown on the LCD. The player selects target coordinates using a 3x3 keypad and fires using an external interrupt button. The game tracks score and number of shots, and provides feedback using an LED and buzzer.

LCD screens:
Menu screen: “Battleship”, “By hehe”, plus a 32x32 bitmap
Game screen: 8x8 grid (“-” = unshot, “x” = hit), Score, Current X, Current Y
Input:
- Keypad (3x3 matrix): keys 1–8 set coordinate values
- Key 9 switches coordinate mode (edit X ↔ edit Y)
- Fire button: PB15 external interrupt (EINT1)
Feedback:
- Hit flashes LED on PC12
- Lose triggers buzzer on PB11
- Rules in code:
- Win when Score == total ship cells (WinCon = count of 1s in map_ship)
- Lose when bullets (shots used) reaches 16

Solution:
- Build a simple state flow:
- State 0: menu
- State 1: gameplay
- State 2: win screen
- State 3: lose screen
Keypad scanning:
- Scan PA0–PA5 as a matrix to detect keypress (1–9).
- Key 9 toggles coordinateMode between X and Y.
- Keys 1–8 update currentX or currentY depending on mode.
- Shooting (EINT1_IRQHandler on PB15):
In menu: press button to start (state 0 → 1).
In gameplay:
- Check map_ship[currentY-1][currentX-1].
- If hit and not previously marked, update map_display and increment score.
- Increment bullets each shot.
- Transition to win when score == WinCon, or lose when bullets >= 16.
Display + timing:
- LCD redraws the grid and status text during gameplay.
- Timer0 interrupt multiplexes the 7-segment to show bullets count and the currently selected coordinate value (X or Y).
