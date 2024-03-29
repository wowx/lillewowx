#
# This file is part of the CMaNGOS Project. See AUTHORS file for Copyright information
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
#

set(EXECUTABLE_NAME ${CMANGOS_BINARY_SERVER_NAME})

set(EXECUTABLE_SRCS
    CliRunnable.cpp
    CliRunnable.h
    MaNGOSsoap.cpp
    MaNGOSsoap.h
    Main.cpp
    Master.cpp
    Master.h
    RASocket.cpp
    RASocket.h
    WorldRunnable.cpp
    WorldRunnable.h
    soapC.cpp
    soapH.h
    soapServer.cpp
    soapStub.h
   )

if(WIN32)
  # add resource file to windows build
  set(EXECUTABLE_SRCS ${EXECUTABLE_SRCS} mangosd.rc)
endif()

add_executable(${EXECUTABLE_NAME}
  ${EXECUTABLE_SRCS}
)

target_include_directories(${EXECUTABLE_NAME}
  INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
  PRIVATE ${OPENSSL_INCLUDE_DIR}
  PRIVATE ${Boost_INCLUDE_DIRS}
)

target_link_libraries(${EXECUTABLE_NAME}
  shared
  game
  gsoap
)

if(WIN32)
  target_link_libraries(${EXECUTABLE_NAME}
    optimized ${MYSQL_LIBRARY}
    optimized ${OPENSSL_LIBRARIES}
    debug ${MYSQL_DEBUG_LIBRARY}
    debug ${OPENSSL_DEBUG_LIBRARIES}
    ${Boost_LIBRARIES}
  )
  if(MINGW)
    target_link_libraries(${EXECUTABLE_NAME}
      wsock32
      ws2_32
    )
  endif()

  # Define OutDir to source/bin/(platform)_(configuaration) folder.
  set_target_properties(${EXECUTABLE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_DEBUG "${DEV_BIN_DIR}")
  set_target_properties(${EXECUTABLE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELEASE "${DEV_BIN_DIR}")
  set_target_properties(${EXECUTABLE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO "${DEV_BIN_DIR}")
  set_target_properties(${EXECUTABLE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL "${DEV_BIN_DIR}")
  set_target_properties(${EXECUTABLE_NAME} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${DEV_BIN_DIR}")
  set_target_properties(${EXECUTABLE_NAME} PROPERTIES PROJECT_LABEL "WorldServer")

  # Add provided dependency lib to dev folder
  string(REGEX REPLACE "/" "\\\\" LibFolder ${DEV_PROVIDED_LIBS_FOLDER})
  add_custom_command(TARGET ${EXECUTABLE_NAME} POST_BUILD
    COMMAND copy \"${LibFolder}\\*.dll\" \"$(OutDir)\")

  # Add conf file to dev folder
  add_custom_command(TARGET ${EXECUTABLE_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different \"${CMAKE_CURRENT_SOURCE_DIR}/mangosd.conf.dist.in\" \"$<TARGET_FILE_DIR:${EXECUTABLE_NAME}>/mangosd.conf.dist\")
endif()

if(UNIX)
  target_link_libraries(${EXECUTABLE_NAME}
    ${OPENSSL_LIBRARIES}
    ${OPENSSL_EXTRA_LIBRARIES}
    ${Boost_LIBRARIES}
  )

  if(POSTGRESQL AND POSTGRESQL_FOUND)
    target_link_libraries(${EXECUTABLE_NAME} ${PostgreSQL_LIBRARIES})
  else()
    target_link_libraries(${EXECUTABLE_NAME} ${MYSQL_LIBRARY})
  endif()
endif()

set(EXECUTABLE_LINK_FLAGS "")

if(UNIX)
  if (APPLE)
    set(EXECUTABLE_LINK_FLAGS "-pthread -framework Carbon")
  else()
    set(EXECUTABLE_LINK_FLAGS "-pthread -rdynamic")
  endif()
endif()

set_target_properties(${EXECUTABLE_NAME} PROPERTIES LINK_FLAGS
  "${EXECUTABLE_LINK_FLAGS}"
)

if (NOT WIN32)
  install(FILES run-mangosd DESTINATION ${BIN_DIR})
endif()
install(TARGETS ${EXECUTABLE_NAME} DESTINATION ${BIN_DIR})
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/mangosd.conf.dist.in ${CMAKE_CURRENT_BINARY_DIR}/mangosd.conf.dist)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/mangosd.conf.dist DESTINATION ${CONF_DIR})

if(MSVC)
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/\${BUILD_TYPE}/${EXECUTABLE_NAME}.pdb DESTINATION ${BIN_DIR} CONFIGURATIONS Debug)
endif()

add_definitions( -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} )
