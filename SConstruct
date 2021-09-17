import os
import sys
from SCons.Executor import execute_nothing
from SCons.Script import *

sys.tracebacklimit = 0

# Try to detect the host platform automatically.
# This is used if no `platform` argument is passed
if sys.platform.startswith('linux'):
    host_platform = 'linux'
elif sys.platform == 'darwin':
    host_platform = 'osx'
elif sys.platform == 'win32' or sys.platform == 'msys':
    host_platform = 'windows'
else:
    raise ValueError(
        'Could not detect host platform automatically, please specify with '
        'host_platform=<platform>'
    )


colors = {}
def no_verbose():
    if sys.stdout.isatty():
        colors["purple"] = "\033[95m"
        colors["blue"] = "\033[94m"
        colors["green"] = "\033[92m"
        colors["yellow"] = "\033[93m"
        colors["end"] = "\033[0m"
    else:
        colors["purple"] = ""
        colors["blue"] = ""
        colors["green"] = ""
        colors["yellow"] = ""
        colors["end"] = ""
    
    compile_shared_source_message = "{}Compiling shared ==> {}$SOURCE{}"\
        .format(colors["green"], colors["yellow"], colors["end"])
    link_shared_library_message = "{}Linking Shared Library ==> {}$TARGET{}"\
        .format(colors["blue"], colors["yellow"], colors["end"])
    
    platform_print_prefix = "{}[{} {}] ".format(colors["purple"], env['platform'],
        env['android_arch'] if env['platform'] == "android" else env['bits'])
    
    env["SHCCCOMSTR"] = platform_print_prefix + compile_shared_source_message
    env["SHCXXCOMSTR"] = platform_print_prefix + compile_shared_source_message
    env["ARCOMSTR"] = platform_print_prefix + link_shared_library_message
    env["RANLIBCOMSTR"] = platform_print_prefix + link_shared_library_message
    env["SHLINKCOMSTR"] = platform_print_prefix + link_shared_library_message
    env["LINKCOMSTR"] = platform_print_prefix + link_shared_library_message


options = Variables([], ARGUMENTS)
options.Add(EnumVariable(
    key='platform',
    help='Specifies target platform to build for',
	default='none',
    allowed_values=['none', 'linux', 'android', 'windows', 'javascript'],
    ignorecase=2
))
options.Add(EnumVariable(
    key='bits',
    help='Target platform bits',
    default=64,
    allowed_values=['32', '64']
))
options.Add(PathVariable(
    key='gd_library_dir',
    help='Path to .a library built from godot-cpp bindings.',
    default=os.environ.get('GD_LIBRARY_DIR'),
    validator=PathVariable.PathIsDir
))
options.Add(PathVariable(
    key='gd_library_name',
    help="""Name of the .a library generating from godot-cpp headers. By
     default it uses name which is used in godot-cpp SConstruct.""",
    default=None,
    validator=PathVariable.PathIsFile
))
options.Add(EnumVariable(
    key='android_arch',
    help='Target android architecture',
    default='armv7',
    allowed_values=['armv7', 'arm64v8', 'x86', 'x86_64']
))
options.Add(PathVariable(
    key='ANDROID_NDK_ROOT',
    help="""Path to your Android NDK installation. By default, uses
        ANDROID_NDK_ROOT from your environment variables.""",
    default=os.environ.get('ANDROID_NDK_ROOT', None),
    validator=PathVariable.PathIsDir
))
options.Add(EnumVariable(
    key='target',
    help='release or debug target',
    default='debug',
    allowed_values=['release', 'debug']
))
options.Add(
    key='android_api_level',
    help='Target Android API level',
    default='18' if ARGUMENTS.get("android_arch", 'armv7') in [
        'armv7', 'x86'] else '21'
)
options.Add(PathVariable(
    key='gd_headers_dir',
    help='Path to godot includes.',
    default=os.environ.get('GD_HEADERS_DIR'),
    validator=PathVariable.PathIsDir
))
options.Add(
	key='add_includes',
	help='Additional include directories to be added to compiler option. '
    'Separate multiple values by <:>',
	default=''
)
options.Add(
    key='library_dir',
    help='Path to desired out put foldel.',
    default='../lib/'
)

env = Environment(CXXFLAGS="-std=c++17", sources=[], ENV=os.environ)
options.Update(env=env)
Help(options.GenerateHelpText(env))

if env.GetOption("help"):
    Return()

if ARGUMENTS.get("VERBOSE") != "1":
    no_verbose()

var_dir = 'src/build/obj/' + env['platform']
if env['platform'] == 'android':
    var_dir += '/' + env['android_arch']
elif env['platform'] in ['windows', 'linux']:
    var_dir += '/' + env['bits']

VariantDir(var_dir, 'Src', duplicate=0)


if env['platform'] == 'none':
    raise ValueError("""\n\tYou must specify target platform. Specify it by 
        platform=<platform>. <platform> can only be 'linux', 'android'.""")

if 'gd_library_dir' not in env:
    raise ValueError("""\n\tYou must specify the path to the directory containing 
        godot .a library. Specify it by gd_library_dir=<dir>""")

if 'gd_library_name' not in env:
    suffix = env['bits']
    if env['platform'] == 'android':
        suffix = env['android_arch']
    elif env['platform'] == 'javascript':
        suffix = 'wasm'
    env['gd_library_name'] = 'godot-cpp.{}.{}.{}'.format(
        env['platform'],
        env['target'],
        suffix,
    )

if env['platform'] == 'linux':
    print('*** Building for Linux ***')
    env.Append(LINKFLAGS=["-Wl,-rpath,'$$ORIGIN'"])
    if env['target'] == 'debug':
        env.Append(CCFLAGS = ['-g3','-Og'])
    elif env['target'] == 'release':
        env.Append(CCFLAGS = ['-O3'])
        env.Append(LINKFLAGS = ['-s', '-flto'])

    if env['bits'] == '64':
        env.Append(CCFLAGS=['-m64'])
        env.Append(LINKFLAGS=['-m64'])
    elif env['bits'] == '32':
        env.Append(CCFLAGS=['-m32'])
        env.Append(LINKFLAGS=['-m32'])
elif env['platform'] == 'windows':
    print('*** Building for Windows ***')
    if host_platform == 'windows':
        # MSVC
        env.Append(LINKFLAGS=['/WX'])
        if env['target'] == 'debug':
            env.Append(CCFLAGS=['/Z7', '/Od', '/EHsc', '/D_DEBUG', '/MDd'])
        elif env['target'] == 'release':
            env.Append(CCFLAGS=['/O2', '/EHsc', '/DNDEBUG', '/MD'])

    elif host_platform == 'linux' or host_platform == 'osx':
        # Cross-compilation using MinGW
        if env['bits'] == '64':
            env['CC'] = 'x86_64-w64-mingw32-gcc'
            env['CXX'] = 'x86_64-w64-mingw32-g++'
            env['AR'] = "x86_64-w64-mingw32-ar"
            env['RANLIB'] = "x86_64-w64-mingw32-ranlib"
            env['LINK'] = "x86_64-w64-mingw32-g++"
        elif env['bits'] == '32':
            env['CC'] = 'i686-w64-mingw32-gcc'
            env['CXX'] = 'i686-w64-mingw32-g++'
            env['AR'] = "i686-w64-mingw32-ar"
            env['RANLIB'] = "i686-w64-mingw32-ranlib"
            env['LINK'] = "i686-w64-mingw32-g++"
        
        if env['target'] == 'debug':
            env.Append(CCFLAGS = ['-g3','-Og'])
        elif env['target'] == 'release':
            env.Append(CCFLAGS = ['-O3'])
            env.Append(LINKFLAGS = ['-s', '-flto'])
        
        env.Append(CCFLAGS=['-Wwrite-strings'])
        env.Append(CFLAGS=['-Wno-discarded-qualifiers'])
        env.Append(LINKFLAGS=[
            '--static',
            '-Wl,--no-undefined',
            '-static-libgcc',
            '-static-libstdc++',
        ])
elif env['platform'] == 'android':
    print('*** Building for Android ***')
    if 'ANDROID_NDK_ROOT' not in env:
        raise ValueError("""\n\tTo build for Android, ANDROID_NDK_ROOT must be
             defined. Please set ANDROID_NDK_ROOT to the root folder of your
             Android NDK installation.""")

    # Validate API level
    api_level = int(env['android_api_level'])
    if env['android_arch'] in ['x86_64', 'arm64v8'] and api_level < 21:
        print("""WARN: 64-bit Android architectures require an API level of at
             least 21; setting android_api_level=21""")
        env['android_api_level'] = '21'
        api_level = 21

    # Setup android toolchain
    if not env['ANDROID_NDK_ROOT'].endswith('/'):
        env['ANDROID_NDK_ROOT'] += '/'
    toolchain = env['ANDROID_NDK_ROOT'] + 'toolchains/llvm/prebuilt/'
    if host_platform == 'windows':
        toolchain += 'windows'
        import platform as pltfm
        if pltfm.machine().endswith('64'):
            toolchain += '-x86_64'
    elif host_platform == "linux":
        toolchain += "linux-x86_64/"

    # Get architecture info
    arch_info_table = {
        "armv7": {
            "march": "armv7-a", "compiler_path": "armv7a-linux-androideabi"
        },
        "arm64v8": {
            "march": "armv8-a", "compiler_path": "aarch64-linux-android"
        },
        "x86": {
            "march": "i686", "compiler_path": "i686-linux-android"
        },
        "x86_64": {
            "march": "x86-64", "compiler_path": "x86_64-linux-android"
        }
    }

    toolchain += ('bin/' + arch_info_table[env['android_arch']]['compiler_path']
                  + env['android_api_level'])

    env['CC'] = toolchain + '-clang'
    env['CXX'] = toolchain + '-clang++'
elif env['platform'] == 'javascript':
    print('*** Building for Javascript ***')
    print('*** Building for Javascript ***')
    env["ENV"] = os.environ
    env["CC"] = "emcc"
    env["CXX"] = "em++"
    env["AR"] = "emar"
    env["RANLIB"] = "emranlib"
    env.Append(CPPFLAGS=["-s", "SIDE_MODULE=1", '-s', 'ASSERTIONS=1'])
    env.Append(LINKFLAGS=["-s", "SIDE_MODULE=1", '-s', 'ASSERTIONS=1'])
    env["SHOBJSUFFIX"] = ".bc"
    env["SHLIBSUFFIX"] = ".wasm"
    # Use TempFileMunge since some AR invocations are too long for cmd.exe.
    # Use POSIX-style paths, required with TempFileMunge.
    env["ARCOM_POSIX"] = env["ARCOM"].replace("$TARGET", "$TARGET.posix").replace("$SOURCES", "$SOURCES.posix")
    env["ARCOM"] = "${TEMPFILE(ARCOM_POSIX)}"

    # All intermediate files are just LLVM bitcode.
    env["OBJPREFIX"] = ""
    env["OBJSUFFIX"] = ".bc"
    env["PROGPREFIX"] = ""
    # Program() output consists of multiple files, so specify suffixes manually at builder.
    env["PROGSUFFIX"] = ""
    env["LIBPREFIX"] = "lib"
    env["LIBSUFFIX"] = ".a"
    env["LIBPREFIXES"] = ["$LIBPREFIX"]
    env["LIBSUFFIXES"] = ["$LIBSUFFIX"]
    env.Replace(SHLINKFLAGS='$LINKFLAGS')
    env.Replace(SHLINKFLAGS='$LINKFLAGS')
    


outpath = env['library_dir']
if not outpath.endswith('/'):
	outpath += "/"
if env['platform'] == 'linux':
    outpath += 'linux/'
elif env['platform'] == 'android':
    outpath += 'android/'
elif env['platform'] == 'windows':
    outpath += 'windows/'
elif env['platform'] == 'javascript':
    outpath += 'html/'

suffix = env['android_arch'] if env['platform'] == 'android' else (
    'wasm' if env['platform'] == 'javascript' else env['bits'])

library_name = "godot-taglib"
outputFile = outpath + 'lib{}.{}.{}.{}.{}'.format(
    library_name,
    env['platform'],
    env['target'],
    suffix,
    'dll' if env['platform'] == 'windows' else ('wasm' if env['platform'] == 'javascript' else 'so')
)

env['sources'] += Glob(var_dir + "/*.cpp")

Export("env")

SConscript(
    [
    ]
)


list_of_add_includes = env['add_includes'].split(sep=':')
for incl in list_of_add_includes:
	env.Append(CPPPATH=incl+':')

workspaceIncludes = ['.:', 'include:']
for incl in workspaceIncludes:
    env.Append(CPPPATH=incl)

gd_headers = env['gd_headers_dir']
if not gd_headers.endswith('/'):
    gd_headers += '/'

env.Append(CPPPATH=gd_headers+'include:')
env.Append(CPPPATH=gd_headers+'include/core:')
env.Append(CPPPATH=gd_headers+'include/gen:')
env.Append(CPPPATH=gd_headers+'godot-headers')

GodotLibrary = env.SharedLibrary(target=outputFile,
                                 source=env['sources'],
                                 LIBS=[env['gd_library_name']],
                                 LIBPATH=[env['gd_library_dir']])
Default(GodotLibrary)
