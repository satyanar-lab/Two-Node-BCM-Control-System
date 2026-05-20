# Hand-rolled Makefile for the BCM Master firmware (STM32L152RE, Cortex-M3).
# Mirrors the STM32CubeIDE project so it can be built from the command line
# with arm-none-eabi-gcc (no IDE required).

TARGET    := bcm_master
BUILD_DIR := build
OPT       := -Og

PREFIX    := arm-none-eabi-
CC        := $(PREFIX)gcc
AS        := $(PREFIX)gcc -x assembler-with-cpp
OBJCOPY   := $(PREFIX)objcopy
SIZE      := $(PREFIX)size

MCU_FLAGS := -mcpu=cortex-m3 -mthumb -mfloat-abi=soft

C_DEFS    := -DUSE_HAL_DRIVER -DSTM32L152xE

C_INCLUDES := \
  -ICore/Inc \
  -IDrivers/STM32L1xx_HAL_Driver/Inc \
  -IDrivers/STM32L1xx_HAL_Driver/Inc/Legacy \
  -IDrivers/CMSIS/Device/ST/STM32L1xx/Include \
  -IDrivers/CMSIS/Include \
  -IMiddlewares/Third_Party/FreeRTOS/Source/include \
  -IMiddlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2 \
  -IMiddlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM3

C_SOURCES := \
  $(wildcard Core/Src/*.c) \
  $(wildcard Drivers/STM32L1xx_HAL_Driver/Src/*.c) \
  Middlewares/Third_Party/FreeRTOS/Source/croutine.c \
  Middlewares/Third_Party/FreeRTOS/Source/event_groups.c \
  Middlewares/Third_Party/FreeRTOS/Source/list.c \
  Middlewares/Third_Party/FreeRTOS/Source/queue.c \
  Middlewares/Third_Party/FreeRTOS/Source/stream_buffer.c \
  Middlewares/Third_Party/FreeRTOS/Source/tasks.c \
  Middlewares/Third_Party/FreeRTOS/Source/timers.c \
  Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM3/port.c \
  Middlewares/Third_Party/FreeRTOS/Source/portable/MemMang/heap_4.c \
  Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2/cmsis_os2.c

ASM_SOURCES := Core/Startup/startup_stm32l152retx.s

CFLAGS  := $(MCU_FLAGS) $(C_DEFS) $(C_INCLUDES) $(OPT) -Wall \
           -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP
ASFLAGS := $(MCU_FLAGS) $(OPT) -Wall -fdata-sections -ffunction-sections

LDSCRIPT := STM32L152RETX_FLASH.ld
LDFLAGS  := $(MCU_FLAGS) -specs=nano.specs -specs=nosys.specs \
            -T$(LDSCRIPT) -Wl,-Map=$(BUILD_DIR)/$(TARGET).map,--cref \
            -Wl,--gc-sections -lc -lm

C_OBJS   := $(addprefix $(BUILD_DIR)/, $(notdir $(C_SOURCES:.c=.o)))
ASM_OBJS := $(addprefix $(BUILD_DIR)/, $(notdir $(ASM_SOURCES:.s=.o)))
OBJS     := $(C_OBJS) $(ASM_OBJS)

vpath %.c $(sort $(dir $(C_SOURCES)))
vpath %.s $(sort $(dir $(ASM_SOURCES)))

.PHONY: all clean
all: $(BUILD_DIR)/$(TARGET).elf $(BUILD_DIR)/$(TARGET).hex $(BUILD_DIR)/$(TARGET).bin

$(BUILD_DIR)/%.o: %.c | $(BUILD_DIR)
	$(CC) -c $(CFLAGS) $< -o $@

$(BUILD_DIR)/%.o: %.s | $(BUILD_DIR)
	$(AS) -c $(ASFLAGS) $< -o $@

$(BUILD_DIR)/$(TARGET).elf: $(OBJS)
	$(CC) $(OBJS) $(LDFLAGS) -o $@
	$(SIZE) $@

$(BUILD_DIR)/%.hex: $(BUILD_DIR)/%.elf
	$(OBJCOPY) -O ihex $< $@

$(BUILD_DIR)/%.bin: $(BUILD_DIR)/%.elf
	$(OBJCOPY) -O binary -S $< $@

$(BUILD_DIR):
	mkdir -p $@

clean:
	rm -rf $(BUILD_DIR)

-include $(wildcard $(BUILD_DIR)/*.d)
