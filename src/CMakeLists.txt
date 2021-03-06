# Copyright 2011, Florent Lamiraux, CNRS
#
# This file is part of dynamic-graph-corba.
# dynamic-graph-corba is free software: you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public License
# as published by the Free Software Foundation, either version 3 of
# the License, or (at your option) any later version.
#
# dynamic-graph-corba is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# General Lesser Public License for more details.  You should have
# received a copy of the GNU Lesser General Public License along with
# dynamic-graph-corba. If not, see <http://www.gnu.org/licenses/>.

include(../cmake/idl.cmake)
include(../cmake/python.cmake)
FINDPYTHON()

INCLUDE_DIRECTORIES(${CMAKE_BINARY_DIR}/src)
GENERATE_IDL_FILE(interpreter ${CMAKE_SOURCE_DIR}/share/idl/dynamic-graph)
GENERATE_IDL_FILE(corba-signal ${CMAKE_SOURCE_DIR}/share/idl/dynamic-graph)

SET(LIBRARY_NAME ${PROJECT_NAME})
ADD_LIBRARY(${LIBRARY_NAME}
  SHARED
  interpreter.cc
  interpreter.impl.hh
  interpreter.impl.cc
  interpreterSK.cc
  ${CMAKE_CURRENT_BINARY_DIR}/dynamic_graph/corba/interpreter_idl.py
)

ADD_LIBRARY(corba-server
  SHARED
  corba-server.cc
  corba-server.hh
  corba-signal.impl.hh
  corba-signal.impl.cc
  corba-signal.hh
  corba-signalSK.cc
  )

PKG_CONFIG_USE_DEPENDENCY(${LIBRARY_NAME} dynamic-graph-python)
PKG_CONFIG_USE_DEPENDENCY(${LIBRARY_NAME} hpp-template-corba)

PKG_CONFIG_USE_DEPENDENCY(corba-server dynamic-graph-python)
PKG_CONFIG_USE_DEPENDENCY(corba-server hpp-template-corba)

INSTALL(TARGETS ${LIBRARY_NAME} corba-server DESTINATION ${CMAKE_INSTALL_LIBDIR})

DYNAMIC_GRAPH_PYTHON_MODULE("corba" ${LIBRARY_NAME} corba-wrap)

DYNAMIC_GRAPH_PYTHON_MODULE("corba_server" corba-server corba_server-wrap)

# Install server and remote interpreter
FIND_PROGRAM(OMNIIDL omniidl)
IF(${OMNIIDL} STREQUAL OMNIIDL-NOTFOUND)
  MESSAGE(FATAL_ERROR "cannot find omniidl.")
ENDIF(${OMNIIDL} STREQUAL OMNIIDL-NOTFOUND)

SET(EXECUTABLE_NAME dg-server)
ADD_EXECUTABLE(${EXECUTABLE_NAME} dg-server.cc)
TARGET_LINK_LIBRARIES(${EXECUTABLE_NAME} ${LIBRARY_NAME})
ADD_REQUIRED_DEPENDENCY("omniORB4 >= 4")
PKG_CONFIG_USE_DEPENDENCY(${EXECUTABLE_NAME} omniORB4)
INSTALL(TARGETS dg-server DESTINATION bin)
INSTALL(PROGRAMS dg-remote DESTINATION bin)
#Install python stubs
ADD_CUSTOM_COMMAND(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/dynamic_graph/corba/interpreter_idl.py
  COMMAND ${OMNIIDL} 
  ARGS -bpython -Wbpackage="dynamic_graph.corba"
  ${CMAKE_SOURCE_DIR}/share/idl/dynamic-graph/interpreter.idl
  MAIN_DEPENDENCY ${CMAKE_SOURCE_DIR}/share/idl/dynamic-graph/interpreter.idl
  )
SET_SOURCE_FILES_PROPERTIES(
  ${CMAKE_CURRENT_BINARY_DIR}/dynamic_graph/corba/interpreter_idl.py
  GENERATED=ON)
INSTALL(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/dynamic_graph/corba/interpreter_idl.py
  DESTINATION ${PYTHON_SITELIB}/dynamic_graph/corba
  )
INSTALL(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/dynamic_graph/corba/dynamicGraph/__init__.py
  DESTINATION ${PYTHON_SITELIB}/dynamic_graph/corba/dynamicGraph
  )
INSTALL(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/dynamic_graph/corba/dynamicGraph__POA/__init__.py
  DESTINATION ${PYTHON_SITELIB}/dynamic_graph/corba/dynamicGraph__POA
  )
