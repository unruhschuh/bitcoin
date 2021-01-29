message("I'm /src/univalue")

set(SRC
	lib/univalue.cpp
	lib/univalue_get.cpp
	lib/univalue_read.cpp
	lib/univalue_write.cpp
  )

set(TEST_DATA_DIR test)

add_library(libunivalue ${SRC})
target_include_directories(libunivalue PUBLIC include)

add_executable(unitester test/unitester.cpp)
target_link_libraries(unitester libunivalue)
target_compile_definitions(unitester PRIVATE JSON_TEST_SRC="${CMAKE_CURRENT_SOURCE_DIR}/${TEST_DATA_DIR}")
# todo: test_unitester_LDFLAGS = -static $(LIBTOOL_APP_LDFLAGS)

add_executable(test_json test/test_json.cpp)
target_link_libraries(test_json libunivalue)
# todo: test_test_json_LDFLAGS = -static $(LIBTOOL_APP_LDFLAGS)

add_executable(no_nul test/no_nul.cpp)
target_link_libraries(no_nul libunivalue)
# todo: test_no_nul_LDFLAGS = -static $(LIBTOOL_APP_LDFLAGS)

add_executable(object test/object.cpp)
target_link_libraries(object libunivalue)
# todo: test_object_LDFLAGS = -static $(LIBTOOL_APP_LDFLAGS)

add_executable(gen gen/gen.cpp)
target_include_directories(gen PRIVATE include)
# todo: consider deleting lib/univalue_escapes.h, since it is generated below
add_custom_command(
  TARGET gen
  POST_BUILD
  COMMAND gen > lib/univalue_escapes.h
  COMMENT "Updating lib/univalue_escapes.h"
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

add_test(NAME object COMMAND object)
add_test(NAME unitester COMMAND unitester)
add_test(NAME no_nul COMMAND no_nul)
