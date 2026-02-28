# NUC140-PROJECT

# Question 1: UART0 Terminal Echo (USB-UART Bridge)


Description:
    Using a USB-UART adapter as a bridge between a PC Terminal and the NUC140 board, configure UART0 so the board can receive any character sent from the terminal and immediately send it back (echo).
Requirements:
    
    CPU clock frequency: 50 MHz

    UART clock source: 22.1184 MHz

    UART channel: UART0 (TX/RX)

    Frame format: 1 start bit + 8 data bits + no parity + 1 stop bit (8N1)

    Baud rate: 115200 bps

    Communication must work continuously (terminal -> NUC140 -> terminal)

Solution / Implementation:

    Configure system clock (HXT + PLL) to reach 50 MHz.

    Configure UART0 clock source to 22.1184 MHz and set baud rate to 115200.

    Configure UART0 line control: 8 data bits, no parity, 1 stop bit.

    Enable UART0 RX interrupt (RDA).

    In UART interrupt handler:

    Read received byte from RBR (RX buffer).

    Write the same byte to THR (TX buffer) to echo back to terminal.

    Keep main loop empty (event-driven via interrupt).

**Question 2: ADC7 Voltage Monitor + Conditional SPI Output + LCD Display
**
Description:
Use the NUC140 ADC to continuously sample an analog voltage on ADC channel 7 (PA7), compute the input voltage, and display values on the LCD. When the voltage crosses a threshold, transmit a short message through SPI.
Requirements:

CPU clock frequency: 50 MHz

ADC: channel 7, 12-bit, Vref = 3.3 V

ADC mode: continuous scan

SPI (output): 1 MHz, idle high, transmit on rising edge, LSB first, 1 byte per transfer

SPI pins used: PD0, PD1, PD3

LCD must show A/D value and calculated voltage

Solution / Implementation:

Configure system clock to 50 MHz.

ADC setup:

Configure PA7 as ADC7 input and disable digital input path.

Enable ADC clock and set ADC to continuous scan mode.

Start conversion and poll ADF flag each loop.

Read ADC result from ADDR[7].

Convert ADC result to voltage using:

Vin = ADC_value * Vref / (2^12 - 1)

LCD display (SPI3):

Print static labels (group name, reference voltage, resolution).

Update “A/D value” and “Voltage value” continuously.

Conditional SPI output (SPI2):

If Vin > threshold (example: 2.0 V), send a short byte message (e.g., “HEHE”) via SPI2.

If Vin <= threshold, do not transmit (and clear any indicator text on LCD).

**Question 3: Battleship Mini-Game (LCD + Keypad + Button Interrupt + 7-Segment + LED + Buzzer)
**
Description:
Implement a simple Battleship-style game on the NUC140 using an 8x8 grid displayed on the LCD. The player selects target coordinates using a keypad and fires using a push button interrupt. The system tracks score, remaining shots, and shows win/lose feedback using LED and buzzer.
Requirements:

LCD menu screen (title + author + small bitmap)

Gameplay screen:

8x8 grid display (“-” = unshot, “x” = hit)

Show Score, Current X, Current Y on LCD

Input:

Keypad selects coordinate values 1..8

Key “9” toggles between editing X and Y

Fire/shoot using external interrupt button

Feedback:

Hit toggles/flashes an LED

Lose triggers buzzer beep

Game rules:

Win when all ship cells are hit (score reaches total ship cells)

Lose when shot limit is reached (example limit: 16 shots)

Solution / Implementation:

Game state machine:

State 0: Menu screen

State 1: Playing (grid + status info)

State 2: Win screen

State 3: Lose screen

Keypad handling:

Scan keypad matrix to detect key press.

If key = 9: toggle coordinate mode (X <-> Y).

If key in 1..8: update currentX or currentY depending on mode.

Shooting (EINT on button):

In menu state: button starts the game (switch to playing).

In playing state: button triggers “fire”:

Check map_ship[currentY-1][currentX-1].

If hit and not already hit, mark map_display and increment score.

Increment bullets (shots used).

If score == WinCondition -> win state.

If bullets >= shot limit -> lose state.

LCD rendering:

Redraw 8x8 grid each loop from map_display.

Display Score, Current X, Current Y at the side.

7-segment display (Timer interrupt multiplexing):

Show bullets (shots used) and current coordinate value depending on mode.

LED and buzzer:

Flash LED on hit.

Beep buzzer once on lose screen.
