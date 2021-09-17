# gdnative-handies

This repo contains three useful and handy files in working with Godot GDNative C++.

### SConstruct
First file is an SConstruct file that provides building GDNative C++ for linux, windows, android and web (wasm file, emscripten is needed). Useful command line options are defined to make building easier. use `scons -h` or `scons --help` to see help text for these options.

### VSCode build and launch file
`tasks.json` and `launch.json` are useful if VSCode is used and they are used to configure automatic building and running target Godot project. Copy them into `.vscode` folder in root of VSCode project folder. Set path to your Godot porject root folder in `launch.json` (use `godot --help` to see Godot run options) and scons options can be set in `task.json` file.
