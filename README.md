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

### Features
- Compile time string encryption (run-time decryption)
- Concepts
- Format error message.
- Automatic management of Windows handles, works similarly to "unique_ptr".
- Compile time random generator.
- Compile time string (Not complete yet)
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
- Launch Mixin-Service with a GUI request with Chinese, requesting elevated privileges when possible (need to define __mixin_link_comctl32__ macro to link the required libraries)
- Access to local processor and enables you to query basic information of local processors.