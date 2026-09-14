add_executable(test_shader_types shader_types_test.cpp)
NoGraphicsAPI_disable_exceptions(test_shader_types)
target_link_libraries(test_shader_types PRIVATE NoGraphicsAPIUtility::types)
add_test(NAME test_shader_types COMMAND test_shader_types)

add_executable(test_math math_test.cpp)
NoGraphicsAPI_disable_exceptions(test_math)
target_link_libraries(test_math PRIVATE NoGraphicsAPIUtility::math)
add_test(NAME test_math COMMAND test_math)

add_executable(test_allocators ../utility/tests/allocator_test.cpp)
NoGraphicsAPI_disable_exceptions(test_allocators)
find_package(Threads REQUIRED)
target_link_libraries(test_allocators PRIVATE NoGraphicsAPIUtility::allocators Threads::Threads)
add_test(NAME test_allocators COMMAND test_allocators)

add_executable(test_upload_queue ../utility/tests/upload_queue_test.cpp)
NoGraphicsAPI_disable_exceptions(test_upload_queue)
target_link_libraries(test_upload_queue PRIVATE NoGraphicsAPIUtility::uploads)
if(NOT COMMAND NoGraphicsAPI_compile_slang)
    include(${PROJECT_SOURCE_DIR}/examples/NoGraphicsAPI_slang.cmake)
endif()
set(upload_queue_shader ${CMAKE_CURRENT_SOURCE_DIR}/shaders/upload_queue.slang)
set(upload_queue_spv ${CMAKE_CURRENT_BINARY_DIR}/shaders/upload_queue.comp.spv)
NoGraphicsAPI_compile_slang(${upload_queue_spv} ${upload_queue_shader} computeMain compute
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/shaders/upload_queue_shared.h)
target_sources(test_upload_queue PRIVATE ${upload_queue_spv})
target_compile_definitions(test_upload_queue PRIVATE NOGRAPHICSAPI_UPLOAD_QUEUE_SPV_PATH="${upload_queue_spv}")
add_test(NAME test_upload_queue COMMAND test_upload_queue)
set_tests_properties(test_upload_queue PROPERTIES
    SKIP_RETURN_CODE 77 TIMEOUT 60 FAIL_REGULAR_EXPRESSION "NoGraphicsAPI validation:")

if(WIN32)
    add_executable(test_math_win32_include math_win32_include_test.cpp)
    NoGraphicsAPI_disable_exceptions(test_math_win32_include)
    target_link_libraries(test_math_win32_include PRIVATE NoGraphicsAPIUtility::math)
    add_test(NAME test_math_win32_include COMMAND test_math_win32_include)
endif()

add_executable(test_api api_test.cpp)
NoGraphicsAPI_disable_exceptions(test_api)
target_link_libraries(test_api PRIVATE NoGraphicsAPI::NoGraphicsAPI)
add_test(NAME test_api COMMAND test_api)

add_executable(test_command_context command_context_test.cpp)
NoGraphicsAPI_disable_exceptions(test_command_context)
target_link_libraries(test_command_context PRIVATE NoGraphicsAPI::NoGraphicsAPI)
add_test(NAME test_command_context COMMAND test_command_context)

add_executable(test_placed_texture placed_texture_test.cpp)
NoGraphicsAPI_disable_exceptions(test_placed_texture)
target_link_libraries(test_placed_texture PRIVATE NoGraphicsAPIUtility::textures)
add_test(NAME test_placed_texture COMMAND test_placed_texture)

add_executable(test_multithreading multithreading_test.cpp)
NoGraphicsAPI_disable_exceptions(test_multithreading)
target_link_libraries(test_multithreading PRIVATE NoGraphicsAPI::NoGraphicsAPI Threads::Threads)
add_test(NAME test_multithreading COMMAND test_multithreading)
add_test(NAME test_multiple_queues COMMAND test_multithreading --multiple-queues)

set_tests_properties(test_command_context test_placed_texture test_multithreading test_multiple_queues PROPERTIES
    SKIP_RETURN_CODE 77 TIMEOUT 60 FAIL_REGULAR_EXPRESSION "Validation Error;NoGraphicsAPI validation:")

set(continuation_vertex_spv ${CMAKE_CURRENT_BINARY_DIR}/shaders/render_continuation.vert.spv)
set(continuation_fragment_spv ${CMAKE_CURRENT_BINARY_DIR}/shaders/render_continuation.frag.spv)
NoGraphicsAPI_compile_slang(${continuation_vertex_spv} ${CMAKE_CURRENT_SOURCE_DIR}/render_continuation.slang vertexMain vertex)
NoGraphicsAPI_compile_slang(${continuation_fragment_spv} ${CMAKE_CURRENT_SOURCE_DIR}/render_continuation.slang fragmentMain fragment)
add_executable(test_render_continuation render_continuation_test.cpp ${continuation_vertex_spv} ${continuation_fragment_spv})
NoGraphicsAPI_disable_exceptions(test_render_continuation)
target_link_libraries(test_render_continuation PRIVATE NoGraphicsAPI::NoGraphicsAPI Threads::Threads)
target_compile_definitions(test_render_continuation PRIVATE
    NOGRAPHICSAPI_CONTINUATION_VERTEX_SPV="${continuation_vertex_spv}"
    NOGRAPHICSAPI_CONTINUATION_FRAGMENT_SPV="${continuation_fragment_spv}")
add_test(NAME test_render_continuation COMMAND test_render_continuation)
set_tests_properties(test_render_continuation PROPERTIES
    SKIP_RETURN_CODE 77 TIMEOUT 60 FAIL_REGULAR_EXPRESSION "Validation Error;NoGraphicsAPI validation:")

if(WIN32)
    add_executable(test_presentation presentation_test.cpp)
    NoGraphicsAPI_disable_exceptions(test_presentation)
    target_link_libraries(test_presentation PRIVATE NoGraphicsAPI::NoGraphicsAPI)
    add_test(NAME test_presentation COMMAND test_presentation)
    set_tests_properties(test_presentation PROPERTIES
        SKIP_RETURN_CODE 77 TIMEOUT 60 FAIL_REGULAR_EXPRESSION "NoGraphicsAPI validation:")
endif()

set(task_shader_spv ${CMAKE_CURRENT_BINARY_DIR}/shaders/task_shader.task.spv)
set(task_mesh_spv ${CMAKE_CURRENT_BINARY_DIR}/shaders/task_mesh.mesh.spv)
set(task_fragment_spv ${CMAKE_CURRENT_BINARY_DIR}/shaders/task_mesh.frag.spv)
NoGraphicsAPI_compile_slang(${task_shader_spv} ${CMAKE_CURRENT_SOURCE_DIR}/task_shader.slang taskMain amplification
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/task_shader_shared.h ${CMAKE_CURRENT_SOURCE_DIR}/task_shader_common.h)
NoGraphicsAPI_compile_slang(${task_mesh_spv} ${CMAKE_CURRENT_SOURCE_DIR}/task_mesh.slang meshMain mesh
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/task_mesh_shared.h ${CMAKE_CURRENT_SOURCE_DIR}/task_shader_common.h)
NoGraphicsAPI_compile_slang(${task_fragment_spv} ${CMAKE_CURRENT_SOURCE_DIR}/task_mesh.slang fragmentMain fragment
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/task_mesh_shared.h ${CMAKE_CURRENT_SOURCE_DIR}/task_shader_common.h)
add_executable(test_task_shader task_shader_test.cpp ${task_shader_spv} ${task_mesh_spv} ${task_fragment_spv})
NoGraphicsAPI_disable_exceptions(test_task_shader)
target_link_libraries(test_task_shader PRIVATE NoGraphicsAPI::NoGraphicsAPI NoGraphicsAPIUtility::types)
target_compile_definitions(test_task_shader PRIVATE
    NOGRAPHICSAPI_TASK_TEST_TASK_SPV="${task_shader_spv}"
    NOGRAPHICSAPI_TASK_TEST_MESH_SPV="${task_mesh_spv}"
    NOGRAPHICSAPI_TASK_TEST_FRAGMENT_SPV="${task_fragment_spv}")
add_test(NAME test_task_shader COMMAND test_task_shader)
set_tests_properties(test_task_shader PROPERTIES SKIP_RETURN_CODE 77 TIMEOUT 60 FAIL_REGULAR_EXPRESSION "Validation Error;NoGraphicsAPI validation:")
