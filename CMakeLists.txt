# 项目名称
PROJECT(Lua C)

# CMAKE最小版本号
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

# 忽略BUILD目录的rpath
set(CMAKE_SKIP_BUILD_RPATH true)

# 关闭MacOSX的RPATH
set(CMAKE_MACOSX_RPATH 0)

# 设置CFLAGS
set(CMAKE_C_FLAGS "-O2 -Wall -Wextra -fPIC -DLUA_COMPAT_5_3 -DLUA_USE_DLOPEN -DLUA_USE_READLINE -DLUA_USE_POSIX")

# 头文件
set (LUA_HEADERS
  lua.h
  luaconf.h
  lualib.h
  lauxlib.h
)

# 依赖库
set (LUALIB
  lapi.c
  lcode.c
  lctype.c
  ldebug.c
  ldo.c
  ldump.c
  lfunc.c
  lgc.c
  llex.c
  lmem.c
  lobject.c
  lopcodes.c
  lparser.c
  lstate.c
  lstring.c
  ltable.c
  ltm.c
  lundump.c
  lvm.c
  lzio.c
  lauxlib.c
  lbaselib.c
  lcorolib.c
  ldblib.c
  liolib.c
  lmathlib.c
  loadlib.c
  loslib.c
  lstrlib.c
  ltablib.c
  lutf8lib.c
  linit.c
)

# lua解释器
set (LUA lua.c)

# lua编译器
set (LUAC luac.c)

# ======== 编译lua静态库 ========
ADD_LIBRARY(LUA_STATIC STATIC ${LUALIB})
TARGET_LINK_LIBRARIES( LUA_STATIC -ldl -lm -lreadline )
SET_TARGET_PROPERTIES ( LUA_STATIC PROPERTIES OUTPUT_NAME "lua" )
# ======== 编译核心库 ========

# ======== 编译lua动态库 ========
ADD_LIBRARY(LUA_DYLIB SHARED ${LUALIB})
TARGET_LINK_LIBRARIES(LUA_DYLIB -ldl -lm -lreadline )
SET_TARGET_PROPERTIES (LUA_DYLIB PROPERTIES OUTPUT_NAME "lua" )
# ======== 编译核心库 ========

# ======== 编译luac编译器 ========
add_executable(luac ${LUAC})
target_link_libraries(luac LUA_DYLIB)
# ======== 编译内置luaclib库 ========

# ======== 编译Lua解释器 ========
add_executable(lua ${LUA})
target_link_libraries(lua LUA_DYLIB)
# ======== 编译内置luaclib库 ========
