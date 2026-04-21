# The name of the LF application inside "./src" to build/run/flash etc.
LF_MAIN ?= HelloUc

# Add modules used by the LF application
USEMODULE += periph_i2c
USEMODULE += printf_float

# TODO: remove
#USEMODULE += gnrc_netif
#USEMODULE += gnrc_ipv6_default
#USEMODULE += ipv6_addr
#USEMODULE += netdev_default
#USEMODULE += gnrc_netif_ieee802154
#USEMODULE += gnrc_ipv6_default
#USEMODULE += auto_init_gnrc_netif
#USEMODULE += auto_init

# Increase the default stack-size
CFLAGS += -DTHREAD_STACKSIZE_MAIN=4096

# Make default debug output report only info and errors.
CFLAGS += -DLF_COLORIZE_LOGS=0
CFLAGS += -DLF_TIMESTAMP_LOGS=0
CFLAGS += -DLF_LOG_LEVEL_ALL=LF_LOG_LEVEL_ERROR

#CFLAGS += -DLF_LOG_LEVEL_PLATFORM=LF_LOG_LEVEL_DEBUG
#CFLAGS += -DLF_LOG_LEVEL_TRIG=LF_LOG_LEVEL_DEBUG
#CFLAGS += -DLF_LOG_LEVEL_SCHED=LF_LOG_LEVEL_INFO

# Enable reactor-uc features & configuration
EVENT_QUEUE_SIZE?=20
REACTION_QUEUE_SIZE?=20

# Execute the LF compiler if build target is "all"
ifeq ($(firstword $(MAKECMDGOALS)),all)
  _ :=  $(shell $(REACTOR_UC_PATH)/lfc/bin/lfc-dev src/$(LF_MAIN).lf)
endif

# ---- RIOT specific configuration ----
# This has to be the absolute path to the RIOT base directory:
RIOTBASE = $(CURDIR)/RIOT

# If no BOARD is found in the environment, use this default:
BOARD ?= adafruit-feather-nrf52840-sense

# Comment this out to disable code in RIOT that does safety checking
# which is not needed in a production environment but helps in the
# development process:
DEVELHELP ?= 1

# Change this to 0 show compiler invocation lines by default:
QUIET ?= 1

include $(REACTOR_UC_PATH)/make/riot/riot-lfc.mk

