# **MachJacker 🏴‍☠️**

<p align="center">
  <img src="MachJacker.png" alt="MachJacker logo" width="500" height="auto" />
</p>

**A modern process injection tool for macOS (ARM64 / Apple Silicon)**

MachJacker is a proof-of-concept utility that leverages the Mach kernel API to inject and execute code within the memory space of other running applications on Apple Silicon Macs. It serves as an educational tool for exploring the low-level mechanics of macOS process manipulation.

## **How It Works**

MachJacker performs a classic form of process injection by directly manipulating the target process at the kernel level. This method bypasses many higher-level security controls.

The injection process follows these steps:

1. **Target Identification**: The tool first inspects the target application's Info.plist to retrieve its **Bundle Identifier**. It then queries the system for a running process matching that identifier to get its **Process ID (PID)**.  
2. **Acquire Task Port**: It calls task\_for\_pid() to request a handle (a "task port") for the target process from the Mach kernel. This is the key that unlocks control over the target. **This step requires root privileges.**  
3. **Allocate Memory**: Using the acquired task port, it calls mach\_vm\_allocate() to reserve two regions of memory inside the target process: one for the shellcode to be executed, and another to serve as a stack for the new thread.  
4. **Write and Protect**: The shellcode is written into the allocated memory region using mach\_vm\_write(). The memory permissions are then adjusted with vm\_protect(): the code region is marked as **Readable \+ Executable**, and the stack region is marked as **Readable \+ Writable**.  
5. **Create and Execute Thread**: Finally, thread\_create\_running() is called to create and immediately start a new thread within the target process. The thread's initial state is configured so that its instruction pointer points directly at the start of the injected shellcode, beginning its execution.

## **Usage**

MachJacker must be compiled and run from the command line with root privileges.

\# Compile the tool  
clang \-o machjacker \-framework Foundation \-framework AppKit machjacker.m

\# Run with sudo  
sudo ./machjacker \<path\_to\_app\> \<path\_to\_shellcode\>

**Example:** Injecting a payload into the running Calculator app.

\# Assume shellcode.bin contains a simple payload (e.g., spawns a shell)  
sudo ./machjacker /System/Applications/Calculator.app ./shellcode.bin

## **🗺️ Roadmap**

This tool is currently in its early stages. The primary goal is to evolve it from a simple shellcode injector into a more versatile and powerful dynamic library injector.

* \[x\] **Core Shellcode Injection (ARM64)**  
  * Successfully inject and run raw shellcode in a remote process.  
  * Robust error handling and logging.  
* \[ \] **Dynamic Library (.dylib) Injection**  
  * **Phase 1: Research & Scaffolding**  
    * Implement logic to find the base address of libdyld.dylib in the remote process.  
    * Implement a function to find the address of the exported dlopen symbol within the remote process's instance of libdyld.dylib.  
  * **Phase 2: Payload Construction**  
    * Modify the injection logic to write the path of the .dylib to be injected into the target process's memory.  
    * Instead of custom shellcode, create a small "stub" shellcode that sets up the correct arguments (the dylib path) in the appropriate registers (x0, x1, etc.) and then calls the address of dlopen.  
  * **Phase 3: Implementation & Testing**  
    * Update the main function to accept a .dylib path instead of a shellcode file.  
    * Use the thread creation technique (thread\_create\_running) to execute the dlopen stub, effectively forcing the target process to load the malicious library.  
* \[ \] **Flexible Target Selection**  
  * Add command-line flags to target processes by PID (-p \<pid\>) or process name (-n \<name\>) in addition to the app bundle path.

## **Disclaimer**

This tool is intended for **educational and research purposes only**. Running arbitrary code inside other processes can cause system instability and security vulnerabilities. Do not use this tool on systems you do not own or have explicit permission to test.

