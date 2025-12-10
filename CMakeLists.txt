cmake_minimum_required(VERSION 3.23)

# Vérifier si on compile comme sous-projet
get_directory_property(HAS_PARENT PARENT_DIRECTORY)
if(HAS_PARENT)
    message("-- embedded_DIL: Building as subproject")
else()
    # Configuration du toolchain SEULEMENT si on compile standalone
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
endif()

project(embedded_DIL C CXX ASM)

# Ajouter le sous-répertoire stm32-components (submodule)
add_subdirectory(stm32-components)

# Créer la librairie avec SEULEMENT nos sources
add_library(${PROJECT_NAME} STATIC
    ${CMAKE_CURRENT_SOURCE_DIR}/Control/Src/led_flasher.c
    ${CMAKE_CURRENT_SOURCE_DIR}/Communication/Src/communicator.c
    ${CMAKE_CURRENT_SOURCE_DIR}/Telemetry/Src/telemtry_bidon.c
)

# CRITICAL: Inclure NOS répertoires avec PUBLIC pour propager aux targets qui linkent
target_include_directories(${PROJECT_NAME} PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/Communication/Inc
    ${CMAKE_CURRENT_SOURCE_DIR}/Control/Inc
    ${CMAKE_CURRENT_SOURCE_DIR}/Telemetry/Inc
)

# Définitions de compilation (si nécessaires pour cette bibliothèque uniquement)
# Les définitions communes doivent venir de stm32_components
target_compile_definitions(${PROJECT_NAME} PUBLIC
    # Ajoutez ici uniquement les defines spécifiques à embedded_DIL
    # Les defines STM32 doivent venir de stm32_components
)

# Lier avec stm32_components (qui propagera ses includes et defines)
target_link_libraries(${PROJECT_NAME} PUBLIC 
    stm32_components
)