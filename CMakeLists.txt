CMAKE_MINIMUM_REQUIRED( VERSION 3.0 FATAL_ERROR )
PROJECT( moon_illusion_v8 )

FIND_PACKAGE(Git REQUIRED)
INCLUDE (ExternalProject)

# This "could" be the same as install dir but this is more convenient
SET(V8_ROOT "${CMAKE_CURRENT_BINARY_DIR}/build_v8" CACHE PATH
"Where I will put final v8 sources and dependencies in.")

# Everything that will be passed to gyp as command line arguments
SET(V8_ARGS "-Dv8_enable_i18n_support=0 -Dcomponent=static_library -Dv8_use_snapshot='false'" CACHE STRING
"What I will pass to GYP before it generates v8 project files.")

# A utility macro that adds GIT url as an external project
MACRO(ADD_EXTERNAL_PROJECT name git_url)
EXTERNALPROJECT_ADD(
    ${name}
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/${name}
    GIT_REPOSITORY ${git_url}
    TIMEOUT 10
    UPDATE_COMMAND ${GIT_EXECUTABLE} pull
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
    LOG_DOWNLOAD 1
    LOG_UPDATE 1
)
ENDMACRO()

# Add everything that needs to be downloaded
ADD_EXTERNAL_PROJECT(gyp       https://chromium.googlesource.com/external/gyp)
ADD_EXTERNAL_PROJECT(gmock     https://chromium.googlesource.com/external/gmock)
ADD_EXTERNAL_PROJECT(gtest     https://chromium.googlesource.com/external/gtest)
ADD_EXTERNAL_PROJECT(cygwin    https://chromium.googlesource.com/chromium/deps/cygwin)
ADD_EXTERNAL_PROJECT(python_26 https://chromium.googlesource.com/chromium/deps/python_26)
ADD_EXTERNAL_PROJECT(icu       https://chromium.googlesource.com/chromium/deps/icu)
ADD_EXTERNAL_PROJECT(v8        https://github.com/v8/v8-git-mirror)

# v8-copy-sources copies all v8 sources to V8_ROOT.
EXTERNALPROJECT_ADD_STEP(v8 v8-copy-sources
COMMAND ${CMAKE_COMMAND} -E copy_directory <SOURCE_DIR> ${V8_ROOT}
COMMENT "v8 base sources are copied to ${V8_ROOT}."
DEPENDERS build DEPENDEES download)

# gmock-copy-sources copies all gmock sources to V8_ROOT/testing/gmock.
EXTERNALPROJECT_ADD_STEP(gmock gmock-copy-sources
COMMAND ${CMAKE_COMMAND} -E copy_directory <SOURCE_DIR> ${V8_ROOT}/testing/gmock
COMMENT "gmock base sources are copied to ${V8_ROOT}."
DEPENDERS build DEPENDEES download
DEPENDS v8)

# gtest-copy-sources copies all gtest sources to V8_ROOT/testing/gtest.
EXTERNALPROJECT_ADD_STEP(gtest gtest-copy-sources
COMMAND ${CMAKE_COMMAND} -E copy_directory <SOURCE_DIR> ${V8_ROOT}/testing/gtest
COMMENT "gtest base sources are copied to ${V8_ROOT}."
DEPENDERS build DEPENDEES download
DEPENDS v8)

# cygwin-copy-sources copies all cygwin sources to V8_ROOT/third_party/cygwin.
EXTERNALPROJECT_ADD_STEP(cygwin cygwin-copy-sources
COMMAND ${CMAKE_COMMAND} -E copy_directory <SOURCE_DIR> ${V8_ROOT}/third_party/cygwin
COMMENT "cygwin base sources are copied to ${V8_ROOT}."
DEPENDERS build DEPENDEES download
DEPENDS v8)

# python_26-copy-sources copies all python_26 sources to V8_ROOT/third_party/python_26.
EXTERNALPROJECT_ADD_STEP(python_26 python_26-copy-sources
COMMAND ${CMAKE_COMMAND} -E copy_directory <SOURCE_DIR> ${V8_ROOT}/third_party/python_26
COMMENT "python_26 base sources are copied to ${V8_ROOT}."
DEPENDERS build DEPENDEES download
DEPENDS v8)

# icu-copy-sources copies all icu sources to V8_ROOT/third_party/icu.
EXTERNALPROJECT_ADD_STEP(icu icu-copy-sources
COMMAND ${CMAKE_COMMAND} -E copy_directory <SOURCE_DIR> ${V8_ROOT}/third_party/icu
COMMENT "icu base sources are copied to ${V8_ROOT}."
DEPENDERS build DEPENDEES download
DEPENDS v8)

# gyp-copy-sources copies all gyp sources to V8_ROOT/third_party/gyp.
EXTERNALPROJECT_ADD_STEP(gyp gyp-copy-sources
COMMAND ${CMAKE_COMMAND} -E copy_directory <SOURCE_DIR> ${V8_ROOT}/third_party/gyp
COMMENT "gyp base sources are copied to ${V8_ROOT}."
DEPENDERS build DEPENDEES download
DEPENDS v8)

# gyp-install invokes gyp to install it.
EXTERNALPROJECT_ADD_STEP(gyp gyp-install
COMMAND setup.py install
WORKING_DIRECTORY ${V8_ROOT}/third_party/gyp
DEPENDERS build DEPENDEES gyp-copy-sources
DEPENDS v8)

# This target is the last one and actually runs gyp
ADD_CUSTOM_TARGET(moon_illusion_v8 ALL
DEPENDS
    gyp
    gmock
    gtest
    cygwin
    python_26
    icu
WORKING_DIRECTORY ${V8_ROOT}
COMMAND python build/gyp_v8 ${V8_ARGS}
COMMENT "You can grab v8 project files in: '${V8_ROOT}/tools/gyp'"
VERBATIM)
