## microfun

> This project is a C++ library and command-line tool for optimizing the loading of large map images in games. It uses a quadtree-based algorithm to split a large image into smaller tiles, with a key optimization of storing uniformly colored tiles as a color value rather than an image file. This significantly reduces the number of files and the overall storage size.

# Project Overview

This project is a C++ library and command-line tool for optimizing the loading of large map images in games. It uses a quadtree-based algorithm to split a large image into smaller tiles, with a key optimization of storing uniformly colored tiles as a color value rather than an image file. This significantly reduces the number of files and the overall storage size.

The project is divided into three main parts:

1.  **`lib`**: A C++ library containing the core logic for quadtree splitting (`QuadTreeSplitter`), tile indexing (`QuadTreeIndex`), and viewport assembling (`ViewportAssembler`).
2.  **`tools`**: Command-line tools that use the `lib` to perform actions like splitting images and checking the output for a given viewport.
3.  **`data`**: Test data, including images to be split.

# Building and Running

The project is built using CMake and can be run from the command line using the `run.sh` script.

## Building

To build the project, you can use the following commands:

```bash
mkdir build
cd build
cmake ..
make
```

## Running

The `run.sh` script provides two main commands:

*   **`split`**: Splits a large image into smaller tiles using the quadtree algorithm.
    ```bash
    ./run.sh split -i <input_image.png> -o <output_directory>
    ```
*   **`check`**: Simulates a viewport and assembles the tiles to create an image for that viewport.
    ```bash
    ./run.sh check -i <resource_directory> -p <x>,<y> -s <width>,<height>
    ```

# Development Conventions

*   **Language**: The project is written in C++.
*   **Style**: The code follows the Google C++ Style Guide.
*   **Testing**: The project uses a custom testing framework in the `tests` directory.
*   **Dependencies**: The project has a dependency on `stb_image` and `stb_image_write` for image loading and saving, which are included in the `lib/include` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/XiaoXKKK) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
