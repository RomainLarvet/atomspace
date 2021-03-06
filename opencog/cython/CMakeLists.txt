
# Need to use -fno-strict-aliasing when compiling cython code, in order
# to avoid nasty compiler warnings about aliasing.  Cython explicitly
# performs aliasing, in order to emulate python object  inheritance.
# See, for example, https://groups.google.com/forum/#!topic/cython-users/JV1-KvIUeIg
#
# Also suppressing unused function warnings for static functions since Cython
# generates a static function that it puts in a header which causes this warning
# to appear whenever anyone includes the header and doesn't use all the functions.
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-strict-aliasing -Wno-unused-function")

ADD_SUBDIRECTORY (opencog)

INCLUDE_DIRECTORIES(
	${PYTHON_INCLUDE_DIRS}
	${CMAKE_CURRENT_SOURCE_DIR}
	${CMAKE_CURRENT_BINARY_DIR}
)

# Use this as a guide:
# https://github.com/OpenKinect/libfreenect/blob/master/wrappers/python/CMakeLists.txt



# direct file and path creation for stuff that doesn't need to be compiled
file(MAKE_DIRECTORY opencog)
# module init
file(COPY opencog/__init__.py DESTINATION opencog)
##

ADD_LIBRARY(PythonEval
	PythonEval.cc
)
ADD_DEPENDENCIES(PythonEval atomspace_cython)

TARGET_LINK_LIBRARIES(PythonEval
	atombase
	atomspace
	${PYTHON_LIBRARIES}
	${Boost_FILESYSTEM_LIBRARY}
)

INSTALL (TARGETS PythonEval DESTINATION "lib${LIB_DIR_SUFFIX}/opencog")

INSTALL (FILES
	PythonEval.h
	PyIncludeWrapper.h
	DESTINATION "include/opencog/cython"
)

INSTALL (FILES 
	opencog/__init__.py
	DESTINATION "${DATADIR}/python/opencog"
)

# ----------------------------------------------------------------
# The scheme wrapper for PythonEval. Allows python snippets to be
# called from scheme.
#
ADD_LIBRARY(PythonSCM
	PythonSCM.cc
)
ADD_DEPENDENCIES(PythonSCM PythonEval)

TARGET_LINK_LIBRARIES(PythonSCM
	PythonEval
	smob
)

INSTALL (TARGETS PythonSCM DESTINATION "lib${LIB_DIR_SUFFIX}/opencog")
