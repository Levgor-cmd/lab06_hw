## Отчёт к лабораторной работе 03 (домашнее задание)

### 1. Подготовка окружения

```bash
lev@debian:~$ export GITHUB_USERNAME=Levgor-cmd
lev@debian:~$ cd ${GITHUB_USERNAME}/workspace
lev@debian:~/Levgor-cmd/workspace$ pushd .
lev@debian:~/Levgor-cmd/workspace$ rm -rf projects/lab03_hw
lev@debian:~/Levgor-cmd/workspace$ git clone https://github.com/tp-labs/lab03 projects/lab03_hw
lev@debian:~/Levgor-cmd/workspace$ cd projects/lab03_hw
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ git remote remove origin
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab03_hw
```

### 2. Создание CMakeLists.txt для каждого компонента

#### Корневой CMakeLists.txt:
```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ cat > CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.4)
project(FormatterProject)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(formatter_lib)
add_subdirectory(formatter_ex_lib)
add_subdirectory(solver_lib)
add_subdirectory(hello_world_application)
add_subdirectory(solver_application)
EOF
```

#### formatter_lib/CMakeLists.txt:
```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ cat > formatter_lib/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.4)
project(formatter)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(formatter STATIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter.cpp)
target_include_directories(formatter PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
EOF
```

#### formatter_ex_lib/CMakeLists.txt:
```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ cat > formatter_ex_lib/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.4)
project(formatter_ex)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(formatter_ex STATIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter_ex.cpp)
target_include_directories(formatter_ex PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(formatter_ex formatter)
EOF
```

#### solver_lib/CMakeLists.txt:
```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ cat > solver_lib/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.4)
project(solver_lib)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(solver_lib STATIC ${CMAKE_CURRENT_SOURCE_DIR}/solver.cpp)
target_include_directories(solver_lib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
EOF
```

#### hello_world_application/CMakeLists.txt:
```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ cat > hello_world_application/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.4)
project(hello_world)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(hello_world ${CMAKE_CURRENT_SOURCE_DIR}/hello_world.cpp)
target_link_libraries(hello_world formatter_ex)
EOF
```

#### solver_application/CMakeLists.txt:
```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ cat > solver_application/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.4)
project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(solver ${CMAKE_CURRENT_SOURCE_DIR}/equation.cpp)
target_link_libraries(solver formatter_ex solver_lib)
EOF
```

### 3. Исправление ошибки в solver_lib/solver.cpp

#### В исходном коде использовался `std::sqrtf` без подключения `<cmath>`. Исправленный файл:

```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ cat > solver_lib/solver.cpp <<'EOF'
#include "solver.h"
#include <cmath>
#include <stdexcept>

void solve(float a, float b, float c, float& x1, float& x2)
{
    float d = (b * b) - (4 * a * c);

    if (d < 0)
    {
        throw std::logic_error{"error: discriminant < 0"};
    }

    x1 = (-b - std::sqrt(d)) / (2 * a);
    x2 = (-b + std::sqrt(d)) / (2 * a);
}
EOF
```

### 4. Сборка проекта

```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ rm -rf _build
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ cmake -H. -B_build
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ cmake --build _build
```

### 5. Запуск программ

```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ _build/hello_world_application/hello_world
-------------------------
hello, world!
-------------------------
```

```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab03_hw$ _build/solver_application/solver
1
-5
4
-------------------------
x1 = 1.000000
-------------------------
-------------------------
x2 = 4.000000
-------------------------
```
