cmake_minimum_required(VERSION 2.8...3.12)

project(lua)

# set(CMAKE_C_STANDARD 99)
# set(CMAKE_CXX_STANDARD 11)

# Lua 头文件
set(LIBLUA_INCLUDE lua.h luaconf.h lualib.h lauxlib.h lua.hpp)

file(GLOB LUA_SOURCES lua.c)
file(GLOB LUAC_SOURCES luac.c)
file(GLOB LIBLUA_SOURCES l*.c)
list(REMOVE_ITEM LIBLUA_SOURCES ${LUA_SOURCES})
list(REMOVE_ITEM LIBLUA_SOURCES ${LUAC_SOURCES})

# 可选C++编译器
if (USE_CXX)
  set_source_files_properties(${LUA_SOURCES} ${LUAC_SOURCES} ${LIBLUA_SOURCES} PROPERTIES LANGUAGE CXX)
endif()

# 生成 Liblua
add_library(liblua_object OBJECT ${LIBLUA_SOURCES})

add_library(liblua STATIC $<TARGET_OBJECTS:liblua_object>)
if (WIN32)
  add_library(liblua-dll SHARED $<TARGET_OBJECTS:liblua_object>)
  target_compile_definitions(liblua-dll PRIVATE LUA_BUILD_AS_DLL LUA_COMPAT_5_3)
else()
  add_definitions(-D_GNU_SOURCE -DLUA_USE_POSIX -DLUA_USE_DLOPEN -DLUA_COMPAT_5_3 -fPIC)
  target_link_libraries(liblua PRIVATE dl PRIVATE m)
endif()

# Liblua 静态库
set_target_properties (liblua PROPERTIES PUBLIC_HEADER "${LIBLUA_INCLUDE}" OUTPUT_NAME "liblua" PREFIX "")
install (TARGETS liblua ARCHIVE DESTINATION lib PUBLIC_HEADER DESTINATION include)

# Liblua 共享库
if (WIN32)
  set_target_properties(liblua-dll PROPERTIES OUTPUT_NAME "liblua" PREFIX "")
  install(TARGETS liblua-dll LIBRARY DESTINATION lib)
endif()

# 生成 Lua 可执行文件
if (LUA_SOURCES)
  add_executable(lua ${LUA_SOURCES})
  if (USE_READLINE) # 使用 readline
    add_definitions(-DLUA_USE_READLINE)
    target_link_libraries(lua PRIVATE liblua PRIVATE readline)
  else()
    target_link_libraries(lua PRIVATE liblua)
  endif()
  install(TARGETS lua RUNTIME DESTINATION bin)
else()
  message("[Warning]: Unable to generate `lua.exe`, the `lua.c` file cannot be found.")
endif()

# 生成 Luac 可执行文件
if(LUAC_SOURCES)
  add_executable(luac ${LUAC_SOURCES})
  target_link_libraries(luac PRIVATE liblua)
  install(TARGETS luac RUNTIME DESTINATION bin)
else()
  message("[Warning]: Unable to generate `luac.exe`, the `luac.c` file cannot be found.")
endif()