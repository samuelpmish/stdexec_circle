# Configuring CMake w/ `circle` (182)

To set `circle` as the compiler, configure with

```
-DCMAKE_C_COMPILER:FILEPATH=/path/to/circle
-DCMAKE_CXX_COMPILER:FILEPATH=/path/to/circle
```

I ran into issues with CMake's automated tests to determine if a CXX compiler is valid, but apparently these can be circumvented by setting

```
-DCMAKE_C_COMPILER_WORKS=1
-DCMAKE_CXX_COMPILER_WORKS=1
```

CMake was also unable to figure out how to use pthreads when using `circle` (I don't understand why the compiler matters here), but another workaround was to include these configure arguments:

```
-DCMAKE_THREAD_LIBS_INIT=-lpthread
-DCMAKE_HAVE_THREADS_LIBRARY=1
-DCMAKE_USE_WIN32_THREADS_INIT=0
-DCMAKE_USE_PTHREADS_INIT=1
-DTHREADS_PREFER_PTHREAD_FLAG=ON
```

So, the final configure line is:

```
/usr/bin/cmake --no-warn-unused-cli 
-DCMAKE_C_COMPILER_WORKS=1 
-DCMAKE_CXX_COMPILER_WORKS=1 
-DCMAKE_THREAD_LIBS_INIT=-lpthread 
-DCMAKE_HAVE_THREADS_LIBRARY=1 
-DCMAKE_USE_WIN32_THREADS_INIT=0 
-DCMAKE_USE_PTHREADS_INIT=1 
-DTHREADS_PREFER_PTHREAD_FLAG=ON 
-DCMAKE_EXPORT_COMPILE_COMMANDS:BOOL=TRUE 
-DCMAKE_BUILD_TYPE:STRING=Debug 
-DCMAKE_C_COMPILER:FILEPATH=/home/sam/code/circle/circle 
-DCMAKE_CXX_COMPILER:FILEPATH=/home/sam/code/circle/circle 
-S/home/sam/code/stdexec_circle 
-B/home/sam/code/stdexec_circle/build 
-G Ninja
```

# Building

CMake is inserting a lot of "-MD -MT" for each translation unit, and it looks like these flags aren't supported on my current build of `circle` (although it sounds like this may be addressed in a newer version of circle).

```
$ ./circle -MD -MT sanity.cxx 
error: macro name -MT must be an identifier
```
