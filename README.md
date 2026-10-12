# ECE 528/L – Homework 2

**Robotics and Embedded Systems with Lab** · CSU Northridge · Fall 2026

Section I: Review Questions

---

## 1. What is a hardware timer? What kinds of tasks do embedded systems commonly use hardware timers for? (5 pts)

A hardware timer is a dedicated counter register that increments or decrements on each edge of a selected clock and operates independently of the CPU.

Common uses include:

- Generating periodic interrupts for tasks
- Creating precise delays and timeouts
- Generating PWM signals for motors, servos, and LEDs
- Counting external events and time-stamping

---

## 2. What is a clock signal used for in embedded systems? List the primary clock sources available on the MSP432P401R and briefly describe each. (5 pts)

A clock signal is a periodic square wave that synchronizes sequential logic. It sets the rate at which the CPU executes instructions, and it acts as the timing reference.


| Source | Type | Description |
|---|---|---|
| LFXTCLK | External crystal | Low-frequency oscillator, typically a 32.768 kHz watch crystal; used for timekeeping (RTC). |
| HFXTCLK | External crystal | High-frequency oscillator, 1–48 MHz; the LaunchPad uses a 48 MHz crystal for an accurate high-speed clock. |
| DCOCLK | Internal | Digitally controlled oscillator; tunable from about 1 to 48 MHz, default 3 MHz. Default source for MCLK and SMCLK. |
| VLOCLK | Internal | Very-low-power, low-frequency oscillator, about 9.4 kHz; low accuracy. |
| REFOCLK | Internal | Low-power reference oscillator, selectable as 32.768 kHz or 128 kHz. |
| MODCLK | Internal | Low-power 25 MHz oscillator used by some peripherals (e.g. ADC, flash). |
| SYSOSC | Internal | System oscillator, about 5 MHz, used for internal system functions such as the ADC and fail-safe operation. |


---

## 3. Consider a 16-bit timer (e.g. Timer_A in up mode). If it counts upward, what is the maximum count value before it rolls over? Show your work. (5 pts)

$$N_{\max} = 2^{16} - 1 = 65{,}536 - 1 = 65{,}535 = \texttt{0xFFFF}$$

The largest possible count is the timer can go to is `TAxCCR0 = 0xFFFF` or `65,535

---

## 4. If a 24-bit timer counts upward from zero at 10 MHz, what is its overflow period? Show your work. (5 pts)

The timer passes through $2^{24}$ states before overflowing back to zero:

$$N = 2^{24} = 16{,}777{,}216 \text{ counts}, \qquad T_{clk} = \frac{1}{10\text{ MHz}} = 100\text{ ns}$$

$$T_{overflow} = N \cdot T_{clk} = 16{,}777{,}216 \times 100 \times 10^{-9}\text{ s} = 1.6777216\text{ s} \approx \mathbf{1.678\text{ s}}$$

---

## 5. Describe the SysTick timer on the MSP432P401R. Provide two example applications. Which clock sources can it use and how are they selected? (5 pts)

Cortex-M4 includes an integrated system timer, SysTick, which provides a simple 24-bit clear-on-write
decrementing wrap-on-zero counter with a flexible control mechanism. 

**Example applications:**

1. An RTOS tick timer that fires at a programmable rate (for example, 100 Hz) and invokes a SysTick routine
2. A simple counter used to measure time to completion and time used.

The clock source is the CPU free running clock **FCLK**. This board doesnt support an external clock.

---

## 6. How many registers does the SysTick timer have and what is the purpose of each? (5 pts)

SysTick has three registers. Three are used to configure and run the timer.

| Register |  Purpose |
|---|---|
| STCSR – Control and Status |  A control and status register to configure its clock, enable the counter, enable the SysTick interrupt, and determine counter status. |
| STRVR – Reload Value |  The reload value for the counter, used to provide the counter's wrap value. |
| STCVR – Current Value |  The current value of the counter. |

---

## 7. Describe the Timer_A peripheral on the MSP432P401R. List its main operating modes and briefly explain each. (5 pts)

Timer_A is a 16-bit timer/counter with up to seven capture/compare registers. Timer_A can support
multiple capture/compares, PWM outputs, and interval timing. Timer_A also has extensive interrupt
capabilities.

The counter mode is selected with the `MC` bits of `TAxCTL`:

| MC | Mode | Behavior |
|:---:|---|---|
| 00 | Stop | The timer is halted. |
| 01 | Up | The timer repeatedly counts from zero to the value of TAxCCR0 |
| 10 | Continuous | The timer repeatedly counts from zero to 0FFFFh. |
| 11 | Up/Down | The timer repeatedly counts from zero up to the value of TAxCCR0 and back down to zero. |



---

## 8. Briefly state the purpose of the following Timer_A registers. (10 pts)

- **`TAxCTL` – Timer_Ax Control**
  Main configuration register: clock source `TASSEL` (bits 9–8), input divider `ID` (bits 7–6), mode control `MC` (bits 5–4), counter clear `TACLR` (bit 2), overflow interrupt enable `TAIE` (bit 1), and overflow flag `TAIFG` (bit 0).

- **`TAxCCTLn` – Capture/Compare Control n**
  Configures each CCR channel: capture edge `CM`, capture input `CCIS`, synchronous capture `SCS`, capture/compare select `CAP`, output mode `OUTMOD` (bits 7–5), interrupt enable `CCIE` (bit 4), output value `OUT`, capture overflow `COV`, and interrupt flag `CCIFG` (bit 0).

- **`TAxR` – Timer_Ax Counter**
  Timer_A register. The TAxR register is the count of Timer_A

- **`TAxCCR0 to TAxCCR6` – Capture/Compare **
  Compare mode: TAxCCRn holds the data for the comparison to the timer value in the Timer_A Register, TAxR. Capture mode: The Timer_A Register, TAxR, is copied into the TAxCCRn register when a capture is performed.

- **`TAxIV` – Interrupt Vector**
  The TAxCCRy CCIFG flags and TAIFG flags are prioritized and combined to source a single interrupt vector. The interrupt vector register TAxIV is used to determine which flag requested an interrupt. The highest-priority enabled interrupt generates a number in the TAxIV register. Any access, read or write, of the TAxIV register automatically resets the highest-pending interrupt flag

- **`TAxEX0` – Expansion 0**
  Contains `TAIDEX` (bits 2–0), a second clock divider from /1 to /8 applied after `ID`, for a total division of up to /64.

---

## 9. What is PWM? Provide at least three example applications that use PWM in embedded systems. (5 pts)

Pulse width modulation (PWM) is a digital signal with a fixed period whose high time (pulse width) is varied. The ratio of high time to period is the duty cycle:

$$D = \frac{t_{on}}{T} \times 100\%, \qquad V_{avg} = D \cdot V_{high}$$


Example applications:

1. **DC motor speed control** 
2. **Servo positioning**
3. **LED dimming and RGB color mixing.**

---

## 10. How many Timer_A modules are available and how many PWM outputs can each module provide? (5 pts)

The MSP432P401R has four Timer_A modules (TA0, TA1, TA2, TA3), each with five outputs CCR0-CCR4.

CCR1-CCR4 can each output PWM for a total of 16 pwm signals.


---

## 11. Initialize P2.4 as a PWM output driven by Timer_A0 CCR1 (TA0.1). (25 pts)


### Code

```c
#define PWM_PERIOD_CONSTANT   60000
#define PWM_DUTY_CYCLE        30000

void Timer_A0_PWM_Init(uint16_t period_constant, uint16_t duty_cycle)
{
    // Return immediately if the duty cycle value is greater than
    // or equal to the given period_constant
    if (duty_cycle >= period_constant) return;

    // Stop Timer A0 by clearing the mode control (MC) bits in the CTL register
    // so that it can be configured safely
    TIMER_A0->CTL &= ~0x0030;

    // Configure pin P2.4 (PM_TA0.1) to use peripheral function mode
    // by setting Bit 4 of the SEL0 register for P2
    // and clearing Bit 4 of the SEL1 register for P2
    P2->SEL0 |= 0x10;
    P2->SEL1 &= ~0x10;

    // Configure pin P2.4 as an output GPIO pin to drive the PWM signal
    // Set Bit 4 of the DIR register for P2
    P2->DIR |= 0x10;

    // Set the Timer A0 Capture/Compare register to the specified period_constant
    // CCR[0] is primarily used as the "period" register
    // General formula: Period = (2*period_constant) / (12 MHz / Prescale Value)
    // In this case: Period = (2*60000) / (12 MHz / 2) = 20 ms
    // Assign the value of period_constant to the CCR[0] register
    TIMER_A0->CCR[0] = period_constant;

    // Configure the Timer A0 expansion register to divide the clock frequency by 1
    // Clear all bits of the EX0 register
    TIMER_A0->EX0 = 0x0000;

    // Configure the output mode as Toggle / Reset for CCR[1]
    // Clear the OUTMOD field of the CCTL[1] register, then set it to 010b
    TIMER_A0->CCTL[1] &= ~0x00E0;
    TIMER_A0->CCTL[1] |= 0x0040;

    // Assign the value of duty_cycle to the CCR[1] register
    // Duty Cycle %: duty_cycle / period_constant
    // In this case: Duty Cycle % = 30000 / 60000 = 50%
    TIMER_A0->CCR[1] = duty_cycle;

    // Modify the following bits in the CTL register
    // Select SMCLK = 12 MHz as timer clock source (TASSEL = 2)
    // Set ID = 1 (Divide timer clock by 2)
    // Set MC = 3 (Up/Down Mode)
    // Set TACLR to clear the timer count and direction
    TIMER_A0->CTL &= ~0x03F0;
    TIMER_A0->CTL |= 0x0274;
}


```

---

## 12. Configure Timer_A0 to generate a periodic interrupt every 2 ms with priority level 2 (IRQ 8). (20 pts)


### Code

```c
#define TIMER_A0_INT_PERIOD   24000   // 2 ms at 12 MHz (CCR0 = 23999)

static void (*Timer_A0_Task)(void);

void Timer_A0_Interrupt_Init(void (*task)(void), uint16_t period)
{
    // Store the user-defined task
    Timer_A0_Task = task;

    // Halt Timer_A0 (clear MC bits 5-4)
    TIMER_A0->CTL &= ~0x0030;

    // TASSEL = 10b (SMCLK), ID = 00b (/1)
    TIMER_A0->CTL &= ~0x03C0;
    TIMER_A0->CTL |=  0x0200;

    // Enable CCR0 interrupt (CCIE, bit 4)
    TIMER_A0->CCTL[0] |= 0x0010;

    // Period: up mode counts 0 .. CCR0 -> CCR0 + 1 ticks
    TIMER_A0->CCR[0] = (period - 1);

    // TAIDEX = 000b (divide by 1)
    TIMER_A0->EX0 = 0x0000;

    // NVIC priority 2 for IRQ 8: IPR2 bits 7-5 -> (2 << 5) = 0x40
    NVIC->IP[2] = (NVIC->IP[2] & 0xFFFFFF00) | 0x00000040;

    // Enable IRQ 8 in the NVIC
    NVIC->ISER[0] |= 0x00000100;

    // TACLR (0x0004) and up mode MC = 01b (0x0010) -> start the timer
    TIMER_A0->CTL |= 0x0014;
}

void TA0_0_IRQHandler(void)
{
    // Acknowledge: clear CCIFG (bit 0)
    TIMER_A0->CCTL[0] &= ~0x0001;

    // Run the user-defined task
    (*Timer_A0_Task)();
}

// Usage: periodic interrupt every 2 ms
// Timer_A0_Interrupt_Init(&My_Task, TIMER_A0_INT_PERIOD);
```

