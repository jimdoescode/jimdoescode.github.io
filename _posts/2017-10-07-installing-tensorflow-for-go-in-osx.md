---
layout: "post"
author: "jimdoescode"
title: "Installing Tensorflow for Go in OSX"
date: "2017-10-07 16:03:14"
tags: [golang,go,osx,tensorflow]
---

The documentation made this sound easy. Well, as with many things, it wasn't.

## Installing the dependencies

In order to build the tensor flow go bindings we need to use a build tool called [bazel](https://www.bazel.build/versions/master/docs/install.html).
Bazel requires Java 8 so you can't just install the latest version of java (java 9) to get bazel to work. I tried 
that and kept getting this output whenever I'd run the `./configure` command.
```
Extracting Bazel installation...
Problem with java installation: couldn't find/access rt.jar in /Library/Java/JavaVirtualMachines/jdk-9.jdk/Contents/Home
Traceback (most recent call last):
  File "configure.py", line 1036, in <module>
    main()
  File "configure.py", line 971, in main
    check_bazel_version('0.5.4')
  File "configure.py", line 444, in check_bazel_version
    curr_version = run_shell(['bazel', '--batch', 'version'])
  File "configure.py", line 141, in run_shell
    output = subprocess.check_output(cmd)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/subprocess.py", line 573, in check_output
    raise CalledProcessError(retcode, cmd, output=output)
subprocess.CalledProcessError: Command '['bazel', '--batch', 'version']' returned non-zero exit status 1
```
Turns out Java 9 got rid of the rt.jar (or maybe moved it) and bazel can't handle that. 
This has been a known issue for well over a year as of this post. The only fix that I found was to actually install java 8.
So I uninstalled java 9 then installed java 8.
```sh
$ brew cask uninstall java
$ brew cask install caskroom/versions/java8
```

## Compiling

Once made bazel was happy I was able to successfully run `./configure` and after that `bazel build --config opt //tensorflow:libtensorflow.so`

All that stuff went smoothly. I'm not sure if I answered all the questions asked by `./configure` correctly, but at least it built.

## Testing

After a successful build I didn't really want to copy the libtensorflow.so file to `/usr/local/lib`, instead I wanted to just specify it via
the test command. I misread the docs, and thought they said to use *only* the variable `DYLD_LIBRARY_PATH` so I ran the test command with that.
```sh
$ DYLD_LIBRARY_PATH=${GOPATH}/src/github.com/tensorflow/tensorflow/bazel-bin/tensorflow go test github.com/tensorflow/tensorflow/tensorflow/go
```
This gave the error output indicating that it wasn't able to find the libtensorflow.so library.
```
# github.com/tensorflow/tensorflow/tensorflow/go
ld: library not found for -ltensorflow
clang: error: linker command failed with exit code 1 (use -v to see invocation)
# github.com/tensorflow/tensorflow/tensorflow/go
ld: library not found for -ltensorflow
clang: error: linker command failed with exit code 1 (use -v to see invocation)
FAIL	github.com/tensorflow/tensorflow/tensorflow/go [build failed]
```
Turns out you need to run with both variables `LIBRARY_PATH` and `DYLD_LIBRARY_PATH`.
```sh
$ LIBRARY_PATH=${GOPATH}/src/github.com/tensorflow/tensorflow/bazel-bin/tensorflow DYLD_LIBRARY_PATH=$LIBRARY_PATH go test github.com/tensorflow/tensorflow/tensorflow/go
```

Moral of the story is, read the documentation carefully and sometimes required dependencies have outdated dependencies.
