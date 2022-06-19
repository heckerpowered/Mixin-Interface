# Mixin-Interface

## Introduction
Mixin-Service is composed of two parts, Mixin-Service and Mixin Template Library.

## Mixin-Service
Mixin-Service is an kernel-driver, Mixin-Service is designed to be easy to defend against rootkit.

### Features
- Intel VT-x based hypervisor (a.k.a virtual machine monitor) (Not completely available yet)
- Read/Write virtual/physical memory (May used to bypass memory protection based on fake cr3 in the future.)
- Secure virtual memory (Prohibit releasing or changing the protection properties of memory pages to more stringent protection)
- Allocate/Free virtual memory.
- [Process guard (Provides 4 levels of guard)](#ProcessGuardLevels)
- Debugger detacher (Only available for debuggers based on Windows debugging system)
- Change handle access
- Instantly shutdown/restart system.
- Terminate process
- Open process (May not bypass access checks) 

<a name="ProcessGuardLevels">Process Guard Levels</a>
- Disable - Disable any guards applied to the specified process.
- [Basic](#GuardLevelBasic) - Enable minimal safeguards on the specified process, affecting but not limited to the following actions.
- [Strict](#GuardLevelStrict) - This guard may prevent the process from loading, it is recommended to enable this level of guard after the process has finished loading.
- [Highest](#GuardLevelHighest) - This level of guard may affect the normal working of the process.

<a name="GuardLevelBasic">Basic</a>
- Make the handle lose any privileges when the handle is opened/created.
- Hide guarded processes in the process list.
- Disable some SSDT functions.

<a name="GuardLevelStrict">Strict</a>

Compared to the "basic" level of guard, enabling the "strict" level of guard disables the following actions:
- Allocate code pages
- Change the properties of the code page to writable or change the properties of the normal page to executable.
- Load dlls without Microsoft signature.
- Create remote threads and block all access from the user
Some of these operations may not work on lower versions of the operating system

<a name="GuardLevelHighest">Highest</a>

Compared to the "strict" level of guard, enabling the "highest" level of guard disables the following actions:
- Create threads.
- Load any dll
- All SSDT functions
- All user-mode APC delivery to any threads of guarded process

## Mixin Template Library
Mixin Template Library is an header-only library and is designed to be easy to use, have no security vulnerability and efficient.

### Requirements
Mixin Template Library requires C++ 20, and is tested only on the MSVC compiler.
> Some features may not available in unicode

### Features
- [Compile time string encryption (run-time decryption)](#E-CompileTimeStringEncryption)
- [Concepts](#E-Concepts)
- [Format error message](#E-FormatErrorMessage)
- [Automatic management of Windows handles, works similarly to "unique_ptr"](#E-UniqueHandle)
- [Compile time random generator](#E-CompileTimeRandomGenerator)
- [Compile time string (Not complete yet)](#E-CompileTimeString)
- Base type to string conversion (supports bool and some container)
- Convert integer to string with hex format.
- Efficient string concatenation (60x faster than stringstream string concatenation)
- Convert multiple values to strings and concatenate
- Coroutine (generator)
- Output string to console (using Win32 Api)
- Calculate crc32 value (can be a memory location, or a value)
- [Integrated LazyImporter](https://github.com/JustasMasiulis/lazy_importer)
- Access to local process.
- Determine current Windows version.
- Detect any debugger based on Windows debugging system attached to this process. (Only available on x64 platform)
- Store values in binary format to a file or read binary values from a file and convert them to a specified type (requires values to have names)
- Brings down the system in a controlled manner. (BSOD but not active instantly, it is not considered as a virus by any anti-virus software)
- Launch Mixin-Service with a GUI request with Chinese, requesting elevated privileges when possible (need to define **\_\_mixin_link_comctl32\_\_** macro to link the required libraries)
- Access to local processor and enables you to query basic information of local processors.

### Examples
<a name="E-CompileTimeStringEncryption">Compile time string encryption</a>
```` 
mixins::println(__xor_string("Literal"));
````
***
<a name="E-Concepts">Concepts</a>
```` 
template<mixins::container container_t> // Accept any container type
constexpr container_t::size_type size(container_t const& container) noexcept
{
  return container.size();
}
```` 
***
<a name="E-FormatErrorMessage">Format error messsage</a>
```` 
mixins::println(mixins::last_error());
```` 
***
<a name="E-UniqueHandle">Automatic management of Windows handles</a>
> works similarly to "unique_ptr"
````
mixins::handle mutex_handle{ CreateMutexA(nullptr, false, "Mixin-Mutex") };

//
// -1 or -2 is not considered an invalid handle in process_handle, process_handle can represent both process or thread handles.
//
mixins::process_handle process_handle{ GetCurrentProcess() };
mixins::process_handle process_handle{ GetCurrentThread() };

mixins::actx_handle actx_handle;
GetCurrentActCtx(&actx_handle);

mixins::reg_handle reg_handle;
RegOpenKeyExA(HKEY_LOCAL_MACHINE, "SYSTEM\\CurrentControlSet\\Services\\heckerpowered.mixin", 0, KEY_READ, &reg_handle);

auto file_mapping = CreateFileMappingA(INVALID_HANDLE_VALUE, NULL, PAGE_READWRITE, 0, 4096, "Mixin-Mapping");
mixins::mapping_handle mapping_handle { MapViewOfFile(file_mapping, FILE_MAP_ALL_ACCESS, 0, 0, 4096) };
```` 
***
<a name="E-CompileTimeRandomGenerator">Compile time random genrator</a>
> The random values generated by random generator may be the same in a short period of time
```` 
mixins::println(mixins::random_generator<0>::value);
```` 
***
<a name="E-CompileTimeString">Compile time string</a>
```` 
> This feature is not yet fully implemented

```` 
mixins::println(\_\_literal("Literal"));
```` 
