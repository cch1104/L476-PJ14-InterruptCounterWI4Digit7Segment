# L476-PJ14-InterruptCounterWI4Digit7Segment
# 📟 STM32 4-Digit 7-Segment Counter

This project implements a **4-digit 7-segment display counter** using STM32.
The value is controlled by external interrupt buttons (UP / DOWN) and displayed using **multiplexing (dynamic scanning)**.

---

## 🧩 Features

* 🔢 Displays numbers from **0000 to 9999**
* ⬆️ Increment using **UP button (EXTI interrupt)**
* ⬇️ Decrement using **DOWN button (EXTI interrupt)**
* ⚡ Uses **multiplexing** to control 4 digits with limited GPIO
* 🧠 Real-time update using interrupt-driven logic

---

## ⚙️ Hardware Configuration

### 📌 GPIO Mapping

| Function      | Pin       |
| ------------- | --------- |
| Segment (a–g) | PC0 ~ PC6 |
| Digit 1 (MSD) | PC7       |
| Digit 2       | PC8       |
| Digit 3       | PC9       |
| Digit 4 (LSD) | PC10      |
| UP Button     | PA0       |
| DOWN Button   | PA1       |

---

## 🔢 Segment Encoding

```c
uint16_t LEDS[] = {
  0x3F, // 0
  0x06, // 1
  0x5B, // 2
  0x4F, // 3
  0x66, // 4
  0x6D, // 5
  0x7D, // 6
  0x07, // 7
  0x7F, // 8
  0x6F  // 9
};
```

---

## 🧠 How It Works

### 1️⃣ Counter Logic (Interrupt)

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin){
    if(GPIO_Pin == UP){
        Count++;
        if(Count > 9999) Count = 0;
    } 
    else if(GPIO_Pin == DOWN){
        Count--;
        if(Count < 0) Count = 0;
    }
}
```

* Uses **external interrupt (EXTI)**
* Keeps value within **0 ~ 9999**

---

### 2️⃣ Digit Extraction

```c
MSD  = Count / 1000;
MID2 = (Count % 1000) / 100;
MID1 = (Count % 100) / 10;
LSD  = Count % 10;
```

---

### 3️⃣ Multiplexing Display

Each digit is turned on one at a time in a fast loop:

```c
GPIOC->ODR = LEDS[MSD];
HAL_GPIO_WritePin(GPIOC, digit1, GPIO_PIN_SET);
HAL_Delay(5);
HAL_GPIO_WritePin(GPIOC, digit1, GPIO_PIN_RESET);
```

* Repeated for all 4 digits
* Persistence of vision makes it appear continuous

---

## 🔁 Main Loop Behavior

* Continuously:

  1. Split `Count` into 4 digits
  2. Output segment pattern
  3. Enable one digit at a time
  4. Repeat quickly

---

## ⚠️ Known Issues / Improvements

### 🚨 1. GPIO Overwrite Risk

```c
GPIOC->ODR = LEDS[x];
```

This may overwrite digit control pins.

👉 Recommended:

```c
GPIOC->ODR = (GPIOC->ODR & 0xFF80) | LEDS[x];
```

---

### 🚨 2. Button Debounce Needed

Mechanical buttons may trigger multiple interrupts.

👉 Suggested fix:

* Add software debounce using `HAL_GetTick()`
* Or use hardware RC filter

---

### 🚨 3. Display Flickering

```c
HAL_Delay(5);
```

👉 Improve by:

* Reducing delay to `1 ms`
* Or using **Timer interrupt scanning (best practice)**

---

### 🚨 4. Volatile Variable

```c
int Count;
```

👉 Should be:

```c
volatile int Count;
```

---

## 🚀 Future Improvements

* ⏱ Use **Timer Interrupt** instead of `HAL_Delay`
* 🔄 Implement **non-blocking display refresh**
* 🧼 Add proper **debounce logic**
* 🎛 Support **hex display (0–F)**
* 💡 Adjust brightness using PWM

---

