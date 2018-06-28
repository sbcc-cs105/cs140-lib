enable_testing()

execute_process(COMMAND uname -s OUTPUT_VARIABLE PLATFORM OUTPUT_STRIP_TRAILING_WHITESPACE)

set(EXECUTABLE_OUTPUT_PATH "bin/")

file(GLOB_RECURSE ASSIGNMENT_SOURCES RELATIVE ${CMAKE_SOURCE_DIR} "src/*.cc")
file(GLOB_RECURSE ASSIGNMENT_INCLUDES RELATIVE ${CMAKE_SOURCE_DIR} "include/*.h")
list(REMOVE_ITEM ASSIGNMENT_SOURCES "src/${SOURCE_PREFIX}_unittest.cc")

add_executable(${EXEC_NAME} ${ASSIGNMENT_SOURCES} ${ASSIGNMENT_INCLUDES})
include_directories(include/ cs140/include)

add_library(${EXEC_NAME}Obj OBJECT ${ASSIGNMENT_SOURCES})
target_compile_options(${EXEC_NAME}Obj PRIVATE -Dmain=_main)

link_directories(cs140/lib/${PLATFORM})
link_libraries(gtestd gtest_maind pthread)
add_executable(${EXEC_NAME}_GTest src/${SOURCE_PREFIX}_unittest.cc $<TARGET_OBJECTS:${EXEC_NAME}Obj>)

add_custom_command(OUTPUT ${CMAKE_HOME_DIRECTORY}/.git/hooks/pre-push COMMAND cp ARGS githooks/pre-push .git/hooks WORKING_DIRECTORY ${CMAKE_HOME_DIRECTORY})
add_custom_target(pre-push-script ALL DEPENDS ${CMAKE_HOME_DIRECTORY}/.git/hooks/pre-push)
add_custom_command(OUTPUT ${CMAKE_HOME_DIRECTORY}/.git/hooks/pre-commit COMMAND cp ARGS githooks/pre-commit .git/hooks WORKING_DIRECTORY ${CMAKE_HOME_DIRECTORY})
add_custom_target(pre-commit-script ALL DEPENDS ${CMAKE_HOME_DIRECTORY}/.git/hooks/pre-commit)
add_custom_command(OUTPUT ${CMAKE_HOME_DIRECTORY}/.git/hooks/project.py COMMAND cp ARGS .githooks/project.py .git/hooks WORKING_DIRECTORY ${CMAKE_HOME_DIRECTORY})
add_custom_target(project-config ALL DEPENDS ${CMAKE_HOME_DIRECTORY}/.git/hooks/project.py)

add_dependencies(${EXEC_NAME}_GTest ${EXEC_NAME}Obj pre-push-script pre-commit-script project-config)
add_dependencies(${EXEC_NAME} pre-push-script pre-commit-script project-config)
