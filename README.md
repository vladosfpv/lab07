# lab-07
Данная лабораторная работа посвещена изучению систем автоматизации развёртывания и управления приложениями на примере Docker

# Task 1
![Снимок экрана от 2023-04-15 16-22-52](https://user-images.githubusercontent.com/125737299/232237200-bf3e9892-836d-4d1a-ae8e-73c6afe9488f.png)



Let's create the executable files for this lab. They will be in separate folders, which will then be linked using `CMakeLists.txt`:

Make `print.hpp`

```
$ mkdir include
$ cd include
```

`print.hpp` contents:

```
#include <fstream>
#include <iostream>
#include <string>

void print(const std::string& text, std::ofstream& out);
void print(const std::string& text, std::ostream& out = std::cout);
```
Make `print.cpp`
```
$ mkdir source
$ cd source
```

`print.cpp` contents:

```
#include <print.hpp>

void print(const std::string& text, std::ostream& out)
{
  out << text;
}

void print(const std::string& text, std::ofstream& out)
{
  out << text;
}
```

Make `main.cpp`
```
$ mkdir demo
$ cd demo
```
`main.cpp` contents:
```
#include <print.hpp>
#include <cstdlib>

int main(int argc, char* argv[])
{
  const char* log_path = std::getenv("LOG_PATH");
  if (log_path == nullptr)
  {
    std::cerr << "undefined environment variable: LOG_PATH" << std::endl;
    return 1;
  }
  std::string text;
  while (std::cin >> text)
  {
    std::ofstream out{log_path, std::ios_base::app};
    print(text, out);
    out << std::endl;
  }
}
```

![Снимок экрана от 2023-04-15 16-33-39](https://user-images.githubusercontent.com/125737299/232237208-5f9c6be7-1577-4249-8248-4f97afbe294c.png)


Also we make the `logs\` directory, where we have a `.txt` file

After we make a `CMakeLists.txt` to link our project

`CMakeLists.txt` contents:
```
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_EXAMPLES "Build examples" OFF)

project(print)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/source/print.cpp)

#Вот добавленные строки

add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print) 
install(TARGETS demo RUNTIME DESTINATION bin)

#Это они закончились

target_include_directories(print PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

if(BUILD_EXAMPLES)
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*.cpp")
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_SOURCE})
    target_link_libraries(${EXAMPLE_NAME} print)
    install(TARGETS ${EXAMPLE_NAME}
      RUNTIME DESTINATION bin
    )
  endforeach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
endif()

install(TARGETS print
    EXPORT print-config
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
install(EXPORT print-config DESTINATION cmake)
```
![Снимок экрана от 2023-04-15 16-34-56](https://user-images.githubusercontent.com/125737299/232237220-86470c65-e9a3-4a5e-b3d5-949a9dc74df4.png)



And make `Dockerfile` to using the Docker

`Dockerfile` contents:
```
FROM ubuntu:20.04

RUN apt update
RUN apt install -yy gcc g++ cmake

COPY . print/
WORKDIR print

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install

ENV LOG_PATH /home/lab-07/logs/log.txt
VOLUME /home/lab-07/logs

WORKDIR _install/bin
ENTRYPOINT ./demo
```
```
$ git add .
$ git commit -m "Pushing project"
$ git push origin master
```
![Снимок экрана от 2023-04-15 16-52-45](https://user-images.githubusercontent.com/125737299/232237234-6e25e866-615a-422c-994e-4b4c94cc07c4.png)



After make a `.yml` file

`Action.yml` contents:
```
name: docker
on:
  push:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: build docker
        run: docker build -t logger .
```

![Снимок экрана от 2023-04-15 16-52-08](https://user-images.githubusercontent.com/125737299/232237228-6fa27938-757a-41a2-902e-3bb175051f87.png)


Now we will test our project in local repositories

Download `Docker`:
```
$ sudo apt install docker.io
```
And test:
```
$ sudo docker build -t logger .
$ sudo docker images
$ sudo docker run -it -v "$(pwd)/logs/:/home/lab-07/logs/" logger
$ sudo docker inspect logger
$ cat logs/log.txt
```
![Снимок экрана от 2023-04-15 18-18-10](https://user-images.githubusercontent.com/125737299/232237248-8dd0f42f-b6c2-4b7d-8e70-98493532ebdc.png)
![Снимок экрана от 2023-04-15 18-18-23](https://user-images.githubusercontent.com/125737299/232237259-bb6f1b5d-85f3-4706-af08-4a88e5e129d8.png)


In `log.txt`will a appear text 
