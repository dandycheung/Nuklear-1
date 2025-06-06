# Nuklear
![](https://cloud.githubusercontent.com/assets/8057201/11761525/ae06f0ca-a0c6-11e5-819d-5610b25f6ef4.gif)

## Contents
1. About section
2. Highlights section
3. Features section
4. Usage section
    1. Flags section
    2. Constants section
    3. Dependencies section
5. Example section
6. API section
    1. Context section
    2. Input section
    3. Drawing section
    4. Window section
    5. Layouting section
    6. Groups section
    7. Tree section
    8. Properties section
7. License section
8. Changelog section
9. Gallery section
10. Credits section

## About
This is a minimal state immediate mode graphical user interface toolkit
written in ANSI C and licensed under public domain. It was designed as a simple
embeddable user interface for application and does not have any dependencies,
a default renderbackend or OS window and input handling but instead provides a very modular
library approach by using simple input state for input and draw
commands describing primitive shapes as output. So instead of providing a
layered library that tries to abstract over a number of platform and
render backends it only focuses on the actual UI.

## Highlights
- Graphical user interface toolkit
- Single header library
- Written in C89 (a.k.a. ANSI C or ISO C90)
- Small codebase (~18kLOC)
- Focus on portability, efficiency and simplicity
- No dependencies (not even the standard library if not wanted)
- Fully skinnable and customizable
- Low memory footprint with total memory control if needed or wanted
- UTF-8 support
- No global or hidden state
- Customizable library modules (you can compile and use only what you need)
- Optional font baker and vertex buffer output
- [Code available on github](https://github.com/Immediate-Mode-UI/Nuklear/)

## Features
- Absolutely no platform dependent code
- Memory management control ranging from/to
    - Ease of use by allocating everything from standard library
    - Control every byte of memory inside the library
- Font handling control ranging from/to
    - Use your own font implementation for everything
    - Use this libraries internal font baking and handling API
- Drawing output control ranging from/to
    - Simple shapes for more high level APIs which already have drawing capabilities
    - Hardware accessible anti-aliased vertex buffer output
- Customizable colors and properties ranging from/to
    - Simple changes to color by filling a simple color table
    - Complete control with ability to use skinning to decorate widgets
- Bendable UI library with widget ranging from/to
    - Basic widgets like buttons, checkboxes, slider, ...
    - Advanced widget like abstract comboboxes, contextual menus,...
- Compile time configuration to only compile what you need
    - Subset which can be used if you do not want to link or use the standard library
- Can be easily modified to only update on user input instead of frame updates

## Usage
This library is self contained in one single header file and can be used either
in header only mode or in implementation mode. The header only mode is used
by default when included and allows including this header in other headers
and does not contain the actual implementation. <br /><br />

The implementation mode requires to define  the preprocessor macro
NK_IMPLEMENTATION in *one* .c/.cpp file before #including this file, e.g.:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~C
    #define NK_IMPLEMENTATION
    #include "nuklear.h"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Also optionally define the symbols listed in the section "OPTIONAL DEFINES"
below in header and implementation mode if you want to use additional functionality
or need more control over the library.

!!! WARNING
    Every time nuklear is included define the same compiler flags. This very important not doing so could lead to compiler errors or even worse stack corruptions.

### Flags
Flag                            | Description
--------------------------------|------------------------------------------
NK_PRIVATE                      | If defined declares all functions as static, so they can only be accessed inside the file that contains the implementation
NK_INCLUDE_FIXED_TYPES          | If defined it will include header `<stdint.h>` for fixed sized types otherwise nuklear tries to select the correct type. If that fails it will throw a compiler error and you have to select the correct types yourself.
NK_INCLUDE_DEFAULT_ALLOCATOR    | If defined it will include header `<stdlib.h>` and provide additional functions to use this library without caring for memory allocation control and therefore ease memory management.
NK_INCLUDE_STANDARD_IO          | If defined it will include header `<stdio.h>` and provide additional functions depending on file loading.
NK_INCLUDE_STANDARD_VARARGS     | If defined it will include header <stdarg.h> and provide additional functions depending on file loading.
NK_INCLUDE_STANDARD_BOOL        | If defined it will include header `<stdbool.h>` for nk_bool otherwise nuklear defines nk_bool as int.
NK_INCLUDE_VERTEX_BUFFER_OUTPUT | Defining this adds a vertex draw command list backend to this library, which allows you to convert queue commands into vertex draw commands. This is mainly if you need a hardware accessible format for OpenGL, DirectX, Vulkan, Metal,...
NK_INCLUDE_FONT_BAKING          | Defining this adds `stb_truetype` and `stb_rect_pack` implementation to this library and provides font baking and rendering. If you already have font handling or do not want to use this font handler you don't have to define it.
NK_INCLUDE_DEFAULT_FONT         | Defining this adds the default font: ProggyClean.ttf into this library which can be loaded into a font atlas and allows using this library without having a truetype font
NK_INCLUDE_COMMAND_USERDATA     | Defining this adds a userdata pointer into each command. Can be useful for example if you want to provide custom shaders depending on the used widget. Can be combined with the style structures.
NK_BUTTON_TRIGGER_ON_RELEASE    | Different platforms require button clicks occurring either on buttons being pressed (up to down) or released (down to up). By default this library will react on buttons being pressed, but if you define this it will only trigger if a button is released.
NK_ZERO_COMMAND_MEMORY          | Defining this will zero out memory for each drawing command added to a drawing queue (inside nk_command_buffer_push). Zeroing command memory is very useful for fast checking (using memcmp) if command buffers are equal and avoid drawing frames when nothing on screen has changed since previous frame.
NK_UINT_DRAW_INDEX              | Defining this will set the size of vertex index elements when using NK_VERTEX_BUFFER_OUTPUT to 32bit instead of the default of 16bit
NK_KEYSTATE_BASED_INPUT         | Define this if your backend uses key state for each frame rather than key press/release events
NK_IS_WORD_BOUNDARY(c)          | Define this to a function macro that takes a single nk_rune (nk_uint) and returns true if it's a word separator. If not defined, uses the default definition (see nk_is_word_boundary())

!!! WARNING
    The following flags will pull in the standard C library:
    - NK_INCLUDE_DEFAULT_ALLOCATOR
    - NK_INCLUDE_STANDARD_IO
    - NK_INCLUDE_STANDARD_VARARGS

!!! WARNING
    The following flags if defined need to be defined for both header and implementation:
    - NK_INCLUDE_FIXED_TYPES
    - NK_INCLUDE_DEFAULT_ALLOCATOR
    - NK_INCLUDE_STANDARD_VARARGS
    - NK_INCLUDE_STANDARD_BOOL
    - NK_INCLUDE_VERTEX_BUFFER_OUTPUT
    - NK_INCLUDE_FONT_BAKING
    - NK_INCLUDE_DEFAULT_FONT
    - NK_INCLUDE_STANDARD_VARARGS
    - NK_INCLUDE_COMMAND_USERDATA
    - NK_UINT_DRAW_INDEX

### Constants
Define                          | Description
--------------------------------|---------------------------------------
NK_BUFFER_DEFAULT_INITIAL_SIZE  | Initial buffer size allocated by all buffers while using the default allocator functions included by defining NK_INCLUDE_DEFAULT_ALLOCATOR. If you don't want to allocate the default 4k memory then redefine it.
NK_MAX_NUMBER_BUFFER            | Maximum buffer size for the conversion buffer between float and string Under normal circumstances this should be more than sufficient.
NK_INPUT_MAX                    | Defines the max number of bytes which can be added as text input in one frame. Under normal circumstances this should be more than sufficient.

!!! WARNING
    The following constants if defined need to be defined for both header and implementation:
    - NK_MAX_NUMBER_BUFFER
    - NK_BUFFER_DEFAULT_INITIAL_SIZE
    - NK_INPUT_MAX

### Dependencies
Function    | Description
------------|---------------------------------------------------------------
NK_ASSERT   | If you don't define this, nuklear will use <assert.h> with assert().
NK_MEMSET   | You can define this to 'memset' or your own memset implementation replacement. If not nuklear will use its own version.
NK_MEMCPY   | You can define this to 'memcpy' or your own memcpy implementation replacement. If not nuklear will use its own version.
NK_INV_SQRT | You can define this to your own inverse sqrt implementation replacement. If not nuklear will use its own slow and not highly accurate version.
NK_SIN      | You can define this to 'sinf' or your own sine implementation replacement. If not nuklear will use its own approximation implementation.
NK_COS      | You can define this to 'cosf' or your own cosine implementation replacement. If not nuklear will use its own approximation implementation.
NK_STRTOD   | You can define this to `strtod` or your own string to double conversion implementation replacement. If not defined nuklear will use its own imprecise and possibly unsafe version (does not handle nan or infinity!).
NK_DTOA     | You can define this to `dtoa` or your own double to string conversion implementation replacement. If not defined nuklear will use its own imprecise and possibly unsafe version (does not handle nan or infinity!).
NK_VSNPRINTF| If you define `NK_INCLUDE_STANDARD_VARARGS` as well as `NK_INCLUDE_STANDARD_IO` and want to be safe define this to `vsnprintf` on compilers supporting later versions of C or C++. By default nuklear will check for your stdlib version in C as well as compiler version in C++. if `vsnprintf` is available it will define it to `vsnprintf` directly. If not defined and if you have older versions of C or C++ it will be defined to `vsprintf` which is unsafe.

!!! WARNING
    The following dependencies will pull in the standard C library if not redefined:
    - NK_ASSERT

!!! WARNING
    The following dependencies if defined need to be defined for both header and implementation:
    - NK_ASSERT

!!! WARNING
    The following dependencies if defined need to be defined only for the implementation part:
    - NK_MEMSET
    - NK_MEMCPY
    - NK_SQRT
    - NK_SIN
    - NK_COS
    - NK_STRTOD
    - NK_DTOA
    - NK_VSNPRINTF

## Example

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~c
// init gui state
enum {EASY, HARD};
static int op = EASY;
static float value = 0.6f;
static int i =  20;
struct nk_context ctx;

nk_init_fixed(&ctx, calloc(1, MAX_MEMORY), MAX_MEMORY, &font);
if (nk_begin(&ctx, "Show", nk_rect(50, 50, 220, 220),
    NK_WINDOW_BORDER|NK_WINDOW_MOVABLE|NK_WINDOW_CLOSABLE)) {
    // fixed widget pixel width
    nk_layout_row_static(&ctx, 30, 80, 1);
    if (nk_button_label(&ctx, "button")) {
        // event handling
    }

    // fixed widget window ratio width
    nk_layout_row_dynamic(&ctx, 30, 2);
    if (nk_option_label(&ctx, "easy", op == EASY)) op = EASY;
    if (nk_option_label(&ctx, "hard", op == HARD)) op = HARD;

    // custom widget pixel width
    nk_layout_row_begin(&ctx, NK_STATIC, 30, 2);
    {
        nk_layout_row_push(&ctx, 50);
        nk_label(&ctx, "Volume:", NK_TEXT_LEFT);
        nk_layout_row_push(&ctx, 110);
        nk_slider_float(&ctx, 0, &value, 1.0f, 0.1f);
    }
    nk_layout_row_end(&ctx);
}
nk_end(&ctx);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

![](https://cloud.githubusercontent.com/assets/8057201/10187981/584ecd68-675c-11e5-897c-822ef534a876.png)

## API

