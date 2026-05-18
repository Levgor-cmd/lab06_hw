[![Build Packages](https://github.com/Levgor-cmd/lab06_hw/actions/workflows/packages.yml/badge.svg?branch=main)](https://github.com/Levgor-cmd/lab06_hw/actions/workflows/packages.yml)

## Отчёт по домашнему заданию к лабораторной работе 06

### 1. Подготовка окружения и репозитория

```bash
lev@debian:~$ export GITHUB_USERNAME=Levgor-cmd
lev@debian:~$ export GITHUB_EMAIL=gorlinskiy.lev@bk.ru
lev@debian:~$ cd ${GITHUB_USERNAME}/workspace
lev@debian:~/Levgor-cmd/workspace$ rm -rf projects/lab06_hw
lev@debian:~/Levgor-cmd/workspace$ git clone https://github.com/Levgor-cmd/lab03_hw projects/lab06_hw
lev@debian:~/Levgor-cmd/workspace$ cd projects/lab06_hw
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ git remote remove origin
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab06_hw
```

### 2. Настройка CPack

#### CPackConfig.cmake:
```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ cat > CPackConfig.cmake <<'EOF'
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT ${GITHUB_EMAIL})
set(CPACK_PACKAGE_VERSION_MAJOR 0)
set(CPACK_PACKAGE_VERSION_MINOR 1)
set(CPACK_PACKAGE_VERSION_PATCH 0)
set(CPACK_PACKAGE_VERSION_TWEAK 0)
set(CPACK_PACKAGE_VERSION "0.1.0.0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Quadratic equation solver")

set(CPACK_RPM_PACKAGE_NAME "solver")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "Development/Tools")
set(CPACK_RPM_PACKAGE_RELEASE 1)

set(CPACK_DEBIAN_PACKAGE_NAME "solver")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "${GITHUB_EMAIL}")

include(CPack)
EOF
```

### 3. Обновление корневого CMakeLists.txt

```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ cat > CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.10)
project(solver_project)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(formatter_lib)
add_subdirectory(formatter_ex_lib)
add_subdirectory(solver_lib)
add_subdirectory(solver_application)

include(CPackConfig.cmake)
EOF
```

### 4. Обновление CMakeLists.txt для solver_application

```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ cat > solver_application/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.10)
project(solver_app)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(solver equation.cpp)
target_link_libraries(solver formatter_ex solver_lib)

install(TARGETS solver RUNTIME DESTINATION bin)
EOF
```

### 5. Сборка проекта

```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ rm -rf _build
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ cmake --build _build
```

#### Результат сборки:
```
[ 25%] Built target formatter
[ 50%] Built target formatter_ex
[ 75%] Built target solver_lib
[100%] Built target solver
```

### 6. Создание пакетов

```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ cd _build
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw/_build$ cpack -G TGZ
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw/_build$ cpack -G DEB
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw/_build$ cpack -G RPM
```

#### Результат:
- `solver_project-0.1.0.0-Linux.tar.gz` — создан
- `solver_project-0.1.0.0-Linux.rpm` — создан
- DEB пакет не создался (требуется дополнительная настройка maintainer)

### 7. Копирование артефактов

```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ mkdir -p artifacts
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ cp _build/*.tar.gz _build/*.rpm artifacts/
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ ls -la artifacts/
```

#### Результат:
```
-rw-rw-r-- 1 lev lev 13774 solver_project-0.1.0.0-Linux.rpm
-rw-rw-r-- 1 lev lev  6899 solver_project-0.1.0.0-Linux.tar.gz
```

### 8. Настройка GitHub Actions

```bash
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ mkdir -p .github/workflows
lev@debian:~/Levgor-cmd/workspace/projects/lab06_hw$ cat > .github/workflows/packages.yml <<'EOF'
name: Build Packages

on:
  push:
    tags:
      - 'v*'
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Install dependencies
      run: |
        sudo apt update
        sudo apt install -y cmake build-essential rpm

    - name: Configure
      run: cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install

    - name: Build
      run: cmake --build _build

    - name: Create packages
      run: |
        cd _build
        cpack -G TGZ
        cpack -G DEB
        cpack -G RPM

    - name: Create Release
      uses: softprops/action-gh-release@v1
      with:
        files: |
          _build/*.tar.gz
          _build/*.deb
          _build/*.rpm
        generate_release_notes: true
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
EOF
```
