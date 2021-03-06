# Copyright (c) 2019 Google LLC

# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

if(SPIRV_BUILD_FUZZER)
  set(PROTOBUF_SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/protobufs/spvtoolsfuzz.proto)

  add_custom_command(
        OUTPUT protobufs/spvtoolsfuzz.pb.cc protobufs/spvtoolsfuzz.pb.h
        COMMAND protobuf::protoc
        -I=${CMAKE_CURRENT_SOURCE_DIR}/protobufs
        --cpp_out=protobufs
        ${PROTOBUF_SOURCE}
        DEPENDS ${PROTOBUF_SOURCE}
        COMMENT "Generate protobuf sources from proto definition file."
  )

  set(SPIRV_TOOLS_FUZZ_SOURCES
        fact_manager.h
        fuzzer.h
        fuzzer_context.h
        fuzzer_pass.h
        fuzzer_pass_add_dead_breaks.h
        fuzzer_pass_add_useful_constructs.h
        fuzzer_pass_obfuscate_constants.h
        fuzzer_pass_permute_blocks.h
        fuzzer_pass_split_blocks.h
        fuzzer_util.h
        id_use_descriptor.h
        protobufs/spirvfuzz_protobufs.h
        pseudo_random_generator.h
        random_generator.h
        replayer.h
        transformation.h
        transformation_add_constant_boolean.h
        transformation_add_constant_scalar.h
        transformation_add_dead_break.h
        transformation_add_type_boolean.h
        transformation_add_type_float.h
        transformation_add_type_int.h
        transformation_add_type_pointer.h
        transformation_move_block_down.h
        transformation_replace_boolean_constant_with_constant_binary.h
        transformation_replace_constant_with_uniform.h
        transformation_split_block.h
        uniform_buffer_element_descriptor.h
        ${CMAKE_CURRENT_BINARY_DIR}/protobufs/spvtoolsfuzz.pb.h

        fact_manager.cpp
        fuzzer.cpp
        fuzzer_context.cpp
        fuzzer_pass.cpp
        fuzzer_pass_add_dead_breaks.cpp
        fuzzer_pass_add_useful_constructs.cpp
        fuzzer_pass_obfuscate_constants.cpp
        fuzzer_pass_permute_blocks.cpp
        fuzzer_pass_split_blocks.cpp
        fuzzer_util.cpp
        id_use_descriptor.cpp
        pseudo_random_generator.cpp
        random_generator.cpp
        replayer.cpp
        transformation.cpp
        transformation_add_constant_boolean.cpp
        transformation_add_constant_scalar.cpp
        transformation_add_dead_break.cpp
        transformation_add_type_boolean.cpp
        transformation_add_type_float.cpp
        transformation_add_type_int.cpp
        transformation_add_type_pointer.cpp
        transformation_move_block_down.cpp
        transformation_replace_boolean_constant_with_constant_binary.cpp
        transformation_replace_constant_with_uniform.cpp
        transformation_split_block.cpp
        uniform_buffer_element_descriptor.cpp
        ${CMAKE_CURRENT_BINARY_DIR}/protobufs/spvtoolsfuzz.pb.cc
        )

  if(MSVC)
    # Enable parallel builds across four cores for this lib
    add_definitions(/MP4)
  endif()

  spvtools_pch(SPIRV_TOOLS_FUZZ_SOURCES pch_source_fuzz)

  add_library(SPIRV-Tools-fuzz ${SPIRV_TOOLS_FUZZ_SOURCES})

  spvtools_default_compile_options(SPIRV-Tools-fuzz)
  target_compile_definitions(SPIRV-Tools-fuzz PUBLIC -DGOOGLE_PROTOBUF_NO_RTTI -DGOOGLE_PROTOBUF_USE_UNALIGNED=0)

  # Compilation of the auto-generated protobuf source file will yield warnings,
  # which we have no control over and thus wish to ignore.
  if(${COMPILER_IS_LIKE_GNU})
    set_source_files_properties(${CMAKE_CURRENT_BINARY_DIR}/protobufs/spvtoolsfuzz.pb.cc PROPERTIES COMPILE_FLAGS -w)
  endif()
  if(MSVC)
    set_source_files_properties(${CMAKE_CURRENT_BINARY_DIR}/protobufs/spvtoolsfuzz.pb.cc PROPERTIES COMPILE_FLAGS /w)
  endif()

  target_include_directories(SPIRV-Tools-fuzz
        PUBLIC ${spirv-tools_SOURCE_DIR}/include
        PUBLIC ${SPIRV_HEADER_INCLUDE_DIR}
        PRIVATE ${spirv-tools_BINARY_DIR}
        PRIVATE ${CMAKE_BINARY_DIR})

  # The fuzzer reuses a lot of functionality from the SPIRV-Tools library.
  target_link_libraries(SPIRV-Tools-fuzz
        PUBLIC ${SPIRV_TOOLS}
        PUBLIC SPIRV-Tools-opt
        PUBLIC protobuf::libprotobuf)

  set_property(TARGET SPIRV-Tools-fuzz PROPERTY FOLDER "SPIRV-Tools libraries")
  spvtools_check_symbol_exports(SPIRV-Tools-fuzz)

  if(ENABLE_SPIRV_TOOLS_INSTALL)
      install(TARGETS SPIRV-Tools-fuzz
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
  endif(ENABLE_SPIRV_TOOLS_INSTALL)

endif(SPIRV_BUILD_FUZZER)
