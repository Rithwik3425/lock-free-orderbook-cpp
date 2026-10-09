cmake_minimum_required(VERSION 3.20)
project(orderbook CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_compile_options(-Wall -Wextra)

# Catches memory bugs while you develop (Debug builds only)
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  add_compile_options(-fsanitize=address,undefined)
  add_link_options(-fsanitize=address,undefined)
endif()

add_library(book src/order_book.cpp)
target_include_directories(book PUBLIC src)

add_executable(orderbook src/main.cpp)
target_link_libraries(orderbook PRIVATE book)


# Create empty source files if they don't exist yet
file(TOUCH ${CMAKE_CURRENT_SOURCE_DIR}/src/types.hpp)
file(TOUCH ${CMAKE_CURRENT_SOURCE_DIR}/src/order_book.hpp)
file(TOUCH ${CMAKE_CURRENT_SOURCE_DIR}/src/order_book.cpp)

if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp")
  file(WRITE ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp
    "#include <iostream>\n\nint main() {\n    std::cout << \"order book starting\\n\";\n    return 0;\n}\n"
  )
endif()
