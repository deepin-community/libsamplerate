find_package(ALSA)
set(HAVE_ALSA ${ALSA_FOUND} PARENT_SCOPE)
# ALSA::ALSA target is exported since CMake >= 3.12, create it for
# old CMake versions
if(ALSA_FOUND)
  if(NOT TARGET ALSA::ALSA)
    add_library(ALSA::ALSA UNKNOWN IMPORTED)
    set_target_properties(ALSA::ALSA PROPERTIES
    INTERFACE_INCLUDE_DIRECTORIES "${ALSA_INCLUDE_DIRS}"
    IMPORTED_LOCATION "${ALSA_LIBRARIES}")
  endif()
endif()

add_executable(timewarp-file timewarp-file.c)
target_link_libraries(timewarp-file
  PRIVATE
    samplerate
    $<$<BOOL:${SndFile_FOUND}>:${SNDFILE_TARGET}>)

add_executable(varispeed-play varispeed-play.c audio_out.c audio_out.h)
target_link_libraries(varispeed-play
  PRIVATE
    samplerate
    $<$<BOOL:${SndFile_FOUND}>:${SNDFILE_TARGET}>)
if(WIN32)
  target_link_libraries(varispeed-play PRIVATE winmm)
elseif(APPLE)
  target_link_libraries(varispeed-play PRIVATE "-framework CoreAudio")
elseif(ALSA_FOUND)
  target_link_libraries(varispeed-play PRIVATE ALSA::ALSA)
endif()
