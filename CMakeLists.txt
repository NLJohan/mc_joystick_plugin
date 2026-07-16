cmake_minimum_required(VERSION 3.22)

include(cmake/get-jrl-cmakemodules.cmake)

project(
  mc_joystick_plugin
  LANGUAGES CXX
  VERSION 0.0.1)

jrl_configure_defaults()

jrl_option(AUTOLOAD_${PROJECT_NAME}_PLUGIN
           "Automatically load ${PROJECT_NAME} plugin" OFF)

# Find mc_rtc if we are not building inside mc_rtc itself
if(NOT TARGET mc_rtc::mc_control)
  jrl_find_package(mc_rtc REQUIRED)
endif()

# ##############################################################################
# joystick_inputs header-only
# ##############################################################################
add_library(joystick_inputs INTERFACE)
jrl_target_headers(
  joystick_inputs PUBLIC HEADERS include/mc_joystick_plugin/joystick_inputs.h
  BASE_DIRS include)
add_library(joystick_inputs_properties ALIAS joystick_inputs)
target_include_directories(
  joystick_inputs
  INTERFACE $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include/mc_joystick_plugin>
            $<INSTALL_INTERFACE:include/mc_joystick_plugin>)
jrl_add_export_component(NAME joystick_inputs TARGETS joystick_inputs)

# Configure and install the plugin configuration
configure_file(etc/${PROJECT_NAME}.in.yaml
               "${CMAKE_CURRENT_BINARY_DIR}/etc/${PROJECT_NAME}.yaml")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/etc/${PROJECT_NAME}.yaml"
        DESTINATION "${MC_PLUGINS_RUNTIME_INSTALL_PREFIX}/etc")

set(plugin_SRC src/plugin.cpp src/joystick/joystick.cc)
set(plugin_HDR src/plugin.h src/joystick/joystick.hh)

# The add_plugin macro is provided by mc_rtc
add_plugin(${PROJECT_NAME} "${plugin_SRC}" "${plugin_HDR}")

target_include_directories(
  ${PROJECT_NAME}
  PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>$<INSTALL_INTERFACE:include>
)
target_link_libraries(${PROJECT_NAME} PUBLIC joystick_inputs)

install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/ DESTINATION include)

# Export the package files (headers, targets, config files, etc.)
jrl_export_package()
# Print summaries
jrl_print_dependencies_summary()
jrl_print_options_summary()
