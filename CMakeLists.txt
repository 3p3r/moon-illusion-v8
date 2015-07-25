CMAKE_MINIMUM_REQUIRED( VERSION 3.0 FATAL_ERROR )
PROJECT( moon_illusion_v8 )

FIND_PACKAGE(Git REQUIRED)
INCLUDE (ExternalProject)

SET(V8_ROOT "${CMAKE_CURRENT_BINARY_DIR}/build_v8" CACHE PATH
"Where I will put final v8 sources and dependencies in.")

# **********************************************************************
# Visual representation of wtf is going on in the dependency tree:
#
#         gyp    gmock    gtest    cygwin    python_26    icu
#          |       |        |        |          |          |
#          +-------+--------+---+----+----------+----------+
#                               |
#                               v8
#                               |
#                              cv8
#                               |
#          +-------+--------+---+----+----------+----------+
#          |       |        |        |          |          |
#         cgyp   cgmock   cgtest   ccygwin   cpython_26  cicu
#          |       |        |        |          |          |
#          +-------+--------+---+----+----------+----------+
#                               |
#                       moon_illusion_v8 :D
#
# **********************************************************************

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

ADD_EXTERNAL_PROJECT(gyp       https://chromium.googlesource.com/external/gyp)
ADD_EXTERNAL_PROJECT(gmock     https://chromium.googlesource.com/external/gmock)
ADD_EXTERNAL_PROJECT(gtest     https://chromium.googlesource.com/external/gtest)
ADD_EXTERNAL_PROJECT(cygwin    https://chromium.googlesource.com/chromium/deps/cygwin)
ADD_EXTERNAL_PROJECT(python_26 https://chromium.googlesource.com/chromium/deps/python_26)
ADD_EXTERNAL_PROJECT(icu       https://chromium.googlesource.com/chromium/deps/icu)
ADD_EXTERNAL_PROJECT(v8        https://github.com/v8/v8-git-mirror)

# Configure v8:
ADD_DEPENDENCIES(v8 gyp gmock gtest cygwin python_26 icu)
EXTERNALPROJECT_GET_PROPERTY(v8 SOURCE_DIR)
ADD_CUSTOM_TARGET(cv8
COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR} ${V8_ROOT}
DEPENDS v8
)

# cgmock (Configure-gmock):
EXTERNALPROJECT_GET_PROPERTY(gmock SOURCE_DIR)
ADD_CUSTOM_TARGET(cgmock
COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR} ${V8_ROOT}/testing/gmock
DEPENDS cv8
)

# cgtest (Configure-gtest):
EXTERNALPROJECT_GET_PROPERTY(gtest SOURCE_DIR)
ADD_CUSTOM_TARGET(cgtest
COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR} ${V8_ROOT}/testing/gtest
DEPENDS cv8
)

# ccygwin (Configure-cygwin):
EXTERNALPROJECT_GET_PROPERTY(cygwin SOURCE_DIR)
ADD_CUSTOM_TARGET(ccygwin
COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR} ${V8_ROOT}/third_party/cygwin
DEPENDS cv8
)

# cpython_26 (Configure-python_26):
EXTERNALPROJECT_GET_PROPERTY(python_26 SOURCE_DIR)
ADD_CUSTOM_TARGET(cpython_26
COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR} ${V8_ROOT}/third_party/python_26
DEPENDS cv8
)

# cicu (Configure-icu):
EXTERNALPROJECT_GET_PROPERTY(icu SOURCE_DIR)
ADD_CUSTOM_TARGET(cicu
COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR} ${V8_ROOT}/third_party/icu
DEPENDS cv8
)

# cgyp (Configure-gyp):
EXTERNALPROJECT_GET_PROPERTY(gyp SOURCE_DIR)
ADD_CUSTOM_TARGET(cgyp
COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR} ${V8_ROOT}/third_party/gyp
DEPENDS cv8
)

# igyp (Install-gyp):
ADD_CUSTOM_TARGET(igyp
WORKING_DIRECTORY ${V8_ROOT}/third_party/gyp
COMMAND setup.py install
DEPENDS cgyp
)

ADD_CUSTOM_TARGET(moon_illusion_v8
DEPENDS cgmock cgtest ccygwin cpython_26 cicu igyp
COMMAND ${SOURCE_DIR} ${V8_ROOT}/third_party/python_26/python build/gyp_v8
)
