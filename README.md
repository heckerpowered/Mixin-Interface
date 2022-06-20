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
- [Base type to string conversion (supports bool and some container)](#E-ToString)
- [Convert integer to string with hex format](#E-IntegerToHex)
- [Efficient string concatenation (60x faster than stringstream string concatenation)](#E-StringConcatenation)
- [Convert multiple values to strings and concatenate](#E-Concat)
- [Coroutine](#E-Coroutine)
- [Output string to console (using Win32 Api)](#E-Print)
- [Calculate crc32 value (can be a memory location, or a value)](#E-Crc32)
- [Integrated LazyImporter](https://github.com/JustasMasiulis/lazy_importer)
- [Access to local process](#E-Process)
- [Determine current Windows version](#E-WindowsVerison)
- [Detect any debugger based on Windows debugging system attached to this process. (Only available on x64 platform)](#E-DetectDebugger)
- [Store values in binary format to a file or read binary values from a file and convert them to a specified type (requires values to have names)](#E-Settings)
- [Brings down the system in a controlled manner (BSOD but not active instantly, it is not considered as a virus by any anti-virus software)](#E-BSOD)
- [Launch Mixin-Service with a GUI request with Chinese, requesting elevated privileges when possible (need to define **\_\_mixin_link_comctl32\_\_** macro to link the required libraries)](#E-MixinLauncher)
- [Access to local processor and enables you to query basic information of local processors](#E-Processors)

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
> This feature is not yet fully implemented

```` 
mixins::println(__literal("Literal"));
```` 
***
<a name="E-ToString">Base type to string conversion</a>
> This feature support convert standard container to string.
> See concept container, the only exception is string, string is not considered a container by mixins::to_string function.
> String value in the container will be quoted.
```` 
std::vector<int> v{1, 2, 3, 4};
auto result = mixins::to_string(v); // [1,2,3,4]
std::vector<std::string> v{"1", "2", "3", "4"};
auto result = mixins::to_string(v); // ["1","2","3","4"]
```` 
***
<a name="E-IntegerToHex">Convert integer to string with hex format</a>
````
mixins::println(mixins::hex(114514)); // 1BF52
````
***
<a name="E-StringConcatenation">Efficient string concatenation</a>
> This feature is expected to be 60x efficient than stringstream.
> Compare to "mixins::concat", this feature can only connect strings.
````
mixins::connect("114514", "1919810"); // 1145141919810
````
***
<a name="E-Concat">Convert multiple values to strings and concatenate</a>
> This feature is expected to be 60x efficient than stringstream.
````
mixins::concat("114514", 1919810); // 1145141919810
````
***
<a name="E-Coroutine">Coroutine</a>
> This feature is a simple implementation of Coroutine, and may be deprecated once C++23 adds the concurrent library.
````
mixins::generator<unsigned> inline coroutine(unsigned value) noexcept
{
	co_yield value++;
	co_yield value++;
	co_return;
}

int main()
{
	auto generator = coroutine(1);
	mixins::println(generator()); // 1
	mixins::println(generator()); // 2
	mixins::println(generator()); // 2
	mixins::println(generator()); // Error, undefined behavior
}
````
***
<a name="E-Print">Output string to console</a>
> This feature uses Win32 Api, so it has a high efficiency.
> This feature will not be deprecated after std::print and std::println are added to the C++23 standard library, any formatting may have security vulnerabilities.
> Other than that, formatting is a design mistake, so I would never recommend using format, nor would I use it, or add it to my own library.
````
mixins::print("114514", "1919810");
mixins::println("114514", "1919810"); // Auto ends line.
````
***
<a name="E-Crc32">Calculate crc32 value</a>
> This feature can accept a memory location and size, or a left value
````
int value = 114514;
mixins::println(mixins::compute_crc32(&value, sizeof(value))); // 1239593146
mixins::println(mixins::compute_crc32(value)); // 1239593146
````
***
<a name="E-Process">Access to local process</a>
> This example shows only a partial use of this feature
````
for(auto&& process : mixins::process::enumerate())
{
	mixins::println(process.name());
}
````
***
<a name="E-WindowsVerison">Determine current Windows version</a>
> This feature can also determine if this Windows is a Windows Server.
````
mixins::println(mixins::is_windows_xp_or_greater());
mixins::println(mixins::is_windows_xp_sp1_or_greater());
mixins::println(mixins::is_windows_xp_sp2_or_greater());
mixins::println(mixins::is_windows_xp_sp3_or_greater());
mixins::println(mixins::is_windows_vista_or_greater());
mixins::println(mixins::is_windows_vista_sp1_or_greater());
mixins::println(mixins::is_windows_vista_sp2_or_greater());
mixins::println(mixins::is_windows_7_or_greater());
mixins::println(mixins::is_windows_7sp1_or_greater());
mixins::println(mixins::is_windows_8_or_greater());
mixins::println(mixins::is_windows_81or_greater());
mixins::println(mixins::is_windows_10_or_greater());
mixins::println(mixins::is_windows_xp_or_2k());
mixins::println(mixins::is_windows_server());
````
***
<a name="Detect Debugger">Detect any debugger based on Windows debugging system attached to this process</a>
> This feature is only enabled on 64-bit platforms.
> This feature not only detects the user mode debugger, it also detects the kernel debugger attached to the system, even if it is temporarily disconnected for some reason.
````
mixins::println(mixins::detect_debugger());
````
***
<a name="E-Settings">Store values in binary format to a file or read binary values from a file and convert them to a specified type</a>
> You need to assign a name to each stored value
````
mixins::setting_manager manager{ "Values" };
manager.write<int>("Mixin", 114514);
mixins::println(manager.read<int>("Mixin"));
````
***
<a name="E-BSOD">Brings down the system in a controlled manner</a>
> This feature does not take effect immediately, it will brings down the system in a controlled manner after a short delay (uncontrolled).
> This feature will not be considered as binging by any anti-virus software.
> This feature requires a valid error code, otherwise it will generate a pop-up and fail to make the system BSOD.
````
mixins::bug_check(STATUS_ACCESS_VIOLATION);
````
***
<a name="E-MixinLauncher">Launch Mixin-Service with a GUI request with Chinese</a>
> This feature need to define **\_\_mixin_link_comctl32\_\_** macro to link the required libraries.
> If the user chooses to restart with other credentials on the GUI, this feature restarts the current process with elevated privileges and calls std::terminate();
> This feature will not fail due to insufficient privileges.
````
mixins::request_init_mixin();
````
***
<a name="E-Processors">Access to local processor and enables you to query basic information of local processors</a>
````
mixins::println("Vendor: ", mixins::processor::vendor()); // e.g. GenuineIntel
mixins::println("Brand: ", mixins::processor::brand()); // e.g. Genuine Intel(R) CPU 2.80GHz
for (auto&& feature : mixins::processor::features())
{
	mixins::println(mixins::processor::feature_name(feature.first), " ", feature.second);
}
````
