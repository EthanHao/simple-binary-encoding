#
# Copyright 2013-2019 Real Logic Ltd.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

set(GTEST_LIBS
    ${GTEST_BINARY_DIR}/${CMAKE_CFG_INTDIR}/${LIBPREFIX}gtest${LIBSUFFIX}
    ${GTEST_BINARY_DIR}/${CMAKE_CFG_INTDIR}/${LIBPREFIX}gtest_main${LIBSUFFIX}
)

function(sbe_test name)
    add_executable("${name}" "${name}.cpp")
    target_link_libraries("${name}" sbe ${GTEST_LIBS} ${CMAKE_THREAD_LIBS_INIT})
    add_test(NAME ${name} COMMAND ${name} WORKING_DIRECTORY ${CODEC_TARGET_DIR})
    add_dependencies(${name} gtest)
    if(${ARGC} GREATER 1)
        add_dependencies(${name} ${ARGV1})
    endif()
endfunction()

find_package(Java REQUIRED)

set(CODE_GENERATION_SCHEMA ${CODEC_SCHEMA_DIR}/code-generation-schema.xml)
set(COMPOSITE_OFFSETS_SCHEMA ${CODEC_SCHEMA_DIR}/composite-offsets-schema.xml)
set(MESSAGE_BLOCK_LENGTH_TEST ${CODEC_SCHEMA_DIR}/message-block-length-test.xml)
set(GROUP_WITH_DATA_SCHEMA ${CODEC_SCHEMA_DIR}/group-with-data-schema.xml)
set(COMPOSITE_ELEMENTS_SCHEMA ${CODEC_SCHEMA_DIR}/composite-elements-schema.xml)

set(GENERATED_CODECS
    ${CODEC_TARGET_DIR}
)

add_custom_command(
    OUTPUT ${GENERATED_CODECS}
    DEPENDS ${CODE_GENERATION_SCHEMA} ${CODE_GENERATION_SCHEMA_CPP} ${COMPOSITE_OFFSETS_SCHEMA} ${MESSAGE_BLOCK_LENGTH_TEST}
            sbe-jar ${SBE_JAR}
    COMMAND ${Java_JAVA_EXECUTABLE}
        -Dsbe.output.dir=${CODEC_TARGET_DIR} -Dsbe.generate.ir="true" -Dsbe.target.language="cpp" -jar ${SBE_JAR}
        ${CODE_GENERATION_SCHEMA}
        ${COMPOSITE_OFFSETS_SCHEMA}
        ${MESSAGE_BLOCK_LENGTH_TEST}
        ${GROUP_WITH_DATA_SCHEMA}
        ${COMPOSITE_ELEMENTS_SCHEMA}
)

add_custom_target(codecs DEPENDS ${GENERATED_CODECS})

# codec tests
sbe_test(BoundsCheckTest codecs)
sbe_test(CodeGenTest codecs)
sbe_test(CompositeOffsetsCodeGenTest codecs)
sbe_test(CompositeOffsetsIrTest codecs)
sbe_test(MessageBlockLengthTest codecs)
sbe_test(GroupWithDataTest codecs)
sbe_test(Rc3OtfFullIrTest codecs)
sbe_test(CompositeElementsTest codecs)
