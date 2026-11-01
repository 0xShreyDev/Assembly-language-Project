# x86-64 Assembly Practice (NASM, Linux)

Assembly language seekhne ke dauran banaye gaye chhote programs. Sab programs **NASM** syntax me hain aur **Linux x86-64** syscalls use karte hain.

## Requirements

- `nasm` (assembler)
- `ld` / `gcc` (linker)
- `gdb` (debugging)

```bash
sudo apt install nasm gcc gdb
```

## Programs

| # | File | Kya karta hai |
|---|------|---------------|
| 1 | `hello.asm` | "hello world" print karta hai, exit code 22 ke saath |
| 2 | `name.asm` | Naam input leke `hello, <name>` print karta hai |
| 3 | `fullname.asm` | First aur last name input leke print karta hai |
| 4 | `math.asm` | add, sub, imul, idiv ke basic examples (kuch print nahi karta) |
| 5 | `key.asm` | User ki key ko original key se compare karta hai |
| 6 | `table.asm` | Number leke uska 1 se 10 tak ka table print karta hai |
| 7 | `calculator.asm` | Do numbers aur operation (`+ - * /`) leke result print karta hai |
| - | `util.asm` | Helper functions: `printstr`, `readint`, `printint`, `readstr`, `endl`, `exit0` |

> Programs 6 aur 7 `%include "util.asm"` use karte hain, to `util.asm` same folder me honi chahiye.

## Build aur Run

`_start` wale programs (1, 2, 3, 5, 6, 7):

```bash
nasm -f elf64 file.asm -o file.o
ld file.o -o file
./file
```

`main` wala program (4, `math.asm`) gcc se link hota hai:

```bash
nasm -f elf64 math.asm -o math.o
gcc -no-pie -nostartfiles math.o -o math
```

Ya `main` ko `_start` bana ke upar wale `ld` command se link karo.

## Program details

### 1. hello.asm
`write` syscall (`rax=1`) se string print hoti hai, phir `exit` syscall (`rax=60`) se program khatam. Exit code dekhne ke liye `echo $?`.

### 2. name.asm
`read` syscall (`rax=0`) se input buffer me aata hai. `read` ka return value (`rax`) bytes ki length hota hai, jo `write` ko length ke taur par diya jata hai.

### 3. fullname.asm
Do inputs, dono ki length alag registers me save karni padti hai. Enter (`\n`) bhi buffer me aata hai, isliye first name ki length se 1 kam karke ek line me print kiya jata hai.

### 4. math.asm
- `add`, `sub`, `imul rbx, rax, 2`
- Division: `idiv rbx` ke pehle `rax` me dividend rakho aur `cqo` se `rdx` set karo. `rdx:rax` 128-bit dividend hota hai, warna "floating point exception" (divide overflow) aata hai.

Result dekhne ke liye gdb:

```bash
gdb ./math
(gdb) break _exit
(gdb) run
(gdb) info registers rax rdx
```

### 5. key.asm
Key: 19 characters. Steps:
1. Input read karo, `dec rax` se trailing `\n` hatao
2. Length check (`cmp rax, orignal_key_len`)
3. `cld`, phir `repe cmpsb` se poori 19 bytes compare
4. Match to "Access Granted!", warna "Access Denied"

Test:

```bash
echo "1734-1530-5836-7598" | ./key
```

Key binary me plain text me hai, `strings key` se nikal sakti hai. Reverse engineering ka accha practice example.

### 6. table.asm
Number read karke loop (1 se 10) chalata hai: `number * rbx` print karta hai.

### 7. calculator.asm
Do numbers aur operator (ASCII: `+`=43, `-`=45, `*`=42, `/`=47) leke `cmp` aur `je` se sahi operation par jump karta hai.

## Key concepts

- **Syscalls**: `rax` = syscall number, args `rdi`, `rsi`, `rdx`. (`read`=0, `write`=1, `exit`=60)
- **Sections**: `.text` (code), `.data` (initialized data), `.bss` (uninitialized buffers)
- **`equ $-label`**: string ki length compile time par nikalna
- **`div` / `idiv`**: `rdx:rax` ko divide karta hai, pehle `xor rdx,rdx` (unsigned) ya `cqo` (signed)
- **`repe cmpsb`**: `rcx` bytes tak string compare, `cld` se direction forward
- **Caller-saved registers**: `syscall` ke baad `rcx` aur `r11` badal jate hain

## Known issues / TODO

- `fullname.asm`: dono names ke beech space aur alag length handling add karni hai
- `calculator.asm`: operator ko byte compare karo (`cmp byte [user_operation], '+'`), division me `cqo` use karo, zero se divide check karo, `error_msg` ke end me `,0` lagao
- `key.asm`: `cld` add karna safe rahega

## Debugging tips

```bash
gdb ./prog
(gdb) break _start
(gdb) run
(gdb) info registers
(gdb) x/20c &buffer      # buffer ke characters dekho
(gdb) x/i $rip           # current instruction
```

## Folder structure

```
asm-practice/
├── README.md
├── util.asm
├── hello.asm
├── name.asm
├── fullname.asm
├── math.asm
├── key.asm
├── table.asm
└── calculator.asm
```
