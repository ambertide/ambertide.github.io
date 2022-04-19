---
layout: post
title: "Using Arduino Libraries as ESP-IDF Components"
date: 2022-04-19 05:00:00 +0300
categories: embedded
---

> Sources for this post comes mostly from [this reply to an issue](https://github.com/espressif/arduino-esp32/issues/5596#issuecomment-907981166) by the [GitHub user @1kc](https://github.com/1kc).

[ESP-IDF](https://github.com/espressif/arduino-esp32/issues/5596#issuecomment-907981166) is the official
framework used to program ESP32 based devices. Meanwhile, [Arduino-ESP32](https://github.com/espressif/arduino-esp32)
can be installed on top of it to support `Arduino.h` and other Arduino libraries when writing code for ESP-32.

This can be very useful for folks like me who are rather new to this whole embedded business, and can't wrap their
head around the ESP-32's wifi library. More importantly though, Arduino Core allows us to use Arduino libraries with
ESP-IDF.

There seems to be some lack of documentation regarding how exactly to do it, so I will go through it step by step,
I will be installing the popular [RF24](https://github.com/nRF24/RF24) library.

## Step 0: Getting the Arduino Core as a ESP-IDF Component

You should already have installed Arduino Core as a component, (like this)[https://docs.espressif.com/projects/arduino-esp32/en/latest/esp-idf_component.html]. This way, you will have something like the following directory structure:

```
.
├── components
│  └── arduino
├── CMakeLists.txt
└── main
   ├── include
   └── src
```

## Step 1: Fetch your library

Now, fetch your source library from GitHub or wherever it resides, and save it under the `components` directory.

```
git clone https://github.com/nRF24/RF24 components/RF24
```

## Step 2: Declaring a component

In general, ESP-IDF uses a system of components to
add extra utilities to your project.
According to the [documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-guides/build-system.html)
a components are"[...] modular pieces of standalone
code which are compiled into static libraries (.a
files) and linked into an app."

To detect successfully integrate our new Arduino
library to our project, we need to let the ESP-IDF
know these two things:

1. This is a component.
2. It depends on the Arduino component.

We will have to change some files to do so, to start
with, create (or replace if it already exists) a
`CMakeLists.txt` file and a `component.mk` file under
your library directory.

```
touch components/RF24/CMakeLists.txt
touch components/RF24/component.mk
```

Under the `component.mk` add the line:

```
COMPONENT_ADD_INCLUDEDIRS = .
```

So that the compiler knows where to look for headers.
A `component.mk` file is crucial and is called a [Component Makefile](https://docs.espressif.com/projects/esp-idf/en/release-v3.3/api-guides/build-system.html#component-makefiles) it may also
include other information such as the name and build directory of the component.

To let the build system know how to handle the
compilation of the source code of the library, add
the following to the `CMakeLists.txt`.

```cmake
idf_component_register(
    SRC_DIRS "."
    INCLUDE_DIRS "."
    REQUIRES arduino
)
```

As you can see, ESP-IDF uses a [CMake Based Build System](https://docs.espressif.com/projects/esp-idf/en/release-v3.3/get-started-cmake/index.html)
and the `idf_component_register` is one of the more
crucial additions on top of CMake. `SRC_DIRS` and
`INCLUDE_DIRS` are rather self-explanatory, and the
`REQUIRES` allows us to signal the build system that
our component relies on another component, in this
case `arduino`.

## Step 3: Rejoice

This concludes more or less our tutorial for how to
use Arduino libraries with your ESP32. Despite its
powerful features, ESP-IDF can be overwhelming for
newcomers, and Arduino libraries are a welcome
addition.

I once again want to note that the majority of this
blog post was sourced from [@1kc's reply to an issue](https://github.com/espressif/arduino-esp32/issues/5596#issuecomment-907981166). It is absurd to me how
this is not a part of the official documentation
but here we are.

As a final note, don't hesitate to delete an
existing CMakeLists.txt file from an Arduino library
to make it fit your needs.
