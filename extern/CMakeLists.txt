include(FetchContent)

if (NOT EMSCRIPTEN)
  if (NOT TARGET dawn_native)
    message(STATUS "aurora: Building Dawn")

    set(DAWN_ENABLE_DESKTOP_GL OFF CACHE INTERNAL "Enable compilation of the OpenGL backend")
    set(DAWN_ENABLE_OPENGLES OFF CACHE INTERNAL "Enable compilation of the OpenGL ES backend")
    set(DAWN_FETCH_DEPENDENCIES ON CACHE INTERNAL "Use fetch_dawn_dependencies.py as an alternative to using depot_tools")

    FetchContent_Declare(dawn
      URL https://github.com/google/dawn/archive/65ce958ed0019a860238703efa0087660c4247d4.tar.gz
      URL_HASH SHA256=bd7df417fffd9c3efa390ce09b7e943887d76807e0b7307e192c835c38882ec1
      DOWNLOAD_EXTRACT_TIMESTAMP TRUE
      EXCLUDE_FROM_ALL
    )
    FetchContent_MakeAvailable(dawn)
    if (NOT TARGET dawn_native)
      message(FATAL_ERROR "Failed to make dawn available")
    endif ()
  else ()
    message(STATUS "aurora: Using existing Dawn")
  endif ()

  if (NOT TARGET SDL3-static)
    message(STATUS "aurora: Building SDL3")

    if (WIN32)
      set(SDL_LIBC ON CACHE BOOL "Use the system C library" FORCE)
    endif ()

    FetchContent_Declare(SDL
      URL https://github.com/libsdl-org/SDL/archive/refs/tags/release-3.2.10.tar.gz
      URL_HASH SHA256=8a6ddc3e70de897f7921da275ac71e375de89dad802aa2932649d1fea9068478
      DOWNLOAD_EXTRACT_TIMESTAMP TRUE
      EXCLUDE_FROM_ALL
    )
    FetchContent_MakeAvailable(SDL)
    if (NOT TARGET SDL3-static)
      message(FATAL_ERROR "Failed to make SDL3 available")
    endif ()
  else ()
    message(STATUS "aurora: Using existing SDL3")
  endif ()
else ()
  set(ABSL_PROPAGATE_CXX_STD ON)
  add_subdirectory(dawn/third_party/abseil-cpp EXCLUDE_FROM_ALL)
endif ()

if (NOT TARGET xxhash)
  message(STATUS "aurora: Building xxhash")
  FetchContent_Declare(xxhash
    URL https://github.com/Cyan4973/xxHash/archive/refs/tags/v0.8.3.tar.gz
    URL_HASH SHA256=aae608dfe8213dfd05d909a57718ef82f30722c392344583d3f39050c7f29a80
    SOURCE_SUBDIR cmake_unofficial
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
    EXCLUDE_FROM_ALL
  )
  set(XXHASH_BUILD_XXHSUM OFF CACHE INTERNAL "Build the xxhsum binary")
  FetchContent_MakeAvailable(xxhash)
  if (NOT TARGET xxhash)
    message(FATAL_ERROR "Failed to make xxhash available")
  endif ()
else ()
  message(STATUS "aurora: Using existing xxhash")
endif ()

if (NOT TARGET fmt)
  message(STATUS "aurora: Building fmt")
  FetchContent_Declare(fmt
    URL https://github.com/fmtlib/fmt/archive/refs/tags/11.1.4.tar.gz
    URL_HASH SHA256=ac366b7b4c2e9f0dde63a59b3feb5ee59b67974b14ee5dc9ea8ad78aa2c1ee1e
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
    EXCLUDE_FROM_ALL
  )
  FetchContent_MakeAvailable(fmt)
  if (NOT TARGET fmt)
    message(FATAL_ERROR "Failed to make fmt available")
  endif ()
else ()
  message(STATUS "aurora: Using existing fmt")
endif ()

if (NOT TARGET imgui)
  message(STATUS "aurora: Building imgui")
  FetchContent_Declare(imgui
    URL https://github.com/ocornut/imgui/archive/refs/tags/v1.91.9b-docking.tar.gz
    URL_HASH SHA256=466fdef9b18de15f0bb6e288e3d00ffa3d82200ec458ce5e4f724a161d9528a5
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
    EXCLUDE_FROM_ALL
  )
  FetchContent_MakeAvailable(imgui)

  add_library(imgui STATIC
    ${imgui_SOURCE_DIR}/imgui.cpp
    ${imgui_SOURCE_DIR}/imgui_demo.cpp
    ${imgui_SOURCE_DIR}/imgui_draw.cpp
    ${imgui_SOURCE_DIR}/imgui_tables.cpp
    ${imgui_SOURCE_DIR}/imgui_widgets.cpp
    ${imgui_SOURCE_DIR}/misc/cpp/imgui_stdlib.cpp
  )
  target_include_directories(imgui PUBLIC ${imgui_SOURCE_DIR})

  add_library(imgui_backends STATIC
    ${imgui_SOURCE_DIR}/backends/imgui_impl_sdl3.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_sdlrenderer3.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_wgpu.cpp
  )
  target_compile_definitions(imgui_backends PRIVATE IMGUI_IMPL_WEBGPU_BACKEND_DAWN)
  target_link_libraries(imgui_backends PRIVATE imgui SDL3::SDL3-static dawn::dawn_native)
  target_link_libraries(imgui PUBLIC imgui_backends)

  # Optional, replaces stb_freetype if available
  find_package(Freetype)
  # Permit disabling for macOS universal builds
  option(IMGUI_USE_FREETYPE "Enable freetype with imgui" ON)
  if (FREETYPE_FOUND AND IMGUI_USE_FREETYPE)
    target_sources(imgui PRIVATE ${imgui_SOURCE_DIR}/misc/freetype/imgui_freetype.cpp)
    target_compile_definitions(imgui PRIVATE IMGUI_ENABLE_FREETYPE)
    target_link_libraries(imgui PRIVATE Freetype::Freetype)
  endif ()
else ()
  message(STATUS "aurora: Using existing imgui")
endif ()
