gdb scripting: gdx -x script_path to_dbg_path

b *main
commands // if main bp hit... exec
        silent //dont say anythin when hit bp
        p/a $rip
        continue
end

display command. with pwngdb or gef pretty useless

set $my_var = $another_var (eg; register,memory)

use it for patching, control flow and skip condition!
```sh {data-lang-off}
# Describes how to use a command.
help
help [command]
help info
help breakpoint
```

When starting or running a program, you can specify arguments in almost exactly the same way as you would on your shell.
For example, you can use `start <ARGV1> <ARGV2> <ARGVN> < <STDIN_PATH>`.
**Running**
```sh {data-lang-off}
start    # Starts program and breaks at main.
starti   # starts and breaks at _start:
run <arg[]>; #run providing arguments, could be used also with start btw
continue # Continue program where you left off.
kill     # Kill process.
quit     # Leave GDB.
```

```sh {data-lang-off}
## Step (into).
s/step
## Step over.
n/next
## Step (into) 1 instruction
si/stepi
## Step over one instruction.
ni/nexti
## Step out function
fin/finish
```

### Disassembly

View instructions at a function or address.
```sh {data-lang-off}
disas <address/function>
disas <start addr>,<end addr>
disas <start addr>,+<offset>
disas main
```

Enable Intel-flavoured ASM syntax.
```sh {data-lang-off}
set disassembly-flavor intel
```

View data as instructions.
```sh {data-lang-off}
x/[n]i <addr>
x/20i 0x5555555dddd0
```

## Registers

### View Registers

Show individual registers.
```sh {data-lang-off}
print [expression]

print $rax
p $rax
can also cast in print,eg : p/x *(uint64_t*) $rsp
# Expressions are evaluated.
p $rbx+$rcx*4
```

Show all registers.
```sh {data-lang-off}
info r/info registers
```

### Modify Registers
```sh {data-lang-off}
set $eax = 0xdeadbeef
```
### View Memory

```sh {data-lang-off}
x/[n][sz][fmt] <addr>

# n: Number of data to print.
# sz: b(byte), h(halfword), w(word), g(giant, 8 bytes)
# fmt: Format to print data.
# - o(octal), x(hex), d(decimal), u(unsigned decimal),
# - z(hex, zero padded on the left)
# - t(binary), f(float), c(char), s(string)
# - a(address), i(instruction),

# 20 words.
x/20wx 0x7fffffffd000

# 20 bytes.
x/20bx 0x7fffffffd000

# View as string.
x/s 0x7fffffffd000
```

### Modify Memory

```sh {data-lang-off}
set {c-type}<address> = <value>

# For self-compiled sources.
set var i = 10
set {int}0x83040 = 4
```

You can also modify memory at a pointer location by casting to an appropriate type and dereferencing.

```sh {data-lang-off}
## C++
set *{uint32_t*}0x7fffffffd000 = 0xdeadbeef
## Rust
set *(0x7fffffffd000 as *const u32) = 0xdeadbeef
```

### Search Memory

```sh {data-lang-off}
find <start>, <end>, <data...>
find <start>, +<length>, <data...>

# Find string (including null byte).
find 0x7fffffffd000, 0x7ffffffff000, "Hello world!"

# Find string (excluding null byte).
find 0x7fffffffd000, 0x7ffffffff000, 'H','e','l','l','o'
```

More options.

```sh {data-lang-off}
find [/sn] ...
# s: b(byte), h(halfword), w(word), g(giant, 8 bytes)
# n: max number of finds
```

**Stack Frame**
```sh {data-lang-off}
info frame
```

**Stack Trace**
Show a trace of previous stack frames.
```sh {data-lang-off}
backtrace
bt
```

## Breakpoints


```sh {data-lang-off}
break *<address>, symbolic break is better! break *my_foo + x
break <line-number | label> # For self-compiled programs.
break <stuff...> if <expression>
# Address.
break *0x401234
b *0x401234

# Line number and expression.
break main.c:6 if i == 5
```

Further Reading:
- [SO: GDB – Break if variable equal value](https://stackoverflow.com/q/14390256/10239789)

### Breakpoint Control

Sometimes we only want to enable or disable certain breakpoints. These commands come handy then. They also apply to watchpoints.

**Get Breakpoint Info**

```sh {data-lang-off}
info break
info b
```

**Control Breakpoints**

```sh {data-lang-off}
# Enable/disable all breakpoints.
enable
disable

# Enable/disable specific breakpoints.
enable <breakpoint-id>
disable <breakpoint-id>

# Remove breakpoints.
delete <breakpoint-id>
```

**Skip `n` Breakpoints**

```sh {data-lang-off}
continue <ignore-count>

# Skip 32 breaks.
continue 32
```

**Hit Breakpoint Once**

```sh {data-lang-off}
# Enable the breakpoint once.
# The breakpoint will be disabled after first hit.
enable once <breakpoint-id>
```

### Watchpoints

Breaks when data changes. More specifically, whenever the *value of an expression* changes, a break occurs.

This includes:
- when an address is **written** to. (`watch`, `awatch`)
- when an address is **read** from. (`rwatch`, `awatch`)
- when an expression evaluates to a given value. (`watch`)

```sh {data-lang-off}
watch <expression>

# Break on write.
watch *0x7fffffffd000

# Break on condition.
## Register
watch $rax == 0xdeadbeef
## Memory
### C/C++
watch *{uint32_t*}0x7fffffffd000 == 0xdeadbeef
### Rust
watch *(0x7fffffffd000 as *const u32) == 0xdeadbeef
```

Watchpoints can be enabled/disabled/deleted like breakpoints, but you can also list them separately.
```sh {data-lang-off}
# Displays table of watchpoints.
info watchpoint
info wat
```

If hardware watchpoints are supported, then you can also use read watchpoints and access watchpoints.

```sh {data-lang-off}
# Check if hardware watchpoints are supported.
show can-use-hw-watchpoints
```

```sh {data-lang-off}
# Read watchpoints: break on read.
rwatch *0x7fffffffd000

# Access watchpoints: break on read or write.
awatch *0x7fffffffd000
```

Further Reading:

- [GDB: Setting Watchpoints](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Set-Watchpoints.html)


### GDB Script

GDB commands can be placed in files and run in the following ways:

- `~/.gdbinit` and `./.gdbinit` are executed automatically on GDB startup.

- On the command line, with `-x`/`--command`:
  ```sh
  gdb --batch --command=test.gdb --args ./test.exe 5
  ```

- Using the `source` command in GDB:
  ```sh {data-language=GDB}
  source myscript.gdb
  ```

Further Reading:
- [GDB Reference - Command Files](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Command-Files.html)
- [GDB Scripting Commands and Examples](https://cgi.cse.unsw.edu.au/~learn/debugging/modules/gdb_init_file/)
- [SO: What are the best ways to automate a GDB debugging session?](https://stackoverflow.com/q/10748501/10239789)


### Enable ASLR

{% abbr "ASLR", "Address space layout randomisation" %} is a common mechanism to randomise stack, heap, and library offsets.

ASLR is disabled by default in GDB. To re-enable:

```sh {data-lang-off}
set disable-randomization off
```

Useful for pwn challenges.

### PIE Breakpoints

GEF only.

{% abbr "PIE", "Position-independent executable" %} are binaries where segments (.data, .text) are loaded at random offsets. In GDB, it seems to always be set to offset 0x555…554000.

Not all binaries have PIE enabled. Use `checksec` to verify.

Use the `pie` commands (`help pie`). Pie breakpoints are separate from regular breakpoints.

```sh {data-lang-off}
pie b <addr>    # PIE breakpoint at offset <addr> in code.
pie run         # Run with pie breakpoints enabled.
```
