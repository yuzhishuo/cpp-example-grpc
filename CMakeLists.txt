cmake_minimum_required(VERSION 3.20.0)

#set(VCPKG_ROOT "/opt/vcpkg")
#set(CMAKE_TOOLCHAIN_FILE "${VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake" CACHE STRING "")

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})

project(samplate-cpp-template VERSION 0.1.0)

find_package(Protobuf CONFIG REQUIRED)

find_package(Threads REQUIRED)

message(STATUS "Using protobuf ${Protobuf_VERSION}")
set(_PROTOBUF_LIBPROTOBUF protobuf::libprotobuf)
set(_REFLECTION gRPC::grpc++_reflection )
set(_PROTOBUF_PROTOC $<TARGET_FILE:protobuf::protoc>)

find_package(gRPC CONFIG REQUIRED)
message(STATUS "Using gRPC ${gRPC_VERSION}")

file(GLOB PROTO_FILES protos/*.proto)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/cpp)

set(_GRPC_GRPCPP gRPC::grpc++ )
set(_GRPC_CPP_PLUGIN_EXECUTABLE $<TARGET_FILE:gRPC::grpc_cpp_plugin> )

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${LOCAL_W_FLAGS} ${LOCAL_C_FLAGS} -fPIC")

foreach(OBJ ${PROTO_FILES})
    
    get_filename_component(OBJNAME ${OBJ} NAME_WLE)
    set(proto_srcs ${CMAKE_CURRENT_BINARY_DIR}/cpp/${OBJNAME}.pb.cc)
    set(proto_hdrs ${CMAKE_CURRENT_BINARY_DIR}/cpp/${OBJNAME}.pb.h)
    list(APPEND SRCS ${proto_srcs})

    set(grpc_srcs ${CMAKE_CURRENT_BINARY_DIR}/cpp/${OBJNAME}.grpc.pb.cc)
    set(grpc_hdrs ${CMAKE_CURRENT_BINARY_DIR}/cpp/${OBJNAME}.grpc.pb.h)
    list(APPEND SRCS ${grpc_srcs})
    add_custom_command(
      OUTPUT "${proto_srcs}" "${proto_hdrs}" "${grpc_srcs}" "${grpc_hdrs}"
      COMMAND ${_PROTOBUF_PROTOC}
      ARGS --grpc_out "${CMAKE_CURRENT_BINARY_DIR}/cpp"
        --cpp_out "${CMAKE_CURRENT_BINARY_DIR}/cpp"
        -I "${CMAKE_CURRENT_LIST_DIR}/protos"
        --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
        "${OBJ}"
      DEPENDS "${OBJ}")
endforeach()

include_directories(./include)
include_directories(./build/cpp)
FIND_LIBRARY(RUNTIME_LIB rt /usr/lib  /usr/local/lib)

aux_source_directory(./src DIR_SRCS)
add_executable(samplate-cpp-template ${DIR_SRCS} ${SRCS} server.cpp )
add_executable(samplate-cpp-template1 ${DIR_SRCS} ${SRCS} client.cpp )

target_link_libraries(samplate-cpp-template PRIVATE protobuf::libprotoc protobuf::libprotobuf 
	gRPC::grpc++
	Threads::Threads
	)

target_link_libraries(samplate-cpp-template1 PRIVATE protobuf::libprotoc protobuf::libprotobuf 
gRPC::grpc++
Threads::Threads
)

