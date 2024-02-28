cmake_minimum_required(VERSION 3.24.2)

project(motor_driver LANGUAGES ASM C CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

add_executable(${CMAKE_PROJECT_NAME}
    main.cpp
    vectors_table.cpp
    reset_isr.cpp
    fault_isrs.cpp
    utils.hpp
    cortexm0plus/exceptions.hpp
    cortexm0plus/special_regs.hpp
    cortexm0plus/nvic.hpp
    cortexm0plus/scb.hpp
    cortexm0plus/systick.hpp
)

target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE
    $<$<COMPILE_LANGUAGE:ASM>:
        -mcpu=cortex-m0plus
        -mthumb
        -fstack-usage
        -Wmissing-include-dirs
        -masm-syntax-unified
    >

    $<$<COMPILE_LANGUAGE:C>:
        -mcpu=cortex-m0
        -std=gnu18
        -ffunction-sections
        -fdata-sections
        -Wall
        -Wextra
        -pedantic
        -Wmissing-include-dirs
        -Wswitch-enum
        -Wconversion
        -fstack-usage
        -nostdlib
        -mfloat-abi=soft
        -mthumb
        -ffreestanding
        -masm-syntax-unified
    >

    $<$<COMPILE_LANGUAGE:CXX>:
        -mcpu=cortex-m0
        -std=gnu++20
        -ffunction-sections
        -fdata-sections
        -fno-exceptions
        -fno-rtti
        -fno-use-cxa-atexit
        -Wall
        -Wextra
        -pedantic
        -Wmissing-include-dirs
        -Wswitch-enum
        -Wconversion
        -fstack-usage
        -nostdlib
        -mfloat-abi=soft
        -mthumb
        -ffreestanding
        -fno-threadsafe-statics
        -masm-syntax-unified
    >
)

target_link_options(${CMAKE_PROJECT_NAME} PRIVATE
    -mcpu=cortex-m0
    -T${CMAKE_SOURCE_DIR}/${CMAKE_PROJECT_NAME}.ld
    -Wl,-Map=${CMAKE_PROJECT_NAME}.map,--cref
    -nostdlib
    -Wl,--cref
    -Wl,--gc-sections
    -static
    -Wl,--print-memory-usage
    -mfloat-abi=soft
    -mthumb
    -Wl,--defsym=__PROCESS_STACK_SIZE_SYMBOL__=0x3000
)

add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_SIZE} -A ${CMAKE_PROJECT_NAME}.elf
    COMMAND ${CMAKE_OBJCOPY} --gap-fill 0 -O binary -S ${CMAKE_PROJECT_NAME}.elf ${CMAKE_PROJECT_NAME}.bin
    COMMAND ${CMAKE_OBJDUMP} -S -d -l -C -z ${CMAKE_PROJECT_NAME}.elf > ${CMAKE_PROJECT_NAME}.asm
)
