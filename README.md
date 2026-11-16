# SEGGER SystemView

This example project shows how to record and analyze FreeRTOS and system
activity with SEGGER SystemView and pyOCD on an STMicroelectronics
[NUCLEO-H743ZI2][nucleo-h743zi2] board.

## Quick Start

1. Install both required desktop applications:

   - [Keil Studio for VS Code](https://marketplace.visualstudio.com/items?itemName=Arm.keil-studio-pack)
   - The [SEGGER SystemView Application](https://www.segger.com/downloads/systemview/)

2. Clone this repository, for example using
   [Git in VS Code](https://code.visualstudio.com/docs/sourcecontrol/intro-to-git),
   or download the ZIP file. Open the repository root folder in VS Code and let
   the required tools and software packs install.
3. Connect the [NUCLEO-H743ZI2][nucleo-h743zi2] board and use the CMSIS view
   [action buttons](https://github.com/Open-CMSIS-Pack/vscode-cmsis-solution#action-buttons)
   to build, load, and run the project with the **systemview-file** target set.
4. Press the blue user button on the board several times to generate SystemView
   messages.
5. Stop the application and the Debug Session with the CMSIS Debugger [Debug Toolbar](https://github.com/Open-CMSIS-Pack/vscode-cmsis-debugger#debugger-user-interface).
6. Start the SEGGER SystemView Application and open
   `./out/SEGGER-SystemView+STM32H743ZITx.SVDat`.
7. Use SEGGER SystemView to inspect the recorded system and FreeRTOS activity.

## About SEGGER SystemView

[SEGGER SystemView](https://www.segger.com/products/development-tools/systemview/)
is a real-time software analysis tool that records and visualizes the runtime
behavior of embedded systems. It shows detailed timing for tasks, interrupts,
software timers, and application events, helping developers find inefficiencies,
resource conflicts, and unintended interactions.

The target-side SystemView instrumentation sends events through SEGGER
[Real Time Transfer (RTT)][segger-rtt].
In this example, pyOCD accesses RTT through the target's debug port and writes
the SystemView data to a recording file. The SEGGER SystemView Application then
opens that file for graphical analysis. This approach does not require a
dedicated UART or extra target pins.

![SEGGER SystemView Application showing recorded FreeRTOS activity][systemview-screenshot]

The same approach can be adapted for other devices and boards. For product
details, downloads, and usage information, see the official
[SEGGER SystemView product page](https://www.segger.com/products/development-tools/systemview/),
[SystemView downloads](https://www.segger.com/downloads/systemview/), and
[SystemView User Guide](https://doc.segger.com/UM08027_SystemView.html).

## Requirements

To use SEGGER SystemView as shown in this example, install the following
software packs:

- `SEGGER::RTT` pack, version 8.58.0 or later
- `SEGGER::SystemView` pack, version 4.12.0 or later
- `ARM::CMSIS-Compiler` pack, version 2.3.0 or later
- `ARM::CMSIS-FreeRTOS` pack, version 11.0.1 or later

The target-side `SEGGER::SystemView` software pack provides the instrumentation
code. The separately installed
[SEGGER SystemView Application](https://www.segger.com/downloads/systemview/)
is required to view and analyze the recorded data.

## Adding SystemView Recording to an Application

1. In the
   [Software Components view](https://github.com/Open-CMSIS-Pack/vscode-cmsis-solution#software-components-view),
   enable the following components:

   - `SEGGER` → `RTT`
   - `SEGGER` → `SystemView`, `FreeRTOS` variant

2. Add the required code to your application:

   - At the end of `FreeRTOSConfig.h`, add:

     ```c
     #include "SEGGER_SYSVIEW_FreeRTOS.h"
     ```

   - Include the configuration header in the application source:

     ```c
     #include "FreeRTOSConfig.h"
     ```

   - Start recording before initializing the RTOS:

     ```c
     traceSTART();
     ```

   - Send application messages to SEGGER SystemView:

     ```c
     SEGGER_SYSVIEW_PrintfHost("Button pressed %d times", count);
     ```

3. In the `csolution.yml` file, add the following
   [`rtt:`](https://open-cmsis-pack.github.io/cmsis-toolbox/pyOCD-Debugger/#rtt)
   node under the
   [`debugger:`](https://open-cmsis-pack.github.io/cmsis-toolbox/pyOCD-Debugger/#debugger)
   node:

   ```yaml
   rtt:
     - channel:
         - number: 1
           mode: systemview
   ```

### Optional Live Connection

pyOCD can also stream data directly to the SEGGER SystemView Application over a
network connection instead of recording it to a file. To use this mode, change
**Target Set** from **systemview-file** to **systemview-server** in the
[Manage Solution view](https://github.com/Open-CMSIS-Pack/vscode-cmsis-solution#manage-solution-view).

[segger-rtt]: https://www.segger.com/products/debug-probes/j-link/technology/about-real-time-transfer/
[systemview-screenshot]: ./SEGGER-SystemView/SystemView_Screenshot.png "SEGGER SystemView"
[nucleo-h743zi2]: https://www.keil.arm.com/boards/stmicroelectronics-nucleo-h743zi2-reve-a8d98ff/guide/
