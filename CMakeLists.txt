# The following lines of boilerplate have to be in your project's CMakeLists
# in this exact order for cmake to work correctly
cmake_minimum_required(VERSION 3.16)


include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(blink_mqtt_ota)

target_add_binary_data(blink_mqtt_ota.elf "emqxsl-ca.crt" TEXT)
