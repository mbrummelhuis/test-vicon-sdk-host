cmake_minimum_required(VERSION 3.14)
project(vicon-datastream-sdk VERSION 1.13.0 LANGUAGES CXX)

# The v1.13 SDK source uses C++17/20 features (e.g. std::string_view).
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(GNUInstallDirs)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

# C++ DataStream client, built as a static library from the SDK source.
add_library(ViconDataStreamSDK_CPP STATIC
    Vicon/CrossMarket/DataStream/ViconDataStreamSDKCore/CoreClient.cpp
    Vicon/CrossMarket/DataStream/ViconDataStreamSDKCore/CoreClientTimingLog.cpp
    Vicon/CrossMarket/DataStream/ViconDataStreamSDKCore/WirelessConfiguration.cpp
    Vicon/CrossMarket/DataStream/ViconDataStreamSDKCoreUtils/ClientUtils.cpp
    Vicon/CrossMarket/DataStream/ViconDataStreamSDKCoreUtils/AxisMapping.cpp
    Vicon/CrossMarket/DataStream/ViconCGStreamClientSDK/CGClient.cpp
    Vicon/CrossMarket/DataStream/ViconCGStreamClient/ViconCGStreamClient.cpp
    Vicon/CrossMarket/DataStream/ViconCGStreamClient/CGStreamReaderWriter.cpp
    Vicon/CrossMarket/DataStream/ViconCGStreamClient/ViconCGStreamBayer.cpp
    Vicon/CrossMarket/StreamCommon/CGStreamAsyncReaderWriter.cpp
    Vicon/CrossMarket/DataStream/ViconDataStreamSDK_CPP/DataStreamClient.cpp
)

# Internal source dependencies use <Component/Header.h> includes rooted at
# these directories; the SDK networks over vendored standalone Asio.
target_include_directories(ViconDataStreamSDK_CPP
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/Vicon/CrossMarket/DataStream
        ${CMAKE_CURRENT_SOURCE_DIR}/Vicon/CrossMarket
        ${CMAKE_CURRENT_SOURCE_DIR}/asio/include
    PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Vicon/CrossMarket/DataStream/ViconDataStreamSDK_CPP>"
        "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}>"
)

target_compile_definitions(ViconDataStreamSDK_CPP PRIVATE
    ASIO_STANDALONE=1
    TCM_LINUX
    TCM_UNIX
    _BUILD_STATIC_LIB
)

target_link_libraries(ViconDataStreamSDK_CPP PRIVATE Threads::Threads)

# Install the public C++ headers and the static library so downstream
# projects can consume them from the install tree.
install(FILES
    Vicon/CrossMarket/DataStream/ViconDataStreamSDK_CPP/DataStreamClient.h
    Vicon/CrossMarket/DataStream/ViconDataStreamSDK_CPP/IDataStreamClientBase.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)
install(TARGETS ViconDataStreamSDK_CPP
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
