# STM32F4 and ESP8266 IoT project

The goal of this project is to create a door lock that can be accessed and controlled from anywhere. The STM32F4 module is used to controll the servo motor, ESP8266 is used for Wi-Fi access so that the user can track lock state from Android app regardles of his/her whereabouts.

## Architecture
Android app  ──UDP──►  Python server  ──Wi-Fi/UDP──►  ESP8266  ──UART──►  STM32F407  ──PWM──►  servo
   (client)              (relay)                       (Wi-Fi)            (lock logic)        (lock)
Android app (AndroidClient, Java) — sends lock/unlock commands and shows
the current state. Networking runs off the UI thread in UdpClientThread.
Python server (PythonServer/Server.ipynb) — a UDP relay that connects the
phone app and the lock, forwarding commands and status between them.
ESP8266 — provides Wi-Fi connectivity and bridges the network to the STM32
over a UART serial link.
STM32F407 (lock_controller) — the lock controller. Receives commands over
UART and drives the servo.

## Firmware (STM32F407)
Written in C with the STM32 HAL, generated/configured with CubeMX
(lock_controller.ioc) and running on FreeRTOS (CMSIS-RTOS v2). Two tasks:
uartRxThread — listens on USART2 for commands coming from the ESP8266
and updates the lock state.
servoCheckThread — drives the servo to the locked/unlocked position.
The servo is controlled by a PWM signal generated on TIM3 (setPWM()),
varying the pulse width to set the servo angle.

## Hardware
STM32F407 development board (STM32F407G-DISC1)
ESP8266 Wi-Fi module
Servo motor (PWM-controlled)
Wiring: STM32 ⇄ ESP8266 over UART (USART2)

## Build and flash
The lock_controller firmware is an STM32CubeIDE project.
Open lock_controller/ in STM32CubeIDE (or regenerate from
lock_controller.ioc in CubeMX).
Build and flash to the STM32F407 over ST-Link.
Flash the ESP8266 with its Wi-Fi firmware and wire it to USART2.
Start the Python server (PythonServer/Server.ipynb) on a reachable host.
Install the Android app from AndroidClient and point it at the server.
Repository layout
.
├── lock_controller/   # STM32F407 firmware (C, HAL, FreeRTOS, CubeMX)
├── AndroidClient/     # Android app (Java) — UDP client
└── PythonServer/      # Python UDP relay server