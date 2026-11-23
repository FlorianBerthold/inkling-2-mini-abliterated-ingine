# ingine_text: tokenizer, Jinja adapter, ChatFormat, StreamParser (CUDA-free).
add_library(ingine_text STATIC
  unicode.cpp
  tokenizer.cpp
  pyjson.cpp
  jinja.cpp
  chat.cpp
)
ingine_internal_includes(ingine_text)
target_include_directories(ingine_text PUBLIC ${PROJECT_SOURCE_DIR}/src)
target_link_libraries(ingine_text PUBLIC nlohmann_json::nlohmann_json PRIVATE ingine_jinja ingine_utf8proc)

# Optional toks backend for the ordinary BPE spans (https://github.com/actual-computer/toks). The native encoder
# stays the fallback for every file toks rejects and for every call that fails; ids are identical either way.
option(INGINE_WITH_TOKS "Encode ordinary BPE spans with toks" OFF)
set(TOKS_ROOT "" CACHE PATH "toks checkout (include/toks.h and the built static library)")
if(INGINE_WITH_TOKS)
  find_path(TOKS_INCLUDE_DIR toks.h HINTS ${TOKS_ROOT}/include ${TOKS_ROOT} REQUIRED)
  find_library(TOKS_LIBRARY NAMES toks HINTS ${TOKS_ROOT}/build/linux-x86_64 ${TOKS_ROOT}/lib ${TOKS_ROOT}/build
               REQUIRED)
  target_compile_definitions(ingine_text PRIVATE INGINE_WITH_TOKS=1)
  target_include_directories(ingine_text PRIVATE ${TOKS_INCLUDE_DIR})
  target_link_libraries(ingine_text PUBLIC ${TOKS_LIBRARY})
  message(STATUS "ingine_text: toks encoder enabled (${TOKS_LIBRARY})")
endif()
