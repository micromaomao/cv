cmake_minimum_required(VERSION 3.0)
project(cv LANGUAGES NONE)

set(cv_icon_depends "")

function(add_tex)
  set(args "${ARGN}")
  list(GET args 0 main_file)
  list(LENGTH args nb_args)
  set(other_byproducts "")
  set(dependent_files "")
  if(nb_args GREATER 1)
    list(SUBLIST args 1 -1 dependent_files)
    foreach(dep ${dependent_files} "${main_file}")
      get_filename_component(dep_name "${dep}" NAME_WE)
      get_filename_component(dep_ext "${dep}" EXT)
      if("${dep_ext}" STREQUAL ".tex")
        list(APPEND other_byproducts "${dep_name}.aux" "${dep_name}.out" "${dep_name}.log")
      endif()
    endforeach()
  else()
    set(dependent_files "")
  endif()
  get_filename_component(main_name "${main_file}" NAME_WE)
  add_custom_command(
    OUTPUT "${main_name}.pdf"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    COMMAND ${CMAKE_COMMAND} -Dmain_name="${main_name}" -P "${CMAKE_CURRENT_SOURCE_DIR}/build_till_no_change.cmake"
    DEPENDS "${main_file}" ${dependent_files}
    BYPRODUCTS ${other_byproducts}
  )
  add_custom_target("${main_name}" ALL DEPENDS "${main_name}.pdf")
endfunction()

function(add_svg_to_tex svg_file)
  get_filename_component(svg_dir "${svg_file}" DIRECTORY)
  get_filename_component(svg_name "${svg_file}" NAME_WE)
  add_custom_command(
    OUTPUT "${svg_dir}/${svg_name}.pdf_tex"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    COMMAND inkscape "${svg_dir}/${svg_name}.svg" --export-pdf="${svg_dir}/${svg_name}.pdf" --export-latex
    DEPENDS "${svg_file}"
    BYPRODUCTS "${svg_dir}/${svg_name}.pdf"
  )
  set(cv_icon_depends ${cv_icon_depends} "${svg_dir}/${svg_name}.pdf_tex" PARENT_SCOPE)
endfunction()

add_svg_to_tex(icons/ucl.svg)
add_svg_to_tex(icons/github.svg)
add_svg_to_tex(icons/tfl.svg)
add_svg_to_tex(icons/bluegithub.svg)
add_tex(cv.tex icons.tex styles.tex ${cv_icon_depends})
add_custom_target(clean-all
  COMMAND make clean && rm -rf Makefile CMakeFiles CMakeCache.txt cmake_install.cmake *.fdb_latexmk *.fls *.synctex.gz)
