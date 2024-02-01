# Implementation Notes

- CMake build system
- Use C++20 (supported on all desktop + emscripten)
- -fno-rtti and -fno-exceptions
- Runtime can use SDL for most OS abstractions
- Implement all built-in drawing routines in software
- Would SDL_Renderer be good enough for graphics? Probably if we don't want any post-effects for CRT.
- Offer as a retroarch core too? How do we do that

Standalone builds can potentially offer a simple UI for selecting games from certain locations. Bundled builds (published with a game already) will just load that game's binary.

Each instruction will execute in a set number of cycles. With a specific clock rate for the virtual CPU, we can then emulate how many instructions per second can be executed. Each instruction increments a counter to introduce fake lag.

BIOS calls (kinda like GBA or DOS) can be implemented through invoking a specific interrupt with parameters.

Can use bitfield structs to structurize the opcode decoding, for example:
```cpp
struct instruction_loadh_rl {
    constexpr opcode_t Opcode = 0x04;

    static instruction_loadh_rl decode(uint32_t data) {
        return reinterpret_cast<instruction_loadh_rl>(data);
    }

    static void execute(vm_ctx& vm) {
        auto instruction = decode(vm.fetch());
        // etc.
    }

    opcode_t opcode() const { return _opcode; }
    bool useHighHalfword() const { return _halfwordOffset > 0; }
    register_t sourceRegister() const { return _sourceRegisterIndex; }
    register_t destRegister() const { return _destRegisterIndex; }
    int16_t sourceAddressOffset() const { return _sourceOffset; }

private:
    int32_t _opcode : 7;
    int32_t _halfwordOffset : 1;

    int32_t _sourceRegisterIndex : 4;
    int32_t _destRegisterIndex : 4;

    int32_t _sourceOffset : 16;
};
```