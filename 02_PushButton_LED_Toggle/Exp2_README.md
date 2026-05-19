# Experiment 2 — GPIO Digital Input: Push Button Controlled LED Toggle

##  Objective
Interface a push button as a digital input device and demonstrate LED state control by toggling it on every confirmed button press.

---

##  Components Required

| Component | Details |
|-----------|---------|
| Microcontroller Board | STM32 Nucleo-F446RE |
| Cable | USB Type-A to Mini-B |
| IDE | STM32CubeIDE |
| Configuration Tool | STM32CubeMX (built into CubeIDE) |

---

##  Background Theory

This experiment builds on GPIO output knowledge from Experiment 1 and introduces **GPIO input**, where the onboard user button drives the onboard LED.

### GPIO Configured as Input
When a GPIO pin is set to **digital input** mode, the microcontroller passively monitors the voltage present on that pin and interprets it as logic `1` (HIGH) or `0` (LOW). Unlike output mode, the MCU does not drive the pin — it only reads it.

### Board Pin Assignments

| Function | Arduino Label | MCU Pin | Default Logic Level |
|----------|---------------|---------|---------------------|
| User LED (LD2) | D13 | **PA5** | Output, Push-Pull |
| User Button (B1) | — | **PC13** | Input, Active LOW |

### Understanding Active-Low Button Logic
The onboard user button **B1** on the Nucleo-F446RE connects to **PC13**, which has a hardware pull-up resistor already fitted on the board. This results in:
- Button **not pressed** → PC13 reads **HIGH (1)**
- Button **pressed** → PC13 reads **LOW (0)**

This convention is called **active-low** — the signal transitions to low when the event of interest (button press) occurs.

### What is Contact Bounce?
Mechanical push buttons do not produce a single clean transition. When pressed, the metal contacts physically bounce several times before settling, generating a burst of rapid HIGH/LOW transitions. This is **contact bounce**, and it can cause the MCU to miscount a single press as multiple events.

A straightforward **software debounce** is implemented here — after a press is detected, a brief `HAL_Delay(200)` pause allows the signal to settle before the next read.

### HAL Functions Utilized

| Function | Purpose |
|----------|---------|
| `HAL_GPIO_ReadPin(GPIOx, GPIO_Pin)` | Returns the current logic level of an input pin (`0` or `1`) |
| `HAL_GPIO_TogglePin(GPIOx, GPIO_Pin)` | Flips the current state of a GPIO output pin |
| `HAL_Delay(ms)` | Blocking millisecond pause — used here for software debounce |

---

## STM32CubeMX Setup

### Step 1 — MCU Selection
- Open STM32CubeMX → **ACCESS TO MCU SELECTOR**
- Search for and select: **STM32F446RETx**
- Click **Start Project**

### Step 2 — GPIO Configuration

**PA5 — LED Output:**

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Output Push Pull |
| GPIO Pull-up/Pull-down | No pull-up and no pull-down |
| Maximum Output Speed | Low |
| User Label | LED *(optional)* |

**PC13 — Push Button Input:**

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Input mode |
| GPIO Pull-up/Pull-down | No pull-up and no pull-down |
| User Label | BTN *(optional)* |

> The Nucleo board already provides a hardware pull-up on PC13, so no internal pull-up configuration is required in CubeMX.

### Step 3 — Clock Source (RCC)
- Go to **System Core → RCC**
- High Speed Clock (HSE): **BYPASS Clock Source**

### Step 4 — Clock Configuration

| Clock Domain | Prescaler | Frequency |
|--------------|-----------|-----------|
| PLL Source | HSE | — |
| SYSCLK | PLL output | 180 MHz |
| AHB (HCLK) | /1 | 180 MHz |
| APB1 | /4 | 45 MHz |
| APB2 | /2 | 90 MHz |

### Step 5 — Project Manager
- Enter a **Project Name** (e.g., `Experiment_2`)
- Toolchain/IDE: **STM32CubeIDE**
- Click **Generate Code** → **Open Project**

### Step 6 — Post-Build Output Settings
- Right-click the project → **Properties → C/C++ Build → Settings → MCU Post Build Outputs**
- Check  **Convert to Binary File (.bin)**
- Check  **Convert to Intel Hex File (.hex)**
- Click **Apply and Close**

---

## Source Code

Open `Core/Src/main.c` and add the following code inside the `while(1)` loop between the user code markers:

```c
/* USER CODE BEGIN WHILE */
while (1)
{
    // Poll PC13 — Active LOW: 0 means button is pressed
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == 0)
    {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);   // Flip LED state on PA5
        HAL_Delay(200);                           // Debounce pause
    }
    /* USER CODE END WHILE */
}
```

### Program Flow

```
Loop begins
    │
    ▼
Sample PC13
    │
    ├── HIGH (button released) ──→ do nothing → loop again
    │
    └── LOW (button held)
            │
            ▼
        Toggle PA5 (LED flips)
            │
            ▼
        Wait 200 ms  ← settles contact bounce
            │
            ▼
        Loop again
```

>  Keep all user code strictly between the `USER CODE BEGIN` and `USER CODE END` markers to prevent it being overwritten during CubeMX regeneration.

---

## Build and Run

1. Connect the Nucleo board via USB
2. Click **Build** (hammer icon) → confirm **0 errors**
3. Click **Debug** → click **Switch** on the perspective dialog
4. Click **Resume (▶)** to start execution
5. Press the blue **B1 user button** on the board — the green LED **LD2** should toggle with every press

---

##  Recorded Observations

### Button Press vs. LED State

| Trial No. | Button Action | PC13 State | LED State Before | LED State After | Remark |
|-----------|---------------|------------|------------------|-----------------|--------|
| 1 | No press (initial) | HIGH | OFF | OFF | No change |
| 2 | 1st Press | LOW | OFF | ON | LED turns ON |
| 3 | Release | HIGH | ON | ON | LED stays ON |
| 4 | 2nd Press | LOW | ON | OFF | LED turns OFF |
| 5 | 3rd Press | LOW | OFF | ON | LED turns ON again |
| 6 | Rapid Press | Bouncing | Varies | Toggles | LED flickers at high speed |

**Key observations:**
- The LED state is **latched** — it holds the new value even after the button is released.
- Every confirmed press triggers exactly one toggle.
- Very rapid presses can cause flickering because the 200 ms window may be insufficient.

---

##  Result

For every valid button press, the LED on PA5 toggled precisely once and retained its state after release. This validates correct setup of **PC13 as a digital input** and **PA5 as a digital output**, with working software-level debounce implemented through `HAL_Delay()`.

---

##  Key Takeaways

- A microcontroller has no inherent notion of a "button press" as an event — it only reads binary logic levels at a point in time. The programmer determines how to interpret signal transitions.
- **Active-low logic** is standard practice in embedded hardware. Always consult the board schematic to know the idle-state polarity of an input signal.
- **Contact bounce** is a physical reality. A single press can generate dozens of spurious transitions within microseconds. A delay-based debounce is the simplest solution, though state-machine or timer-based approaches are more robust.
- The LED in this experiment is **stateful** — unlike the periodic blink in Experiment 1, it remembers and holds its last condition. This forms the basis for latches and toggles in real embedded control systems.
- Combining GPIO input and output in the same application creates a meaningful stimulus-response loop — the foundational building block of all embedded control systems.

---

##  Project Layout

```
02_PushButton_LED_Toggle/
├── Core/
│   ├── Inc/
│   │   └── main.h
│   └── Src/
│       └── main.c          ← User code resides here (while loop)
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
├── Experiment_2.ioc        ← CubeMX pin and clock settings file
└── README.md               ← This document
```
