# libffi

Fork from the original [libffi](https://github.com/libffi/libffi) which can be compiled using the [Cmake](https://cmake.org/) build system. This project is meant to help the build of [Calculate Binding](https://github.com/newlawrence/Calculate_Binding) project, so the number of platforms supported may be lower than the original's (at first, mainly **x86**).

##### Library version [3.2.1](https://github.com/libffi/libffi/tree/20562ac0427c3578250d04c6e34fb0127d4551cf)

### List of currently supported platforms and compilers

* **Windows x86 - Microsoft Visual C++ Compiler**
* **Windows x86-64 - Microsoft Visual C++ Compiler**

### Build instructions

Using the **Microsoft Visual C++ Compiler** under **Windows**:
```
cmake -H. -B{build_dir} -G"NMake Makefiles" -DCMAKE_BUILD_TARGET=<Debug|Release>
cmake --build {build_dir} --target ffi
```

### License

See accompanying [copying](https://github.com/newlawrence/libffi/blob/master/copying) file.
