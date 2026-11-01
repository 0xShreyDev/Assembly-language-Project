# Assembly Language Practice (x86-64, NASM, Linux)

Assembly language seekhne ke dauran banaye gaye chhote projects. Sab programs **NASM** syntax me hain aur **Linux x86-64** syscalls use karte hain. Final project ek **calculator** hai.

## Projects

| Folder | Description |
|--------|-------------|
| `helloWorld` | Hello, world print karna (`write` aur `exit` syscall) |
| `userInput` | User se naam input leke `hello, <name>` print karna |
| `multiInput` | Multiple inputs lena (first name + last name) aur ek line me print karna |
| `math` | Assembly me calculation: `add`, `sub`, `imul`, `idiv` |
| `condition` | Conditions check karna (`cmp`, `je`, `jne`). Key checker: user ki key original key se compare hoti hai |
| `tableLoop` | Loop se multiplication table print karna |
| `calculater` | **Final project**: do numbers aur operator (`+ - * /`) leke result print karta hai |

> `tableLoop` aur `calculater` `%include "util.asm"` use karte hain (helper functions: `printstr`, `readint`, `printint`, `readstr`, `endl`, `exit0`). `util.asm` unke saath ya root folder me honi chahiye.

## Requirements

```bash
sudo apt install nasm gcc gdb
```

## Build aur Run

`_start` wale programs:

```bash
nasm -f elf64 file.asm -o file.o
ld file.o -o file
./file
```

Agar `util.asm` root folder me hai aur program subfolder me:

```bash
nasm -f elf64 -I ../ file.asm -o file.o
ld file.o -o file
./file
```

`math` folder ka program `main` use karta hai, to gcc se link hoga:

```bash
nasm -f elf64 math.asm -o math.o
gcc -no-pie -nostartfiles math.o -o math
```

## Key concepts

- **Syscalls**: `rax` = number, args `rdi`, `rsi`, `rdx`. (`read`=0, `write`=1, `exit`=60)
- **Sections**: `.text` (code), `.data` (initialized data), `.bss` (buffers)
- **`equ $-label`**: string ki length compile time par
- **`read` ka `\n`**: Enter bhi buffer me aata hai, isliye length se 1 kam (`dec`) karte hain
- **`div` / `idiv`**: `rdx:rax` ko divide karta hai. Pehle `xor rdx,rdx` (unsigned) ya `cqo` (signed), warna "floating point exception"
- **`repe cmpsb`**: `rcx` bytes tak string compare, `cld` ke saath
- **`syscall` ke baad** `rcx` aur `r11` badal jate hain

## Known issues / TODO

- `multiInput`: dono names ke beech space, aur alag length registers (`r12`, `r13`)
- `calculater`: operator ko byte compare karo (`movzx rdi, byte [user_operation]`), division me `cqo` use karo, zero se divide check, `error_msg` ke end me `,0`
- `condition`: `cld` add karo

## Debugging

```bash
gdb ./prog
(gdb) break _start
(gdb) run
(gdb) info registers
(gdb) x/20c &buffer
(gdb) x/i $rip
```
