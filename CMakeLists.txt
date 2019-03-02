cmake_minimum_required(VERSION 3.10)

set(sources
	main.cc
	message_throttler.hpp
	message_throttler_interface.hpp
	message_throttler_commons.hpp
)

find_package(Boost REQUIRED)

add_executable(main ${sources})
include_directories(${Boost_INCLUDE_DIRS})