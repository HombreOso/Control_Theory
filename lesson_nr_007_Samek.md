Yes. The useful way to repeat an STM32 lesson manually is not to memorize register values. You want to learn a repeatable process:

**board schematic → MCU datasheet → reference manual → calculate register address/bit fields → write C → compile → flash → inspect registers/debug.**

One limitation first: I could not retrieve the actual YouTube page/transcript for `pQs8vp7JOSk`, so I cannot verify the exact sequence demonstrated in that particular video. The tutorial below therefore reconstructs the standard **bare-metal STM32 GPIO / onboard LED exercise** on your **NUCLEO-F446RE**, and—more importantly—shows exactly how to derive every value from ST's documentation. The same method applies to UART, timers, ADC, SPI, interrupts, etc.

# 1. The documentation you should use

For the NUCLEO-F446RE there are four documents worth learning to navigate.

### 1. Board User Manual — UM1724

This answers:

> **"What is physically connected to what on my Nucleo board?"**

For example, it tells us that the green user LED **LD2** is connected to MCU **PA5**, corresponding to Arduino **D13**. It also tells us that the USER button is on **PC13** and that USART2 uses **PA2/PA3**. ([STMicroelectronics](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf "STM32 Nucleo-64 boards (MB1136) - User manual"))

[NUCLEO-F446RE product page and board documentation](https://www.st.com/en/evaluation-tools/nucleo-f446re.html?utm_source=chatgpt.com)

### 2. STM32F446xC/E Datasheet — DS10693

This answers:

> **"What pins/functions does the STM32F446RE itself have?"**

The current datasheet is DS10693 Rev. 11, May 2026. The STM32F446RE has 512 KB Flash and 128 KB SRAM. ([STMicroelectronics](https://www.st.com/resource/en/datasheet/stm32f446re.pdf "Datasheet - STM32F446xC/E - Arm<Sup>®</Sup> Cortex<Sup>®</Sup>-M4 32-bit MCU+FPU, 225 DMIPS, up to 512 KB flash/128+4 KB RAM, USB OTG HS/FS, seventeen TIMs, three ADCs and twenty communication interfaces"))

Most importantly, use:

**Chapter 4 — Pinout and pin description**

For example, the datasheet identifies PA5 and its alternate functions, and PA2/PA3 as USART2 pins. ([STMicroelectronics](https://www.st.com/resource/en/datasheet/stm32f446re.pdf "Datasheet - STM32F446xC/E - Arm<Sup>®</Sup> Cortex<Sup>®</Sup>-M4 32-bit MCU+FPU, 225 DMIPS, up to 512 KB flash/128+4 KB RAM, USB OTG HS/FS, seventeen TIMs, three ADCs and twenty communication interfaces"))

[STM32F446xC/E datasheet DS10693](https://www.st.com/resource/en/datasheet/stm32f446re.pdf?utm_source=chatgpt.com)

### 3. Reference Manual — RM0390

This is the most important document for bare-metal programming.

It answers:

> **"Which register do I write, at what offset, and what does each bit mean?"**

The important chapters for this lesson are:

|Topic|RM0390|
|---|---|
|Clock / RCC|Chapter 6|
|GPIO|Chapter 7|
|Interrupts|Cortex-M / NVIC sections|
|EXTI|Chapter 11|
|Timers|Timer chapters|
|USART|USART peripheral chapter|
|Memory map|Chapter 2 / register boundary tables|

For GPIO, RM0390 section 7.4 gives the register definitions. For example, `GPIOx_MODER` is offset `0x00`; each pin has two mode bits; `01` means general-purpose output. ([STMicroelectronics](https://www.st.com/st-web-ui/static/active/en/resource/technical/document/reference_manual/DM00135183.pdf?utm_source=chatgpt.com "RM0390
Reference manual
STM32F446xx advanced Arm®-"))

The GPIO base ranges and RCC location are in the memory/register map. GPIOA occupies `0x40020000–0x400203FF`, while RCC is at `0x40023800`. ([STMicroelectronics](https://www.st.com/resource/en/reference_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf?utm_source=chatgpt.com "STM32F446xx advanced Arm<Sup>®</Sup>-based 32-bit MCUs - Reference manual"))

[STM32F446 RM0390 Reference Manual](https://www.st.com/resource/en/reference_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf?utm_source=chatgpt.com)

### 4. ARM Cortex-M4 Programming Manual — PM0214

This is for the **CPU itself**, rather than STM32-specific peripherals.

You use it when learning:

- vector table
    
- exceptions
    
- NVIC
    
- SysTick
    
- interrupt entry/exit
    
- Cortex-M registers
    
- assembly instructions
    

ST lists PM0214 together with the F446 reference manual and datasheet. ([STMicroelectronics](https://www.st.com/en/microcontrollers-microprocessors/stm32f446/documentation.html?utm_source=chatgpt.com "STM32F446 - Documentation - STMicroelectronics"))

---

# 2. What we are going to build

Let's start with the fundamental experiment:

> **Press nothing. The onboard green LED LD2 blinks continuously.**

No:

- HAL
    
- `HAL_GPIO_WritePin()`
    
- CubeMX-generated initialization
    
- `digitalWrite()`
    
- Arduino framework
    

We will directly access memory-mapped registers.

The hardware path is:

```text
CPU
 │
 │ writes
 ▼
RCC register
 │
 │ enables GPIOA clock
 ▼
GPIOA
 │
 │ PA5 output
 ▼
PA5
 │
 ▼
LD2 LED
```

On this particular Nucleo:

```text
LD2
 │
 └── PA5
      │
      └── Arduino D13
```

The board manual explicitly says LD2 is connected to PA5, and that HIGH on the corresponding MCU I/O turns the LED on while LOW turns it off. ([STMicroelectronics](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf "STM32 Nucleo-64 boards (MB1136) - User manual"))

---

# 3. How to find PA5 yourself in the documentation

This is an important habit.

Do **not** start by searching Google for:

> STM32F446 PA5 LED register

Instead:

### Step 1 — Board manual

Open UM1724.

Search:

```text
LD2
```

You find:

```text
User LD2: ... connected to ... PA5
```

and:

```text
SB21 (LD2-LED)
ON: Green user LED LD2 is connected to D13
```

([STMicroelectronics](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf "STM32 Nucleo-64 boards (MB1136) - User manual"))

So:

```text
LED = PA5
```

### Step 2 — MCU datasheet

Search for:

```text
PA5
```

The datasheet tells you that PA5 is an actual MCU I/O pin and lists its alternate functions. ([STMicroelectronics](https://www.st.com/resource/en/datasheet/stm32f446re.pdf "Datasheet - STM32F446xC/E - Arm<Sup>®</Sup> Cortex<Sup>®</Sup>-M4 32-bit MCU+FPU, 225 DMIPS, up to 512 KB flash/128+4 KB RAM, USB OTG HS/FS, seventeen TIMs, three ADCs and twenty communication interfaces"))

For a simple GPIO output we don't need an alternate function.

### Step 3 — Reference manual

Now search RM0390 for:

```text
GPIOx_MODER
```

You discover:

```text
Address offset: 0x00
```

and:

```text
MODERy[1:0]
00 = Input
01 = General purpose output
10 = Alternate function
11 = Analog
```

([STMicroelectronics](https://www.st.com/st-web-ui/static/active/en/resource/technical/document/reference_manual/DM00135183.pdf?utm_source=chatgpt.com "RM0390
Reference manual
STM32F446xx advanced Arm®-"))

Since our pin is **5**:

```text
pin = 5

2 × pin = 10
```

Therefore PA5 is controlled by:

```text
MODER[11:10]
```

and we want:

```text
01
```

So the field should become:

```text
bits 11:10 = 01
```

That is how you derive the famous expression:

```c
GPIOA_MODER |= (1U << 10);
```

But notice something important: we're not memorizing `10`.

We derived it:

```text
PA5
  ↓
pin number = 5
  ↓
2 bits per pin
  ↓
2 × 5 = 10
  ↓
MODER bits 11:10
```

That's the skill you want.

---

# 4. Finding the GPIOA address

Now we have to answer:

> Where is GPIOA in memory?

The MCU datasheet memory map/register boundaries show:

```text
GPIOA = 0x40020000
GPIOB = 0x40020400
GPIOC = 0x40020800
...
```

The F446 reference manual likewise places GPIOA in that AHB1 peripheral region. ([STMicroelectronics](https://www.st.com/resource/en/reference_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf?utm_source=chatgpt.com "STM32F446xx advanced Arm<Sup>®</Sup>-based 32-bit MCUs - Reference manual"))

Therefore:

```c
#define GPIOA_BASE 0x40020000UL
```

Now look at the GPIO register map in RM0390.

We get:

```text
GPIOx_MODER   offset 0x00
GPIOx_OTYPER  offset 0x04
GPIOx_OSPEEDR offset 0x08
GPIOx_PUPDR   offset 0x0C
GPIOx_IDR     offset 0x10
GPIOx_ODR     offset 0x14
GPIOx_BSRR    offset 0x18
```

For example, RM0390 explicitly gives `GPIOx_ODR` offset `0x14` and `GPIOx_BSRR` offset `0x18`. ([STMicroelectronics](https://www.st.com/st-web-ui/static/active/en/resource/technical/document/reference_manual/DM00135183.pdf?utm_source=chatgpt.com "RM0390
Reference manual
STM32F446xx advanced Arm®-"))

Therefore:

```text
GPIOA_MODER = 0x40020000 + 0x00
            = 0x40020000

GPIOA_ODR   = 0x40020000 + 0x14
            = 0x40020014

GPIOA_BSRR  = 0x40020000 + 0x18
            = 0x40020018
```

---

# 5. Why do we have to enable the GPIO clock?

This is one of the most important STM32 concepts.

You might think:

> "GPIOA exists at 0x40020000, so I'll just write to it."

But the peripheral clock is gated.

We therefore need:

```text
RCC
 ↓
enable GPIOA clock
 ↓
GPIOA becomes usable
```

The RCC peripheral itself starts at:

```text
0x40023800
```

The register we need is:

```text
RCC_AHB1ENR
```

with offset:

```text
0x30
```

Therefore:

```text
RCC_AHB1ENR = 0x40023800 + 0x30
            = 0x40023830
```

The AHB1 enable register contains one enable bit per peripheral, including `GPIOAEN`. ([Scribd](https://id.scribd.com/document/809599799/RM0390?utm_source=chatgpt.com "RM0390 | PDF | Computer Science | Computer Hardware"))

GPIOA corresponds to bit 0:

```text
bit 0 = GPIOAEN
```

Therefore:

```c
RCC_AHB1ENR |= (1U << 0);
```

or simply:

```c
RCC_AHB1ENR |= 1U;
```

The first form is much better pedagogically because it tells you what is happening.

---

# 6. First completely manual program

Here is the entire program using only memory addresses.

```c
#include <stdint.h>

/* =========================
 * Memory mapped peripherals
 * ========================= */

#define RCC_BASE        0x40023800UL
#define GPIOA_BASE      0x40020000UL

/* RCC */
#define RCC_AHB1ENR \
    (*(volatile uint32_t *)(RCC_BASE + 0x30UL))

/* GPIOA */
#define GPIOA_MODER \
    (*(volatile uint32_t *)(GPIOA_BASE + 0x00UL))

#define GPIOA_OTYPER \
    (*(volatile uint32_t *)(GPIOA_BASE + 0x04UL))

#define GPIOA_PUPDR \
    (*(volatile uint32_t *)(GPIOA_BASE + 0x0CUL))

#define GPIOA_ODR \
    (*(volatile uint32_t *)(GPIOA_BASE + 0x14UL))

#define GPIOA_BSRR \
    (*(volatile uint32_t *)(GPIOA_BASE + 0x18UL))


#define LED_PIN 5U


static void delay(volatile uint32_t count)
{
    while (count--)
    {
        __asm volatile ("nop");
    }
}


int main(void)
{
    /* -------------------------------------------------
     * 1. Enable GPIOA peripheral clock
     *
     * RCC_AHB1ENR bit 0 = GPIOAEN
     * ------------------------------------------------- */

    RCC_AHB1ENR |= (1U << 0);


    /* -------------------------------------------------
     * 2. Configure PA5 as general-purpose output
     *
     * MODER5 = bits [11:10]
     *
     * 00 = input
     * 01 = output
     * 10 = alternate function
     * 11 = analog
     * ------------------------------------------------- */

    GPIOA_MODER &= ~(3U << (LED_PIN * 2U));
    GPIOA_MODER |=  (1U << (LED_PIN * 2U));


    /* -------------------------------------------------
     * 3. Push-pull output
     *
     * OTYPER5 = 0
     * ------------------------------------------------- */

    GPIOA_OTYPER &= ~(1U << LED_PIN);


    /* -------------------------------------------------
     * 4. No pull-up / pull-down
     *
     * PUPDR5 = 00
     * ------------------------------------------------- */

    GPIOA_PUPDR &= ~(3U << (LED_PIN * 2U));


    /* -------------------------------------------------
     * 5. Blink forever
     * ------------------------------------------------- */

    while (1)
    {
        /* LED ON */
        GPIOA_BSRR = (1U << LED_PIN);

        delay(800000U);

        /* LED OFF */
        GPIOA_BSRR = (1U << (LED_PIN + 16U));

        delay(800000U);
    }
}
```

This is essentially the whole GPIO exercise.

But let's understand **every line** instead of treating it as magic.

---

# 7. Why `volatile`?

This:

```c
(*(volatile uint32_t *)address)
```

means:

> "Treat this memory location as hardware I/O. Do not assume its value is an ordinary C variable."

Without `volatile`, the compiler is allowed to make optimizations that are inappropriate for memory-mapped hardware.

Conceptually:

```c
GPIOA_ODR
```

is not RAM.

It is a hardware register.

Writing:

```c
GPIOA_ODR = 0x20;
```

means:

```text
CPU
 ↓
AHB bus
 ↓
GPIO peripheral
 ↓
output latch
 ↓
PA5
```

---

# 8. Why `uint32_t`?

STM32F4 peripheral registers are generally 32-bit registers.

For example:

```text
GPIOA_MODER
31                     0
+-----------------------+
|       32-bit value    |
+-----------------------+
```

So:

```c
uint32_t
```

is appropriate.

---

# 9. Understanding the bit mask

This line is extremely important:

```c
GPIOA_MODER &= ~(3U << (LED_PIN * 2U));
```

For PA5:

```text
LED_PIN = 5

LED_PIN * 2 = 10
```

and:

```text
3 decimal = binary 11
```

so:

```text
3 << 10
```

becomes:

```text
00000000000000000000110000000000
                      ↑↑
                     11 10
```

The `~` turns it into:

```text
11111111111111111111001111111111
```

Then:

```c
GPIOA_MODER &= mask;
```

clears only bits 11 and 10.

Why do we do that first?

Because:

```c
GPIOA_MODER |= (1U << 10);
```

alone would only force bit 10 to `1`; it would not guarantee that bit 11 is `0`.

The correct general pattern for configuring a multi-bit field is:

```c
REGISTER &= ~MASK;   // clear field
REGISTER |= VALUE;   // insert desired value
```

This is called a **read-modify-write** operation.

---

# 10. Why is PA5 `01`, not `1`?

RM0390 defines two bits per GPIO pin:

```text
Pin 0 → MODER[1:0]
Pin 1 → MODER[3:2]
Pin 2 → MODER[5:4]
Pin 3 → MODER[7:6]
Pin 4 → MODER[9:8]
Pin 5 → MODER[11:10]
...
```

The four possibilities are:

```text
00 = Input
01 = General-purpose output
10 = Alternate function
11 = Analog
```

([STMicroelectronics](https://www.st.com/st-web-ui/static/active/en/resource/technical/document/reference_manual/DM00135183.pdf?utm_source=chatgpt.com "RM0390
Reference manual
STM32F446xx advanced Arm®-"))

Therefore:

```text
PA5
bits 11:10
= 01
```

So:

```c
GPIOA_MODER |= (1U << 10);
```

sets:

```text
bit 10 = 1
bit 11 = 0
```

---

# 11. GPIO output type

The next register is:

```text
GPIOx_OTYPER
offset = 0x04
```

RM0390 says:

```text
0 = push-pull
1 = open-drain
```

([STMicroelectronics](https://www.st.com/st-web-ui/static/active/en/resource/technical/document/reference_manual/DM00135183.pdf?utm_source=chatgpt.com "RM0390
Reference manual
STM32F446xx advanced Arm®-"))

For the LED we use push-pull:

```c
GPIOA_OTYPER &= ~(1U << 5);
```

For PA5 that means:

```text
OTYPER bit 5 = 0
```

---

# 12. GPIO pull-up / pull-down

The pull configuration uses:

```text
GPIOx_PUPDR
```

Each pin again uses two bits.

For PA5:

```text
PUPDR[11:10]
```

For no pull:

```text
00
```

So:

```c
GPIOA_PUPDR &= ~(3U << 10);
```

For this LED example the reset state is already suitable, so technically you could omit this. I keep it in the tutorial because it demonstrates the complete GPIO configuration process.

---

# 13. Turning the LED on

There are two ways.

## Method A — ODR

`GPIOx_ODR` is at offset `0x14`. RM0390 defines each ODR bit as the corresponding port output value. ([STMicroelectronics](https://www.st.com/st-web-ui/static/active/en/resource/technical/document/reference_manual/DM00135183.pdf?utm_source=chatgpt.com "RM0390
Reference manual
STM32F446xx advanced Arm®-"))

For PA5:

```c
GPIOA_ODR |= (1U << 5);
```

LED off:

```c
GPIOA_ODR &= ~(1U << 5);
```

You can even toggle:

```c
GPIOA_ODR ^= (1U << 5);
```

This works.

## Method B — BSRR

The better hardware-oriented approach is:

```c
GPIOA_BSRR = (1U << 5);
```

to set PA5.

And:

```c
GPIOA_BSRR = (1U << (5 + 16));
```

to reset PA5.

RM0390 specifically describes BSRR as providing atomic bit set/reset. ([STMicroelectronics](https://www.st.com/st-web-ui/static/active/en/resource/technical/document/reference_manual/DM00135183.pdf?utm_source=chatgpt.com "RM0390
Reference manual
STM32F446xx advanced Arm®-"))

Why is this nice?

The 32-bit BSRR looks conceptually like:

```text
31                         16 15                          0
+----------------------------+-----------------------------+
|     BR15 ... BR0           |       BS15 ... BS0          |
+----------------------------+-----------------------------+
                 ↑
              reset          set
```

For PA5:

```text
BS5 = bit 5
BR5 = bit 21
```

Therefore:

```text
set PA5:
1 << 5

reset PA5:
1 << 21
```

No read-modify-write is needed.

---

# 14. Why does the LED actually light?

The important distinction is:

```text
MCU pin PA5 ≠ LED automatically
```

The STM32 datasheet tells us PA5 exists.

The **board schematic/user manual** tells us what the board physically connected to PA5.

That is why the documentation chain matters:

```text
RM0390
    ↓
"What does GPIO register X do?"

Datasheet
    ↓
"What is PA5?"

NUCLEO schematic / UM1724
    ↓
"What is PA5 physically connected to?"
```

For the Nucleo-F446RE, UM1724 says the user LED is connected to PA5 and that HIGH corresponds to LED ON. ([STMicroelectronics](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf "STM32 Nucleo-64 boards (MB1136) - User manual"))

---

# 15. Clock: why the delay works without configuring the PLL

This is another important lesson.

The STM32F446 starts using the internal HSI clock.

RM0390 describes HSI as a **16 MHz internal RC oscillator**, and states that after reset HSI is selected as the system clock. ([STMicroelectronics](https://www.stmicroelectronics.com/resource/en/reference_manual/dm00135183.pdf?utm_source=chatgpt.com "RM0390
Reference manual
STM32F446xx advanced Arm®-"))

So our simple program can run without doing a PLL setup.

Conceptually:

```text
Reset
  ↓
HSI ≈ 16 MHz
  ↓
CPU starts executing
```

That is perfect for your first experiment.

Later, when you want:

```text
180 MHz CPU
```

you need to study:

```text
RCC_CR
RCC_PLLCFGR
RCC_CFGR
FLASH_ACR
PWR
```

and the clock tree in RM0390.

Do **not** start there. First learn GPIO with the reset clock.

---

# 16. The delay loop is deliberately crude

This:

```c
static void delay(volatile uint32_t count)
{
    while (count--)
    {
        __asm volatile ("nop");
    }
}
```

is **not** a proper embedded-system timebase.

It is just useful for demonstrating:

```text
CPU executes instructions
        ↓
time passes
        ↓
GPIO changes
```

Its actual time depends on:

- CPU clock
    
- compiler optimization
    
- generated assembly
    
- wait states
    
- pipeline behavior
    

So:

```c
delay(800000);
```

does **not** mean:

```text
800 ms
```

in any rigorous sense.

Once you understand GPIO, replace this with **SysTick** or a general-purpose timer.

---

# 17. Now the genuinely "manual" part: how does the program start?

There is one layer people often skip.

You write:

```c
int main(void)
{
    ...
}
```

But the processor does not magically know that `main()` is where execution begins.

On reset, the Cortex-M reads the **vector table**.

The first entries are conceptually:

```text
Address +0x00 → initial stack pointer
Address +0x04 → Reset_Handler
Address +0x08 → NMI_Handler
Address +0x0C → HardFault_Handler
...
```

So for a true from-scratch project we need:

```text
Flash
 ├── vector table
 ├── Reset_Handler
 └── main()
```

---

# 18. Minimal startup file

Here is a deliberately tiny startup file.

### `startup.c`

```c
#include <stdint.h>

extern int main(void);

/*
 * Linker provides the top of RAM.
 */
extern uint32_t _estack;

void Reset_Handler(void);

void Default_Handler(void)
{
    while (1)
    {
        /* Trap here */
    }
}

/*
 * Minimal vector table.
 *
 * We are not enabling interrupts in this first exercise,
 * so the remaining entries can point to Default_Handler.
 */
__attribute__((section(".isr_vector")))
const void *vector_table[] =
{
    &_estack,           /* Initial stack pointer */
    Reset_Handler,      /* Reset */
    Default_Handler,    /* NMI */
    Default_Handler,    /* HardFault */
    Default_Handler,    /* MemManage */
    Default_Handler,    /* BusFault */
    Default_Handler,    /* UsageFault */
    0,
    0,
    0,
    0,
    Default_Handler,    /* SVCall */
    Default_Handler,    /* Debug monitor */
    0,
    Default_Handler,    /* PendSV */
    Default_Handler     /* SysTick */
};


void Reset_Handler(void)
{
    main();

    while (1)
    {
    }
}
```

This is intentionally minimal.

For production code you would also deal with:

```text
.data initialization
.bss zeroing
C runtime
constructors
interrupt vectors
VTOR
FPU setup
clock initialization
```

But doing the minimal version first is excellent for understanding.

---

# 19. Linker script

We need to tell the linker:

> "Where is Flash and where is RAM?"

The datasheet gives the STM32F446 memory map, including Flash starting at `0x08000000` and SRAM at `0x20000000`. ([STMicroelectronics](https://www.st.com/resource/en/datasheet/stm32f446re.pdf "Datasheet - STM32F446xC/E - Arm<Sup>®</Sup> Cortex<Sup>®</Sup>-M4 32-bit MCU+FPU, 225 DMIPS, up to 512 KB flash/128+4 KB RAM, USB OTG HS/FS, seventeen TIMs, three ADCs and twenty communication interfaces"))

For the STM32F446RE variant we have:

```text
Flash = 512 KB
SRAM  = 128 KB
```

([STMicroelectronics](https://www.st.com/resource/en/datasheet/stm32f446re.pdf "Datasheet - STM32F446xC/E - Arm<Sup>®</Sup> Cortex<Sup>®</Sup>-M4 32-bit MCU+FPU, 225 DMIPS, up to 512 KB flash/128+4 KB RAM, USB OTG HS/FS, seventeen TIMs, three ADCs and twenty communication interfaces"))

### `stm32f446re.ld`

```ld
ENTRY(Reset_Handler)

MEMORY
{
    FLASH (rx)  : ORIGIN = 0x08000000, LENGTH = 512K
    RAM   (rwx) : ORIGIN = 0x20000000, LENGTH = 128K
}

SECTIONS
{
    .isr_vector :
    {
        . = ALIGN(128);
        KEEP(*(.isr_vector))
    } > FLASH

    .text :
    {
        *(.text*)
        *(.rodata*)
    } > FLASH

    .bss :
    {
        . = ALIGN(4);
        *(.bss*)
        *(COMMON)
        . = ALIGN(4);
    } > RAM

    _estack = ORIGIN(RAM) + LENGTH(RAM);
}
```

Notice how much of this came directly from documentation:

```text
Flash:
0x08000000

RAM:
0x20000000

Flash size:
512 KB

RAM size:
128 KB
```

That's exactly the manual engineering process you were asking for.

---

# 20. Project structure

You can now make this tiny project:

```text
stm32f446-baremetal/
│
├── main.c
├── startup.c
├── stm32f446re.ld
└── Makefile
```

---

# 21. Makefile

```makefile
TARGET = stm32f446

CC      = arm-none-eabi-gcc
OBJCOPY = arm-none-eabi-objcopy
SIZE    = arm-none-eabi-size

CFLAGS  = -mcpu=cortex-m4 \
          -mthumb \
          -O0 \
          -g3 \
          -ffreestanding \
          -fno-builtin \
          -Wall \
          -Wextra

LDFLAGS = -T stm32f446re.ld \
          -nostdlib \
          -Wl,--gc-sections,-Map=$(TARGET).map

SRC = main.c startup.c

OBJ = $(SRC:.c=.o)

all: $(TARGET).bin

$(TARGET).elf: $(OBJ)
	$(CC) $(CFLAGS) $(OBJ) $(LDFLAGS) -o $@
	$(SIZE) $@

$(TARGET).bin: $(TARGET).elf
	$(OBJCOPY) -O binary $< $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f *.o *.elf *.bin *.map
```

Then:

```bash
make
```

You should get something like:

```text
stm32f446.elf
stm32f446.bin
stm32f446.map
```

---

# 22. Flashing it manually

The Nucleo board has an integrated ST-LINK, so you do not need a separate programmer/debug probe. ([Keil](https://www.keil.arm.com/boards/stmicroelectronics-nucleo-f446re-revc-fb1ab08/guide/?utm_source=chatgpt.com "Arm Keil | STMicroelectronics NUCLEO-F446RE"))

ST's current STM32CubeProgrammer provides a command-line interface and supports SWD/ST-LINK programming. ([STMicroelectronics](https://www.st.com/content/ccc/resource/technical/document/user_manual/group0/76/3e/bd/0d/cf/4d/45/25/DM00403500/files/DM00403500.pdf/jcr%3Acontent/translations/en.DM00403500.pdf?utm_source=chatgpt.com "STM32CubeProgrammer software description - User manual"))

Connect the Nucleo's **ST-LINK USB connector** to the PC.

Then:

```bash
STM32_Programmer_CLI -c port=SWD
```

You should see the device.

Program:

```bash
STM32_Programmer_CLI \
    -c port=SWD \
    -w stm32f446.bin 0x08000000 \
    -v \
    -rst
```

ST documents the `-w <file> <start_address>` syntax and `-v` verification. ([Dev.ST](https://dev.st.com/stm32cube-docs/prog/2.23.0/en/docs/markup/CubeProg_Command_Lines.html?utm_source=chatgpt.com "MCU command lines - STM32CubeProgrammer 2.23.0 documentation"))

The address:

```text
0x08000000
```

comes from the MCU memory map.

Again: **don't memorize it blindly—learn where to find it.**

---

# 23. What you should inspect in the debugger

This is where the lesson becomes much more interesting.

Connect with STM32CubeIDE, VS Code + GDB, or another debugger, but don't let the IDE configure the peripheral for you.

Put a breakpoint after:

```c
RCC_AHB1ENR |= (1U << 0);
```

Inspect:

```text
0x40023830
```

You should see bit 0 set.

Then inspect:

```text
0x40020000
```

which is GPIOA_MODER.

For PA5:

```text
bits [11:10] = 01
```

Then inspect:

```text
0x40020014
```

GPIOA_ODR.

And:

```text
0x40020018
```

GPIOA_BSRR.

This is the point where you begin to see the processor as:

```text
C statement
      ↓
machine instruction
      ↓
memory write
      ↓
peripheral register
      ↓
electrical signal
      ↓
LED
```

That is the essence of bare-metal embedded programming.

---

# 24. A much better way to learn the RM0390

Do not read RM0390 from page 1 to page 1300.

Instead, whenever you want to implement a peripheral, use this workflow.

## Example: "I want PA5 to output something"

Ask:

### Question 1 — Which MCU pin?

Search the board manual:

```text
LD2
```

Result:

```text
PA5
```

### Question 2 — Is PA5 capable of GPIO?

Search datasheet:

```text
PA5
```

Result:

```text
I/O
```

### Question 3 — Which peripheral clock controls GPIOA?

Search RM0390:

```text
GPIOAEN
```

Result:

```text
RCC_AHB1ENR bit 0
```

### Question 4 — What register configures direction?

Search:

```text
GPIOx_MODER
```

Result:

```text
offset 0x00
two bits per pin
01 = output
```

### Question 5 — What register controls output?

Search:

```text
GPIOx_ODR
```

or:

```text
GPIOx_BSRR
```

### Question 6 — How do I know the physical board behaves as expected?

Go back to:

```text
UM1724 / schematic
```

This workflow is far more valuable than memorizing STM32 code.

---

# 25. The same methodology for a button

The Nucleo-F446RE user button B1 is connected to:

```text
PC13
```

The board manual explicitly documents this. ([STMicroelectronics](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf "STM32 Nucleo-64 boards (MB1136) - User manual"))

Now imagine the exercise is:

> LED follows button.

Your investigation becomes:

```text
B1
 ↓
PC13
 ↓
GPIOC
 ↓
RCC_AHB1ENR
 ↓
GPIOC_MODER
 ↓
input mode
 ↓
GPIOC_IDR
 ↓
PC13 state
 ↓
GPIOA_BSRR
 ↓
PA5
 ↓
LED
```

Notice how almost all the work is **documentation navigation**, not C programming.

---

# 26. The same methodology for UART

This is especially useful because it teaches you alternate functions.

On the Nucleo-F446RE:

```text
PA2 = USART2_TX
PA3 = USART2_RX
```

The board manual says the Nucleo routes USART2 to the onboard ST-LINK/VCP by default, using the appropriate solder-bridge configuration. ([STMicroelectronics](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf "STM32 Nucleo-64 boards (MB1136) - User manual"))

The datasheet identifies PA2 as having `USART2_TX` and PA3 as having `USART2_RX`. ([STMicroelectronics](https://www.st.com/resource/en/datasheet/stm32f446re.pdf "Datasheet - STM32F446xC/E - Arm<Sup>®</Sup> Cortex<Sup>®</Sup>-M4 32-bit MCU+FPU, 225 DMIPS, up to 512 KB flash/128+4 KB RAM, USB OTG HS/FS, seventeen TIMs, three ADCs and twenty communication interfaces"))

Then your documentation workflow is:

```text
1. Board manual
   ↓
   PA2 / PA3

2. Datasheet
   ↓
   USART2 alternate function

3. Datasheet alternate-function table
   ↓
   Which AF number?

4. RM0390 GPIO chapter
   ↓
   MODER = alternate function

5. RM0390 GPIO AFR
   ↓
   program AF number

6. RCC
   ↓
   enable GPIOA clock
   enable USART2 clock

7. USART chapter
   ↓
   BRR
   CR1
   CR2
   ISR
   TDR
   RDR

8. Board manual
   ↓
   VCP connected to ST-LINK
```

That is exactly the pattern you will use later for SPI, I²C, timers and PWM.

---

# 27. A useful distinction: Datasheet vs Reference Manual

This distinction is worth memorizing.

### Datasheet

Think:

> **"What is the chip?"**

It gives you:

```text
pins
electrical characteristics
packages
memory size
maximum clock
alternate functions
I/O capabilities
```

For example:

```text
PA5
PA2
PA3
```

([STMicroelectronics](https://www.st.com/resource/en/datasheet/stm32f446re.pdf "Datasheet - STM32F446xC/E - Arm<Sup>®</Sup> Cortex<Sup>®</Sup>-M4 32-bit MCU+FPU, 225 DMIPS, up to 512 KB flash/128+4 KB RAM, USB OTG HS/FS, seventeen TIMs, three ADCs and twenty communication interfaces"))

### Reference Manual

Think:

> **"How do I control the chip?"**

It gives you:

```text
registers
bits
offsets
clock trees
peripheral operation
reset values
state machines
interrupt behavior
```

For example:

```text
GPIOA_MODER
GPIOA_ODR
GPIOA_BSRR
RCC_AHB1ENR
```

([STMicroelectronics](https://www.st.com/st-web-ui/static/active/en/resource/technical/document/reference_manual/DM00135183.pdf?utm_source=chatgpt.com "RM0390
Reference manual
STM32F446xx advanced Arm®-"))

### Board manual / schematic

Think:

> **"What did ST connect on this development board?"**

For example:

```text
PA5 → LD2
PC13 → USER button
PA2/PA3 → ST-LINK VCP
```

([STMicroelectronics](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf "STM32 Nucleo-64 boards (MB1136) - User manual"))

---

# 28. Register-level code versus CMSIS

There is one more useful intermediate level.

Our code currently says:

```c
#define GPIOA_MODER \
    (*(volatile uint32_t *)(0x40020000UL + 0x00UL))
```

That is **raw MMIO**.

CMSIS lets you write something like:

```c
RCC->AHB1ENR |= (1U << 0);

GPIOA->MODER &= ~(3U << 10);
GPIOA->MODER |=  (1U << 10);

GPIOA->BSRR = (1U << 5);
```

This is still register-level programming.

The difference is that ST's device header gives you:

```text
RCC
GPIOA
MODER
ODR
BSRR
...
```

rather than you defining the addresses yourself.

For learning, I recommend this progression:

```text
Level 1
raw addresses
     ↓
Level 2
CMSIS register structures
     ↓
Level 3
LL / low-level drivers
     ↓
Level 4
HAL
     ↓
Level 5
application architecture
```

The important part is understanding **Level 1 → Level 2** before becoming dependent on HAL.

---

# 29. One improvement to the LED program

Once the basic program works, change:

```c
GPIOA_BSRR = (1U << LED_PIN);
delay(800000U);

GPIOA_BSRR = (1U << (LED_PIN + 16U));
delay(800000U);
```

to:

```c
GPIOA_ODR ^= (1U << LED_PIN);
delay(800000U);
```

Now you are experimenting with the ODR register.

Then remove the artificial delay entirely and investigate:

```text
SysTick
```

The Cortex-M4 has SysTick, and the STM32 documentation describes it as part of the timer/CPU infrastructure. ([STMicroelectronics](https://www.st.com/resource/en/datasheet/stm32f446re.pdf "Datasheet - STM32F446xC/E - Arm<Sup>®</Sup> Cortex<Sup>®</Sup>-M4 32-bit MCU+FPU, 225 DMIPS, up to 512 KB flash/128+4 KB RAM, USB OTG HS/FS, seventeen TIMs, three ADCs and twenty communication interfaces"))

Then you can build a proper:

```c
delay_ms()
```

based on a known clock.

That is the natural next lesson.

---

# 30. The complete learning path I'd use with your board

For the **STM32F446RE Nucleo-64**, I would do the exercises in this order:

```text
1. LED output
   PA5
   RCC
   MODER
   ODR / BSRR

        ↓

2. Button input
   PC13
   IDR
   pull configuration

        ↓

3. LED controlled by button
   GPIO input + GPIO output

        ↓

4. SysTick
   deterministic millisecond timing

        ↓

5. UART
   PA2 / PA3
   Alternate Function
   USART registers

        ↓

6. UART interrupt
   NVIC
   vector table
   ISR

        ↓

7. Timer
   PSC
   ARR
   CNT

        ↓

8. PWM
   Alternate Function
   timer channel

        ↓

9. ADC
   GPIO analog mode
   ADC registers

        ↓

10. DMA
    peripheral ↔ RAM

        ↓

11. SPI / I2C

        ↓

12. RTOS / ROS2-level systems
```

For someone coming from electrical engineering, this progression is particularly useful because it connects the **electrical behavior** to the **software register model** instead of treating STM32 as a black box.

---

# 31. Your "manual investigation sheet"

For every future embedded exercise, make a little table like this before writing code:

|Item|Question|Source|
|---|---|---|
|Board connection|What physical device am I controlling?|UM1724 / schematic|
|MCU pin|Which GPIO pin?|Datasheet|
|Alternate function|Which AF number?|Datasheet|
|Peripheral|GPIO/UART/TIM/ADC?|Datasheet|
|Peripheral clock|Which RCC bus?|RM0390|
|Clock-enable register|Which register/bit?|RM0390|
|GPIO mode|MODER value?|RM0390|
|Pull/type/speed|PUPDR/OTYPER/OSPEEDR?|RM0390|
|Data register|ODR/IDR/TDR/RDR/etc.?|RM0390|
|Interrupt|Which IRQ/vector?|RM0390 + PM0214|
|Memory address|Peripheral base + offset|RM0390|
|Physical result|What should I measure/see?|Schematic / datasheet|

That table is essentially your **manual replacement for CubeMX**.

---

# 32. The most important thing to understand

When you see code such as:

```c
RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
GPIOA->MODER |= (1 << 10);
GPIOA->ODR |= (1 << 5);
```

don't think:

> "Those are STM32 incantations."

Think:

```text
GPIOAEN
   ↓
GPIOA clock enabled

MODER5
   ↓
PA5 configured as GPIO output

ODR5
   ↓
PA5 driven HIGH

PA5
   ↓
board connection

LD2
   ↓
LED ON
```

Once you can make that chain mentally, moving between STM32 families becomes dramatically easier.

## Official references

[NUCLEO-F446RE / MB1136 documentation](https://www.st.com/en/evaluation-tools/nucleo-f446re.html?utm_source=chatgpt.com)

[STM32F446xC/E Datasheet DS10693](https://www.st.com/resource/en/datasheet/stm32f446re.pdf?utm_source=chatgpt.com)

[STM32F446 Reference Manual RM0390](https://www.st.com/resource/en/reference_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf?utm_source=chatgpt.com)

[STM32CubeProgrammer documentation / CLI](https://dev.st.com/stm32cube-docs/prog/latest/en/index.html?utm_source=chatgpt.com)

The next useful step is to take the **exact circuit/program shown in your YouTube lesson** and reproduce it line-by-line on the F446RE. Since the video's page itself was not retrievable here, paste its transcript or the lesson title/screenshots, and the mapping can be made exact down to **"look at RM0390 section X → register Y → bits Z → write this line of C."**