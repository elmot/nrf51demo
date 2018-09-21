SET(CMAKE_SYSTEM_NAME Generic)
SET(CMAKE_SYSTEM_VERSION 1)
cmake_minimum_required(VERSION 3.7)

# specify the cross compiler
SET(CMAKE_C_COMPILER_WORKS 1)
SET(CMAKE_C_COMPILER arm-none-eabi-gcc)
SET(CMAKE_CXX_COMPILER_WORKS 1)
SET(CMAKE_CXX_COMPILER arm-none-eabi-g++)
set(AS arm-none-eabi-as)
set(AR arm-none-eabi-ar)
set(OBJCOPY arm-none-eabi-objcopy)
set(OBJDUMP arm-none-eabi-objdump)
set(SIZE arm-none-eabi-size)


SET(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/ble_app_beacon_gcc_nrf51.ld)

SET(COMMON_FLAGS "-mcpu=cortex-m0 -mthumb -mabi=aapcs --std=gnu99 -Wall -Werror -O3 -mfloat-abi=soft -ffunction-sections -fdata-sections -fno-strict-aliasing -fno-builtin --short-enums")

SET(CMAKE_CXX_FLAGS_INIT "${COMMON_FLAGS} -std=c++11")
SET(CMAKE_C_FLAGS_INIT "${COMMON_FLAGS} -std=gnu99")
SET(CMAKE_EXE_LINKER_FLAGS_INIT "-Wl,-gc-sections,--print-memory-usage -T ${LINKER_SCRIPT}")

PROJECT(ble_app_beacon_s130_pca10028 C CXX ASM)
set(CMAKE_CXX_STANDARD 11)

add_definitions(-DBOARD_PCA10028 -DSOFTDEVICE_PRESENT -DNRF51 -DS130 -DBLE_STACK_SUPPORT_REQD -DSWI_DISABLE0 )

SET(SDK_ROOT ${CMAKE_SOURCE_DIR}/nrf51sdk)

file(GLOB SOURCES
        src/main.c
        ${SDK_ROOT}/components/toolchain/gcc/gcc_startup_nrf51.s
        ${SDK_ROOT}/components/libraries/button/app_button.c
        ${SDK_ROOT}/components/libraries/util/app_error.c
        ${SDK_ROOT}/components/libraries/fifo/app_fifo.c
        ${SDK_ROOT}/components/libraries/timer/app_timer.c
        ${SDK_ROOT}/components/libraries/util/nrf_assert.c
        ${SDK_ROOT}/components/libraries/uart/app_uart_fifo.c
        ${SDK_ROOT}/components/drivers_nrf/delay/nrf_delay.c
        ${SDK_ROOT}/components/drivers_nrf/common/nrf_drv_common.c
        ${SDK_ROOT}/components/drivers_nrf/gpiote/nrf_drv_gpiote.c
        ${SDK_ROOT}/components/drivers_nrf/uart/nrf_drv_uart.c
        ${SDK_ROOT}/examples/bsp/bsp.c
        ${SDK_ROOT}/components/ble/common/ble_advdata.c
        ${SDK_ROOT}/components/ble/common/ble_conn_params.c
        ${SDK_ROOT}/components/ble/common/ble_srv_common.c
        ${SDK_ROOT}/components/toolchain/system_nrf51.c
        ${SDK_ROOT}/components/softdevice/common/softdevice_handler/softdevice_handler.c

        )

include_directories(
        config
        config/ble_app_beacon_s130_pca10028
        ${SDK_ROOT}/examples/bsp
        ${SDK_ROOT}/components/libraries/util
        ${SDK_ROOT}/components/toolchain/gcc
        ${SDK_ROOT}/components/toolchain
        ${SDK_ROOT}/components/ble/common
        ${SDK_ROOT}/components/drivers_nrf/config
        ${SDK_ROOT}/components/drivers_nrf/common
        ${SDK_ROOT}/components/libraries/fifo
        ${SDK_ROOT}/components/drivers_nrf/gpiote
        ${SDK_ROOT}/components/drivers_nrf/uart
        ${SDK_ROOT}/components/libraries/uart
        ${SDK_ROOT}/components/device
        ${SDK_ROOT}/components/softdevice/common/softdevice_handler
        ${SDK_ROOT}/components/drivers_nrf/delay
        ${SDK_ROOT}/components/libraries/timer
        ${SDK_ROOT}/components/drivers_nrf/hal
        ${SDK_ROOT}/components/libraries/button
        ${SDK_ROOT}/components/softdevice/s130/headers
)

add_executable(${PROJECT_NAME}.elf ${SOURCES} ${LINKER_SCRIPT})
set_target_properties(${PROJECT_NAME}.elf PROPERTIES LINKER_LANGUAGE CXX)

set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-Map=${PROJECT_BINARY_DIR}/${PROJECT_NAME}.map -L ${SDK_ROOT}/components/toolchain/gcc")

set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)

add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE}
        COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE}
        COMMENT "Building ${HEX_FILE}
Building ${BIN_FILE}")

add_custom_target(UPLOAD
        arm-none-eabi-gdb -iex "target remote tcp:127.0.0.1:3333"
        -iex "monitor program $<TARGET_FILE:${PROJECT_NAME}.elf>"
        -iex "monitor reset init"
        -iex "disconnect" -iex "quit")




# FROM CMAKE
#[[
# keep every function in separate section. This will allow linker to dump unused functions
LDFLAGS += -mthumb -mabi=aapcs -L $(TEMPLATE_PATH) -T$(LINKER_SCRIPT)
# let linker to dump unused sections
LDFLAGS += -Wl,--gc-sections
# use newlib in nano version
LDFLAGS += --specs=nano.specs -lc -lnosys

# Assembler flags
ASMFLAGS += -x assembler-with-cpp
ASMFLAGS += -DBOARD_PCA10028
ASMFLAGS += -DSOFTDEVICE_PRESENT
ASMFLAGS += -DNRF51
ASMFLAGS += -DS130
ASMFLAGS += -DBLE_STACK_SUPPORT_REQD
ASMFLAGS += -DSWI_DISABLE0
]]
