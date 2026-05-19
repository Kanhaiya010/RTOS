# Experiment 1 — GPIO Digital Output: LED Blink Using Software Delay

##  Objective
Configure a GPIO pin on the **STM32F446RE** microcontroller as a digital output and verify LED blinking behavior using software-based delay routines.

---

## Components Required

| Component | Details |
|-----------|---------|
| Microcontroller Board | STM32 Nucleo-F446RE |
| Cable | USB Type-A to Mini-B |
| IDE | STM32CubeIDE |
| Configuration Tool | STM32CubeMX (built into CubeIDE) |

---

##  Background Theory

The **STM32F446RE** microcontroller exposes multiple general-purpose I/O (GPIO) ports — **GPIOA through GPIOH** — each capable of being independently set to one of four operating modes:

| Mode | Description |
|------|-------------|
| **Input** | Reads an incoming digital signal |
| **Output** | Drives a digital HIGH or LOW signal |
| **Alternate Function** | Handed over to on-chip peripherals (UART, SPI, I2C, etc.) |
| **Analog** | Connected to internal ADC/DAC circuits |

### Why Pin PA5?
On the **Nucleo-F446RE** development board, the green user LED (**LD2**) is physically connected to **Arduino header pin D13**, which corresponds to microcontroller pin **PA5**. No external LED or wiring is required — configuring PA5 as an output directly drives LD2.

### Push-Pull Output Mode
PA5 is set up in **push-pull output** mode. In this configuration, the driver circuit can:
- Actively drive the pin **HIGH** (~3.3 V) → LED turns **ON**
- Actively drive the pin **LOW** (0 V) → LED turns **OFF**

This contrasts with open-drain mode, which can only pull the signal low and depends on an external resistor to bring it high.

### HAL Functions Utilized

| Function | Purpose |
|----------|---------|
| `HAL_GPIO_TogglePin(GPIOx, GPIO_Pin)` | Inverts the current logic state of a given GPIO pin |
| `HAL_Delay(ms)` | Stalls execution for the specified number of milliseconds using the SysTick timer |

---

##  STM32CubeMX Setup

### Step 1 — MCU Selection
- Launch STM32CubeMX → **ACCESS TO MCU SELECTOR**
- Search for and select: **STM32F446RETx**
- Click **Start Project**

### Step 2 — GPIO Configuration
- Navigate to **System Core → GPIO**
- Right-click pin **PA5** → Select **GPIO_Output**
- Apply these GPIO settings:

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Output Push Pull |
| GPIO Pull-up/Pull-down | No pull-up and no pull-down |
| Maximum Output Speed | Low |
| User Label | LD2 *(optional)* |

### Step 3 — Clock Source (RCC)
- Go to **System Core → RCC**
- High Speed Clock (HSE): **BYPASS Clock Source**
- This allows the board to use the external clock signal supplied by the ST-Link interface

### Step 4 — Clock Configuration

| Clock Domain | Prescaler | Frequency |
|--------------|-----------|-----------|
| PLL Source | HSE | — |
| SYSCLK | PLL multiplier | 180 MHz |
| AHB (HCLK) | /1 | 180 MHz |
| APB1 | /4 | 45 MHz |
| APB2 | /2 | 90 MHz |

### Step 5 — Project Manager
- Enter a **Project Name** (e.g., `Experiment_1`)
- Toolchain/IDE: **STM32CubeIDE**
- Click **Generate Code** → **Open Project**

### Step 6 — Post-Build Output Settings
- Right-click the project → **Properties → C/C++ Build → Settings → MCU Post Build Outputs**
- Check  **Convert to Binary File (.bin)**
- Check  **Convert to Intel Hex File (.hex)**
- Click **Apply and Close**

---

##  Source Code

After code generation, open `Core/Src/main.c` and insert the following lines inside the `while(1)` loop between the designated user code markers:

```c
/* USER CODE BEGIN WHILE */
while (1)
{
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);   // Flip LED LD2 state on PA5
    HAL_Delay(500);                           // Pause for 500 ms

    /* USER CODE END WHILE */
}
```

>  Always place your code between `USER CODE BEGIN` and `USER CODE END` markers. Anything written outside these blocks will be **erased** during CubeMX code regeneration.

---

##  Build and Run

1. Plug in the Nucleo board via USB
2. Click **Build** (hammer icon) → confirm **0 errors, 0 warnings**
3. Click **Debug** → click **Switch** when the perspective-switch dialog appears
4. Click **Resume (▶)** to begin execution
5. The onboard LED LD2 should start blinking at the chosen rate

---

##  Recorded Observations

The delay value was changed across trials to observe its effect on blink speed:

| S. No. | GPIO Pin | Mode / Pull | Delay (ms) | Approx. Blink Period (s) | Comment |
|--------|----------|-------------|------------|--------------------------|---------|
| 1 | PA5 | Output, PP, No PU/PD | 100 | 0.1 s | Very fast — barely perceptible |
| 2 | PA5 | Output, PP, No PU/PD | 200 | 0.3 s | Moderate and visible |
| 3 | PA5 | Output, PP, No PU/PD | 500 | 1 s | Comfortable, clearly visible |
| 4 | PA5 | Output, PP, No PU/PD | 1000 | 2 s | Slow — ON and OFF phases clearly distinct |

> **Note:** The total blink period is approximately 2 × the delay value, since each ON and OFF phase each consumes one full delay interval.

---

##  Result

GPIO pin **PA5** on the STM32F446RE Nucleo board was successfully configured as a **push-pull digital output** via STM32CubeIDE. The onboard user LED **LD2** blinked correctly at every configured rate. Altering the delay value produced a proportional and clearly visible change in blink frequency, validating correct GPIO output behavior and system clock setup.

---

##  Key Takeaways

- GPIO pins are multi-functional — the same physical pin may serve as input, output, alternate function, or analog, purely based on software configuration.
- **Push-pull mode** actively controls both the high and low states of the pin, making it suitable for direct LED drive without extra components.
- `HAL_Delay()` is a **blocking call** built on the SysTick timer. While waiting, the CPU is fully stalled and cannot process anything else — this is a fundamental limitation of bare-metal super loop programs.
- The **precision of `HAL_Delay()`** depends on accurate SYSCLK configuration. An incorrect clock setup will produce incorrect delays.
- The **super loop (`while(1)`)** is the most basic program structure in embedded software — linear, single-task, and running continuously.
- Blink period = 2 × delay, because one complete ON-OFF cycle takes two delay durations.

---

##  Project Layout

```
01_GPIO_LED_Blink_SoftwareDelay/
├── Core/
│   ├── Inc/
│   │   └── main.h
│   └── Src/
│       └── main.c          ← User code resides here (while loop)
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
├── Experiment_1.ioc        ← CubeMX pin and clock settings file
└── README.md               ← This document
```
