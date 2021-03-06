############################################################################
# CMakeLists.txt file for building ROOT graf2d/win32gdk package
# @author Pere Mato, CERN
############################################################################
ROOT_USE_PACKAGE(gui/gui)
include_directories(${FREETYPE_INCLUDE_DIRS} ${CMAKE_CURRENT_SOURCE_DIR}/gdk/src 
					${CMAKE_CURRENT_SOURCE_DIR}/gdk/src/gdk
					${CMAKE_CURRENT_BINARY_DIR}/gdk/src/glib)

set(iconvlib  ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/iconv-1.3.lib)
set(iconvdll  ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_CFG_INTDIR}/iconv-1.3.dll)
set(iconvliba ${CMAKE_CURRENT_BINARY_DIR}/gdk/src/iconv/iconv-1.3.lib)
set(iconvdlla ${CMAKE_CURRENT_BINARY_DIR}/gdk/src/iconv/iconv-1.3.dll)

set(gliblib  ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/glib-1.3.lib)
set(glibdll  ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_CFG_INTDIR}/glib-1.3.dll)
set(glibliba ${CMAKE_CURRENT_BINARY_DIR}/gdk/src/glib/glib-1.3.lib)
set(glibdlla ${CMAKE_CURRENT_BINARY_DIR}/gdk/src/glib/glib-1.3.dll)

set(gdklib   ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/gdk-1.3.lib)
set(gdkdll   ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_CFG_INTDIR}/gdk-1.3.dll)
set(gdkliba  ${CMAKE_CURRENT_BINARY_DIR}/gdk/src/gdk/gdk-1.3.lib)
set(gdkdlla  ${CMAKE_CURRENT_BINARY_DIR}/gdk/src/gdk/gdk-1.3.dll)

if(winrtdebug)
  set(nmcxxflags "${CMAKE_CXX_FLAGS}" DEBUG=1)
else()
  set(nmcxxflags "${CMAKE_CXX_FLAGS}")
endif()

# Build iconv

add_custom_command( OUTPUT ${iconvliba} ${iconvdlla}
					COMMAND cmake -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/gdk gdk
					COMMAND echo "*** Building ${iconvliba}"
					COMMAND cmake -E chdir gdk/src/iconv 
							nmake -nologo -f makefile.msc NMCXXFLAGS=${nmcxxflags} VC_MAJOR=${VC_MAJOR}
				  )

add_custom_command( OUTPUT ${iconvlib} ${iconvdll}
					COMMAND cmake -E copy_if_different ${iconvliba} ${iconvlib}
					COMMAND cmake -E copy_if_different ${iconvdlla} ${iconvdll}
					DEPENDS ${iconvliba} ${iconvdlla}
				  )

add_custom_target(iconv DEPENDS ${iconvlib})
set_target_properties(iconv PROPERTIES FOLDER Builtins)

# Build glib

add_custom_command( OUTPUT ${glibliba} ${glibdlla}
					COMMAND echo "*** Building ${glibliba}"
					COMMAND cmake -E chdir gdk/src/glib 
							nmake -nologo -f makefile.msc NMCXXFLAGS=${nmcxxflags} VC_MAJOR=${VC_MAJOR}
					DEPENDS iconv
				  )

add_custom_command( OUTPUT ${gliblib} ${glibdll}
					COMMAND cmake -E copy_if_different ${glibliba} ${gliblib}
					COMMAND cmake -E copy_if_different ${glibdlla} ${glibdll}
					DEPENDS ${glibliba} ${glibdlla})

add_custom_target(glib DEPENDS ${gliblib})
set_target_properties(glib PROPERTIES FOLDER Builtins)

# Build gdk

add_custom_command( OUTPUT ${gdkliba}
					COMMAND echo "*** Building ${gdkliba}"
					COMMAND cmake -E chdir gdk/src/gdk/win32 
							nmake -nologo -f makefile.msc NMCXXFLAGS=${nmcxxflags} VC_MAJOR=${VC_MAJOR}
					COMMAND cmake -E chdir gdk/src/gdk 
							nmake -nologo -f makefile.msc NMCXXFLAGS=${nmcxxflags} VC_MAJOR=${VC_MAJOR}
					DEPENDS glib
				  )

add_custom_command( OUTPUT ${gdklib} ${gdkdll}
					COMMAND cmake -E copy_if_different ${gdkliba} ${gdklib}
					COMMAND cmake -E copy_if_different ${gdkdlla} ${gdkdll}
					DEPENDS ${gdkliba} )

add_custom_target(GDKLIB DEPENDS ${gdklib})
set_target_properties(GDKLIB PROPERTIES FOLDER Builtins)

#
# ROOT Dicts for the win32gdk library
#

ROOT_GENERATE_DICTIONARY(G__Win32gdk TGWin32.h TGWin32GL.h MODULE Win32gdk LINKDEF LinkDef.h)


#
# The library itself
#

ROOT_LINKER_LIBRARY(Win32gdk *.cxx *.c G__Win32gdk.cxx LIBRARIES Cint Core Graf ${FREETYPE_LIBRARIES} 
					${gdklib} ${gliblib} Glu32.lib Opengl32.lib)

add_dependencies(Win32gdk GDKLIB glib)

if(builtin_freetype)
  add_dependencies(Win32gdk FREETYPE)
endif()
					
install(FILES  ${gdkdlla}
			   ${glibdlla}
			   ${iconvdlla}
			   DESTINATION ${CMAKE_INSTALL_BINDIR})

ROOT_INSTALL_HEADERS()
