# reactor-uc unofficial DATE26 tutorial

![RIOT OS Logo](https://www.riot-os.org/assets/img/riot-logo.png)
![nrf-board](https://cdn-learn.adafruit.com/assets/assets/000/088/831/large1024/sensors_Feather_Sense_top.jpg?1583171226)

- **Git:** <https://github.com/riot-os/RIOT>
- **Supported Boards:** <https://www.riot-os.org/boards.html>
- **Documentation:** <https://doc.riot-os.org/>
- **Adafruit page:** <https://learn.adafruit.com/adafruit-feather-sense>
- **RIOT docs for board:** <https://api.riot-os.org/group__boards__adafruit-feather-nrf52840-sense.html>

______

This is a tutorial for Lingua Franca applications running on RIOT OS with the [Adafruit Feather Sense](https://learn.adafruit.com/adafruit-feather-sense) board. 

## 1. Prerequisites

### 1.1. Basic

You must use one of the following operating systems:

- `Linux` Officially supported are Debian & Ubuntu
- `macOS`

Your system must have the following software packages (you likely have at least some of these already):

- `git` — [a distributed version control system](https://git-scm.com/)
- `make` — Version 4.0 or higher required for RIOT (see [macOS Hints](#macos-hints))
- `java` — [Java 17](https://openjdk.org/projects/jdk/17)
- Optional: `nix` — [a purely functional package manager](https://nix.dev/tutorials/install-nix)

#### Installation on Debian & Ubuntu

```bash
sudo apt update
sudo apt install git openjdk-17-jdk openjdk-17-jre nix cmake build-essential python3
sudo pip install pyserial
```

#### Installation on macOS

```bash
brew install git cmake openjdk@17 make
curl -L https://nixos.org/nix/install | sh
pip install pyserial
```

Note that on macOS, `make` will be installed as `gmake`, so use `gmake` instead of `make` in all commands below.

### 1.2. Micro C Target for Lingua Franca

This template uses [reactor-uc](https://github.com/lf-lang/reactor-uc), the "micro C" target for Lingua Franca. Clone this repo with one of the following commands:

#### Clone via HTTPS

```bash
git clone https://github.com/lf-lang/reactor-uc.git --recurse-submodules
```

#### Or Clone via SSH

```bash
git clone git@github.com:lf-lang/reactor-uc.git --recurse-submodules
```

And make sure that the `REACTOR_UC_PATH` environment variable is pointing to it.

### 1.3. Install a Cross-Compiler for your Board

This README only covers arm-based boards. For boards having a CPU with different architecture, please check which cross-compilers are available for your operating system.

A quick way to check if you already have an arm cross-compiler installed:

```bash
which arm-none-eabi-gcc
```

#### Debian & Ubuntu

```bash
sudo apt install gcc-arm-none-eabi 
```

#### Nix

The template repo includes support for using the [nix](https://nix.dev) package manager to perform the installation. It is currently set to support ARM-based boards that use the `arm-none-eabi-gcc` cross-compiler.

The following command creates a shell environment in which all necessary dependencies are installed.

```bash
nix develop
```

This creates a new shell in which the cross-compiler is available.
**IMPORTANT**: Don't forget to run ``nix develop`` again when you return to your project in a new shell.

#### MacOS

On Mac, you need to install the full toolchain for  `arm-none-eabi-gcc` using the following HomeBrew command.

```bash
brew install --cask gcc-arm-embedded
```

**IMPORTANT** You should not install the arm-none-eabi-gcc formula. If you accidentally did this, you can uninstall the formula and install the full toolchain like this:

```bash
brew uninstall arm-none-eabi-gcc
brew install --cask gcc-arm-embedded
```

## 2. Start Using this Repository

The RIOT OS sources are provided as a submodule of the new repository, to fetch them do:

```bash
git submodule update --init --recursive
```

## 3. Configure the Makefile

The repository has a `Makefile` that governs the build. By default, it compiles the LF program in `src/HelloUc.lf`. To compile a different program, edit the `Makefile` to set `LF_MAIN` to your program and `BOARD` to your board. 

```Makefile
LF_MAIN ?= HelloUc
BOARD ?= adafruit-feather-nrf52840-sense
```

Alternatively, you can override the board on the command line. For example:

```sh
make LF_MAIN=HelloUc all
```

## 4. LED Reactor

Open `src/Led.lf` and implement a reactor for controlling the on-board LEDs. 
You can find the reactor-uc reaction api [here](http://micro-lf.org/documentation/reaction_api/), you can read the value of a trigger with `port_name->value` and you 
can set ports with `lf_set(port_name, <value>)`.


## 5. HelloUc Reactor

In `HelloUc.lf`, use the LED reactor to toggle the LED at a fixed rate.


## 6. Build

```bash
make all
```

Or override the Makefile configuration with parameters:

```bash
make LF_MAIN=HelloUc BOARD=adafruit-feather-nrf52840-sense all
```

## 7. Flash the Program onto Your Board

```bash
make flash
```

Or override the Makefile configuration with parameters:

```bash
make LF_MAIN=HelloUc BOARD=adafruit-feather-nrf52840-sense flash
```

## 8. Open a Terminal

You can open a terminal that interacts with stdin and stdout of your program as follows:

```bash
make term
```

This will display any output your program generates using, for example, `printf`.

You can also get debug output from the `reactor-uc` runtime by changing the following line in the `Makefile`:

```
CFLAGS += -DLF_LOG_LEVEL_ALL=LF_LOG_LEVEL_ERROR
```

to

```
CFLAGS += -DLF_LOG_LEVEL_ALL=LF_LOG_LEVEL_DEBUG
```

## 9. Sensor Makefile Configuration

Add the following lines to your Makefile to enable I2C support in RIOT:

```Makefile
# so i2c and printf support for floats is compiled into the riot kernel
USEMODULE += periph_i2c 
USEMODULE += printf_float
```

## 10. Implementing the Sensor

The Adafruit Feather Sense includes many sensors. We'll focus on the [LSM6DS33](https://www.pololu.com/file/0J1087/LSM6DS33.pdf) accelerometer and gyro. See the [full sensor list](https://learn.adafruit.com/adafruit-feather-sense) for details.

The file `LSM6DS33.lf` has a section for reading sensor values that you need to complete. Refer to the [RIOT I2C documentation](https://api.riot-os.org/group__drivers__periph__i2c.html) and read register `OUTX_L_G` (see the datasheet for details).

To test your sensor implementation, compile and run `Sensor.lf` (which includes the sensor reactor). This command compiles, flashes, and opens the serial console:


```bash
make LF_MAIN=Sensor BOARD=adafruit-feather-nrf52840-sense all flash term
```


## 11. Using Sensor Values

Make the LED blink faster or slower based on the device's orientation. When flat on a table (angle ≈ 0), use a 1-second LED toggle period. 

```
OFFSET = 4 * PI ~ 12.566
ORIENTATION_TO_TIME = 4 * PI * 1000 ~ 12566
PERIOD = ORIENTATION_TO_TIME / (current_angle + OFFSET)
```

This produces our `PERIOD` in milliseconds.

Flash the program and rotate the device around its longest axis to see the LED blink rate change.

## 12. Annotations

reactor-uc supports many annotations; see the [full list](http://micro-lf.org/documentation/annotations/). In this scenario, we'll add a `timeout` and configure a buffer size for actions.

- **Timeout:** add the `@timeout(<time_value>)` annotation to the main reactor.
- **Action Buffer Size:** add the `@max_pending_event(<number>)` before the action declaration.

Now recompile your program.

You can validate if the code generator correctly adjusted the action buffer size by opening `src-gen/Sensor/Sensor/Sensor.h` file and searching for the `LF_DEFINE_ACTION_STRUCT` macro.

The timeout property can be found inside the `src-gen/Sensor/lf_start.c` file inside the `DynamicScheduler_ctor`.



## 13. Delayed Connections and the Buffer Annotation

Before we go federated it is good to look into the `@buffer` annotation, which can be added to delayed connections to increase the associated buffer for storing the values. If you have a timer with a high frequency it is very easy to run out of space inside the connection.

Compile the `src/DelayedConn.lf` program and see when it stops dropping values, by changing the `@buffer` annotation.

## 14. The Link Local Address of the Device

Add this temporarily to your Makefile

```
USEMODULE += gnrc_netif
USEMODULE += gnrc_ipv6_default
USEMODULE += ipv6_addr
USEMODULE += netdev_default
USEMODULE += gnrc_netif_ieee802154
USEMODULE += gnrc_ipv6_default
USEMODULE += auto_init_gnrc_netif
USEMODULE += auto_init
```

Then compile and run the `src/Ipv6LinkLocal.lf` program this program will print the Ipv6 Link Local address of this board. Copy and Save this address.


## 15. Going Federated

We need to tell reactor-uc that we want the COAP network channel to be added to the compilation unit:

```Makefile
CFLAGS += -DNETWORK_CHANNEL_COAP_RIOT
```

The compilation command also changes because now we need to specify which federate to compile and flash:

```bash
make LF_MAIN=SimpleCoapFederated LF_FED=r1 BOARD=adafruit-feather-nrf52840-sense all flash term

make LF_MAIN=SimpleCoapFederated LF_FED=r2 BOARD=adafruit-feather-nrf52840-sense all flash term
```

In reactor-uc you also configure the network channels by adding annotations.

```
 @interface_coap(name="if1", address="<Paste Your Link Local Address Here>")
```

Coordinate with your neighbor agree on which federate your board runs and exchange the Ipv6 link local addresses accordingly. Also make sure your program have the same structure.

This creates a CoAP network channel named `if1` with the specified IPv6 address. The `@link` annotation specifies which network channel interface to use for a connection.

In the serial output you should now see the two federates communicating. 

## 16. The Final Boss - Federated Blinking

![The final boss](https://images-wixmp-ed30a86b8c4ca887773594c2.wixmp.com/f/8bcbac46-c322-4678-9738-e08774e90a1e/ddc4s1q-db698149-fd79-4460-b9f5-4bc49b11dc41.png/v1/fill/w_894,h_894/bowser_brawl_render_remake_by_unbecomingname_ddc4s1q-pre.png?token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1cm46YXBwOjdlMGQxODg5ODIyNjQzNzNhNWYwZDQxNWVhMGQyNmUwIiwiaXNzIjoidXJuOmFwcDo3ZTBkMTg4OTgyMjY0MzczYTVmMGQ0MTVlYTBkMjZlMCIsIm9iaiI6W1t7ImhlaWdodCI6Ijw9MjAwMCIsInBhdGgiOiIvZi84YmNiYWM0Ni1jMzIyLTQ2NzgtOTczOC1lMDg3NzRlOTBhMWUvZGRjNHMxcS1kYjY5ODE0OS1mZDc5LTQ0NjAtYjlmNS00YmM0OWIxMWRjNDEucG5nIiwid2lkdGgiOiI8PTIwMDAifV1dLCJhdWQiOlsidXJuOnNlcnZpY2U6aW1hZ2Uub3BlcmF0aW9ucyJdfQ.YbH13sRdLsjM7thEWKlIS902vqHGgzwvP6UZ4EjfO2M)

You made it to the final level.

Open the the `src/FederatedBlinking.lf` file now we want to combine everything learned and
here we want to let the local Blink faster if neighbors microcontroller is turned.

The make sure all the necessary Modules are added inside your `Makefile` additionally make sure you flash the correct verion onto the correct board (otherwise the addresses dont match). 


