
if (NOT CMAKE_INSTALL_PREFIX OR
    CMAKE_INSTALL_PREFIX STREQUAL "/usr/local")
  set(CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_SOURCE_DIR}/../../../Install")
endif()

## TODO : support find_package for external go runtime
## find_package(go 1.10 REQUIRED)

file(GLOB_RECURSE GETH_1.9.1DEV_SRCS ${CMAKE_CURRENT_SOURCE_DIR} *.go *.s)

add_custom_target(geth_1.9.1dev ALL
  DEPENDS ${CMAKE_INSTALL_PREFIX}/plugins/libgeth_1.9.1dev.so
  )

add_custom_command(OUTPUT ${CMAKE_INSTALL_PREFIX}/plugins/libgeth_1.9.1dev.so
  DEPENDS geth_setup_build
  DEPENDS ${GETH_1.9.1DEV_SRCS} # for re-compilation when the source is modified
  COMMAND ${CMAKE_COMMAND} -E env "GOPATH=${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace" go install -buildmode c-shared -v ./cmd/geth
  COMMAND ${CMAKE_COMMAND} -E env "GOPATH=${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace" go install -v ./cmd/geth
  COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace/pkg/linux_amd64_shared/github.com/ethereum/go-ethereum/cmd/geth.a ${CMAKE_INSTALL_PREFIX}/plugins/libgeth_1.9.1dev.so
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace/src/github.com/ethereum/go-ethereum
  )

add_custom_target(geth_setup_build
  COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace/src/github.com/ethereum
  COMMAND ${CMAKE_COMMAND} -E create_symlink ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace/src/github.com/ethereum/go-ethereum  
  )

## add custom target and command for creating geth with gccgo
add_custom_target(geth_1.9.1dev_gccgo ALL
        DEPENDS ${CMAKE_INSTALL_PREFIX}/plugins/libgeth_1.9.1dev_gccgo.so
        )
add_custom_command(OUTPUT ${CMAKE_INSTALL_PREFIX}/plugins/libgeth_1.9.1dev_gccgo.so
        DEPENDS geth_setup_build_gccgo
        DEPENDS ${GETH_1.9.1DEV_SRCS}
        COMMAND ${CMAKE_COMMAND} -E env "GOPATH=${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace" go install -buildmode c-shared -compiler gccgo -v ./cmd/geth
        COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace/pkg/gccgo_linux_amd64_fPIC/github.com/ethereum/go-ethereum/cmd/libgeth.a ${CMAKE_INSTALL_PREFIX}/plugins/libgeth_1.9.1dev_gccgo.so
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace/src/github.com/ethereum/go-ethereum
        )
add_custom_target(geth_setup_build_gccgo
        COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace/src/github.com/ethereum
        COMMAND ${CMAKE_COMMAND} -E create_symlink ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/build/_workspace/src/github.com/ethereum/go-ethereum
        )



# Define targets that can be used in same build-tree
add_library(Geth_1_9_1dev::Plugin SHARED IMPORTED GLOBAL)
add_dependencies(Geth_1_9_1dev::Plugin geth_1.9.1dev)
set_target_properties(Geth_1_9_1dev::Plugin PROPERTIES
  IMPORTED_LINK_INTERFACE_LANGUAGES "C"
  IMPORTED_LOCATION ${CMAKE_INSTALL_PREFIX}/plugins/libgeth_1.9.1dev.so)


add_library(Geth_1_9_1dev_gccgo::Plugin SHARED IMPORTED GLOBAL)
add_dependencies(Geth_1_9_1dev_gccgo::Plugin geth_1.9.1dev_gccgo)
set_target_properties(Geth_1_9_1dev_gccgo::Plugin PROPERTIES
        IMPORTED_LINK_INTERFACE_LANGUAGES "C"
        IMPORTED_LOCATION ${CMAKE_INSTALL_PREFIX}/plugins/libgeth_1.9.1dev_gccgo.so)

