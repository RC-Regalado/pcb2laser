cmake_minimum_required(VERSION 3.10)

project(PdfRenderer C CXX)
find_library(WKHTMLTOX_LIB wkhtmltox REQUIRED PATHS /usr/local/lib /usr/lib /path/to/your/lib)

# Buscar los encabezados de wkhtmltox
find_path(WKHTMLTOX_INCLUDE_DIR wkhtmltox/pdf.h PATHS /usr/local/include /usr/include /path/to/your/include)

include(common.cmake)

get_filename_component(hw_proto "protos/render.proto" ABSOLUTE)
get_filename_component(hw_proto_path "${hw_proto}" PATH)

set(hw_proto_srcs "${CMAKE_CURRENT_BINARY_DIR}/render.pb.cc")
set(hw_proto_hdrs "${CMAKE_CURRENT_BINARY_DIR}/render.pb.h")
set(hw_grpc_srcs "${CMAKE_CURRENT_BINARY_DIR}/render.grpc.pb.cc")
set(hw_grpc_hdrs "${CMAKE_CURRENT_BINARY_DIR}/render.grpc.pb.h")
add_custom_command(
      OUTPUT "${hw_proto_srcs}" "${hw_proto_hdrs}" "${hw_grpc_srcs}" "${hw_grpc_hdrs}"
      COMMAND ${_PROTOBUF_PROTOC}
      ARGS --grpc_out "${CMAKE_CURRENT_BINARY_DIR}"
        --cpp_out "${CMAKE_CURRENT_BINARY_DIR}"
        -I "${hw_proto_path}"
        --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
        "${hw_proto}"
      DEPENDS "${hw_proto}")

include_directories("${CMAKE_CURRENT_BINARY_DIR}")
include_directories(${WKHTMLTOX_INCLUDE_DIRS})

add_library(hw_grpc_proto
  ${hw_grpc_srcs}
  ${hw_grpc_hdrs}
  ${hw_proto_srcs}
  ${hw_proto_hdrs})
target_link_libraries(hw_grpc_proto
  ${_REFLECTION}
  ${_GRPC_GRPCPP}
  ${_PROTOBUF_LIBPROTOBUF})

add_executable(render src/main.cpp)
target_link_libraries(render
    hw_grpc_proto
    absl::flags
    absl::flags_parse
    ${_REFLECTION}
    ${_GRPC_GRPCPP}
    ${_PROTOBUF_LIBPROTOBUF}
    ${WKHTMLTOX_LIB}
  )

add_compile_options(
    -lwkhtmltox
)

# Establece el directorio de salida
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
