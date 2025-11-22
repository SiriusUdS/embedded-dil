cmake_minimum_required(VERSION 3.23)



# Configuration du toolchain AVANT project()
# Mais seulement si on n'est pas déjà en cross-compilation
if(NOT CMAKE_TOOLCHAIN_FILE)
    set(CMAKE_SYSTEM_NAME Generic)
    set(CMAKE_SYSTEM_PROCESSOR arm)
    
    set(CMAKE_C_COMPILER arm-none-eabi-gcc)
    set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
    set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
    
    set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
    
    set(COMMON_FLAGS "-mcpu=cortex-m7 -mthumb -mfloat-abi=hard -mfpu=fpv5-d16")
    set(CMAKE_C_FLAGS_INIT "${COMMON_FLAGS} -fdata-sections -ffunction-sections")
    set(CMAKE_CXX_FLAGS_INIT "${COMMON_FLAGS} -fdata-sections -ffunction-sections")
    set(CMAKE_ASM_FLAGS_INIT "${COMMON_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS_INIT "${COMMON_FLAGS} -specs=nano.specs -Wl,--gc-sections")
endif()

project(embedded_DIL C CXX ASM)

   add_subdirectory(
    stm32-components
   )

# Créer la librairie avec les sources
add_library(${PROJECT_NAME} STATIC

        ${CMAKE_CURRENT_SOURCE_DIR}/Control/Src/led_flasher.c
)

# Inclure les répertoires
target_include_directories(${PROJECT_NAME} PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/Communication/Inc
    ${CMAKE_CURRENT_SOURCE_DIR}/Control/Inc
    ${CMAKE_CURRENT_SOURCE_DIR}/Telemetry/Inc
)

# Lier avec le HAL approprié
target_link_libraries(${PROJECT_NAME} PUBLIC 
stm32_components)
    