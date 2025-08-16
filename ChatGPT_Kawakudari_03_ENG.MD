**My question:**

I am studying forth-dwc. I would like to have a reference manual. Can you create one?
https://github.com/CCurl/forth-dwc

**ChatGPT:**

Thinking time: 24s

Forth-dwc Reference Manual (Draft)

**forth-dwc Reference Manual (Draft)**

Target: CCurl/forth-dwc (Last checked: 2025-08-11) License: MIT (Implementation is
C/Forth)

------------------------------------------------------------------------

**1. Overview**

- A minimal Forth implementation using the **DWC (DWORD-Code)** scheme. Standalone execution is possible, and **embedded** implementation in C/C++
is also possible.

- Implementation consists mainly of **3 files**: dwc-vm.c, dwc-vm.h, and system.c.

- There are **32** basic primitives and **13** system primitives.

- Each command is represented by 1 **DWORD** (primitive number/literal/XT in dictionary).

**2. Build and execution**

**2.1 Windows**

- dwc.sln / dwc.vcxproj for Visual Studio are included.

- **Release / Win32** is recommended for building (minimum size).

**2.2 Linux/macOS**

- Use makefile.

- Example: make → ./dwc to start.

When executed, an external interpreter (outer interpreter) starts, loads boot.fth,
 etc., and becomes interactive.

------------------------------------------------------------------------

**3. Language Mode (ColorForth Influence)**

DWC has 4
states (**State**), which are switched by **control characters in blank space**.

-----------------------------------------------------------------------
**State** **Meaning** **Default action**
---------------- ------------- ----------------------------------------
COMPILE Compile Compile words/numbers

DEFINE Definition start Add word to dictionary and go to COMPILE

INTERPRET Interpretation Execute word immediately

COMMENT Comment Ignore everything except )/))
-----------------------------------------------------------------------

**Immediate (IMMEDIATE) state transition words**

:, ;, \[, \], (, ), ((, ))

- : → DEFINE

- ; → Compile EXIT and INTERPRET

- \[ → INTERPRET

- \] → COMPILE

- (, (( → COMMENT

- ), )) → End comment (to COMPILE/INTERPRET respectively)

------------------------------------------------------------------------

**4. Code representation (DWORD-Code)**

- **0..44**: Primitive number

- **Literal**: Value with LIT_BITS set in the upper bits

- **Dictionary word**: XT (code address)

------------------------------------------------------------------------

**5. Primitive List**

**Notation**: Stack effect is (in \-- out), TOS=top of stack, NOS=next.

**5.1 DWC Primitives (0--31)**

0 exit ( \-- ) Restore RTOS to PC. Stop when PC=0.

1 lit ( \-- n ) Push next word, PC++.

2 jmp ( \-- ) PC = code\[PC\].

3 jmpz ( n \-- ) If TOS==0, PC=code\[PC\]; otherwise, PC++. Discard TOS.

4 jmpnz ( n \-- ) If TOS!=0, set PC=code\[PC\]; otherwise, PC++. Discard TOS.

5 njmpz ( n \-- n ) (Conditional branch, non-destructive) If TOS==0, set PC=code\[PC\]; otherwise,
PC++.

6 njmpnz ( n \-- n ) (conditional branch, non-destructive) If TOS!=0, PC=code\[PC\], else
PC++.

7 dup ( n \-- n n ) Duplicate.

8 drop ( n \-- ) Discard.

9 swap ( a b \-- b a ) Swap.

10 over ( a b \-- a b a ) Duplicate NOS.

11 ! ( n a \-- ) Cell storage (n to a).

12 @ (a \-- n) Read cell.

13 c! (b a \--) Store byte.

14 c@ (a \-- b) Read byte.

15 \>r (n \--) Push to return stack.

16 r@ ( \-- n ) Copy RTOS and push to data stack.

17 r\> ( \-- n ) Pop RTOS and push to data stack.

18 \* ( a b \-- c ) Multiply.

19 + ( a b \-- c ) Addition.

20 - ( a b \-- c ) Subtraction.

21 /mod ( a b \-- r q ) Quotient/remainder.

22 \< ( a b \-- f ) a\<b → 1.

23 = ( a b \-- f ) a=b → 1.

24 \> ( a b \-- f ) a\>b → 1.

25 +! ( n a \-- ) Add to the cell of a.

26 \' ( \-- a ) Get the address (dictionary) of the next word.

27 for ( n \-- ) Start FOR loop.

28 next ( \-- ) End FOR loop.

29 and ( a b \-- c ) AND.

30 or ( a b \-- c ) OR.

31 xor ( a b \-- c ) XOR.

**5.2 System Primitives (32--44)**

32 key ( \-- n ) Next key input (block).

33 ?key ( \-- n ) Key present (1/0).

34 emit ( n \-- ) Character output.

35 ztype ( a \-- ) Output NUL terminating string.

36 fopen ( nm md \-- h ) Open file in mode md (h=0 on failure).

37 fclose ( h \-- ) Close file.

38 fread ( a sz h \-- n ) Read.

39 fwrite ( a sz h \-- n ) Write.

40 ms ( n \-- ) Sleep for milliseconds.

41 timer ( \-- n ) Current time (implementation-dependent).

42 add-word ( \-- ) Add the next word to the dictionary.

43 outer ( a \-- ) Execute the outer interpreter on a.

44 system ( a \-- ) Execute \`system(a)\`.

------------------------------------------------------------------------

**6. Dictionary and language features**

**6.1 INLINE**

- When a word is **INLINE specified**, its definition is copied and expanded instead of being called.

- Controls performance/size trade-offs.

**6.2 Transient words**

- t0--t9 are **temporary words** and are not registered in the dictionary. They are used for small factorization.

- Upper case T0, etc. are normal words (registered in the dictionary).

------------------------------------------------------------------------

**7. Typical superwords (examples)**

Specific superwords are defined in boot.fth / editor.fth.
 Typical examples (conceptual):

- Basic I/O: . CR SPACE TYPE, etc.

- Control syntax: IF \... THEN BEGIN \... UNTIL FOR \... NEXT

- String/buffer operations: HERE ALLOT , C, etc.

* For an accurate list that matches the implementation, refer to the actual .fth file.

------------------------------------------------------------------------

**8. Embedding**

- **Embedded sample** available in system.c.

- Initialize the VM from within a C/C++ application, run outer, and use system
hooks for OS integration as needed.

------------------------------------------------------------------------

**9. Sample Session**

: SQUARE ( n \-- n ) DUP \* ;

5 SQUARE . \\ → Outputs 25

\\ Comment

\[ 1 2 + \] . \\ Immediate interpretation outputs 3

------------------------------------------------------------------------

**10. Development Notes**

- It is common to design the system to automatically load boot.fth.

- editor.fth is a simple editor implementation (using key input primitives).

- Since this is a minimal core, **file operations and device-dependent terms should be extended as necessary**.

------------------------------------------------------------------------

**Appendix A: Terms**

- **XT**: eXecution Token (execution address)

- **TOS/NOS**: Top/Next Of Stack

- **RTOS**: Return-Stack's TOS

- **CELL/BYTE**: Implementation-dependent size (DWC cells follow 32-bit/64-bit environments)

------------------------------------------------------------------------

**Appendix B: Reference Implementation Files**

- dwc-vm.c / dwc-vm.h / system.c

- boot.fth / editor.fth

- dwc.sln / dwc.vcxproj / makefile

-----------------------------------------------------------------

**Update History**

- 2025-08-11 First edition (draft)

Now available! First, we have prepared a draft version of the “forth-dwc
Reference Manual” on the right side of the canvas. The contents include an overview, build method, state transition (ColorForth
 influence)/ DWORD Code Specifications / Stack Effect Table for All Primitives / Assembly Procedures. 

Reference information (mainly README):

- Purpose and Structure of DWC (3-file implementation, VM
is approximately 250 lines / 32+13 primitives, DWORD Code determination rules). [GitHub](https://github.com/CCurl/forth-dwc)

- Behavior of the four states and immediate words (:, ;, \[, \], (, ), ((, )), explanation of INLINE and
transient
words, list of all primitives (0–44). [GitHub](https://github.com/CCurl/forth-dwc)

**Next steps (please let us know your preferences)**

1. Extract and add a **complete list of superwords** from boot.fth / editor.fth (with usage examples)
2. Provide detailed **startup procedures and execution examples** for the command line (Windows/Linux)
3. Export files for distribution in **PDF / Markdown**

I will continue to expand this. Is it okay to add (1) “Complete list of superwords” first?

Source

**My question:

Please explain the meaning of the terms. Please explain the following term “cell.” extern cell
fOpen(cell name, cell mode); extern void fClose(cell fh); extern cell
fRead(cell buf, cell sz, cell fh); extern cell fWrite(cell buf, cell sz,
cell fh); extern cell fSeek(cell fh, cell offset);

**ChatGPT:**

In this case, **cell** refers to the “Forth cell type” in forth-dwc, expressed in the C
language.

------------------------------------------------------------------------

**1. Meaning of cell in Forth**

- **cell** refers to the “basic integer type in Forth.”

- Its size is implementation-dependent and is often equal to the machine's pointer size.

- 32-bit environment → **4 bytes (32-bit)**

- 64-bit environment → **8 bytes (64-bit)**

In Forth, data (either an address or an integer) that fits into one slot of the stack is called a
cell.

------------------------------------------------------------------------

**2. Definition of cell in forth-dwc**

Looking at dwc-vm.h on GitHub, you can see that it is typedef'd according to the environment.

c

Copy Edit

typedef intptr_t cell; // Actually, a signed integer with the same pointer width

- intptr_t is a C standard “signed integer with the same size as a pointer.”

- This allows **both integer values and pointers to be stored in the stack with the same cell
type.

------------------------------------------------------------------------

**3. Meaning of the functions you listed**

These are wrappers that implement Forth file I/O from the C side.

c

Copy Edit

extern cell fOpen(cell name, cell mode);

- name is a string (address) on the Forth side

- mode is the opening method (e.g., read/write)

- The return value is a file handle (cell type)

c

Copy and edit

extern void fClose(cell fh);

- fh = file handle

c

Copy and edit

extern cell fRead(cell buf, cell sz, cell fh);

- buf = buffer address to read from

- sz = number of bytes to read

- fh = file handle

- Return value is the actual number of bytes read

c

Copy and edit

extern cell fWrite(cell buf, cell sz, cell fh);

- Write version

c

Copy and edit

extern cell fSeek(cell fh, cell offset);

- Change file position

- Return value is the new position (or error code)

------------------------------------------------------------------------

**4. Summary**

- **cell** = Forth's “data unit for one word,” typedef'd to intptr_t in C
.

- This allows integers, pointers, and addresses to be handled uniformly

- The functions you referenced are the bridge between the Forth VM and the OS file API

------------------------------------------------------------------------

If desired, I can create C code that immediately verifies in your environment whether **cell** is actually 32-bit or 64-bit
within forth-dwc.
My question:

**My question:**

There is still a lot I don't know. I would like to add some extensions by modifying system.c, but first I want to properly understand dwc-vm.h. Could you explain the following three lines?
enum { COMPILE=1, DEFINE, INTERPRET, COMMENT }; typedef struct { ucell
xt; byte sz; byte fl; byte ln; char nm\[NAME_LEN+1\]; } DE_T; typedef
struct { char \*name; ucell value; } NVP_T;

**ChatGPT:**

Great. The three lines you mentioned represent **execution state constants**, **a single dictionary entry**, and **a name-value pair**. Let's break them down quickly.

------------------------------------------------------------------------

**1) enum { COMPILE=1, DEFINE, INTERPRET, COMMENT };**

- **Purpose**: Enumerators representing the **state** of the outer interpreter (outer
interpreter).

- **Values**: COMPILE=1, DEFINE=2, INTERPRET=3, COMMENT=4 (C
enumeration automatically increments the value by 1 when the first value is assigned)

- **Meaning in brief**

- COMPILE... State of **compiling** (converting to code) words and numbers

- DEFINE ...... **Start of definition** of a new word (add the word to the dictionary and transition to COMPILE,
 etc.)

- INTERPRET... state where words are **executed immediately**

- COMMENT ... state where tokens are **ignored** in comments

- These are switched by “immediate words” such as :, ;, \[, \], (, ),
 etc., and change the behavior of the outer interpreter. \
(The number 0 is omitted because many designs leave room to use 0 for “undefined/invalid” or similar purposes.

------------------------------------------------------------------------

**2)**

c

Copy Edit

typedef struct {

ucell xt;

byte sz;

byte fl;

byte ln;

char nm\[NAME_LEN+1\];

} DE_T;

DE_T is a **D**ictionary
**E**ntry(dictionary entry) This structure represents one entry. It stores Forth
 “words” with metadata.

- ucell xt

- **Execution Token** (execution token /
code address). This is the **destination** when executing this word.

- ucell is an “unsigned cell”. In implementation, it is often equivalent to uintptr_t
(same as the pointer width).

- byte sz

- The **size** of the word body. In DWC,
 this is used to determine **how many cells/bytes to copy when inlining the word**, as “copy expansion (INLINE)” is available.

- The “unit (bytes/cells)” is implementation-dependent, but in DWC,
 since it is “DWORD-Code,” it is often used as the number of cells (number of words).

- byte fl

- **Flag**. Typical examples include

- **IMMEDIATE** (execute immediately regardless of interpretation state)

- **INLINE** (expand code instead of calling)

- **TRANSIENT** (temporary words, not registered in the dictionary)
bits are set (the exact bit assignment can be confirmed in the #define
group).

- byte ln

- **Actual length of the word name** (number of bytes). Used in combination with nm
below for fast comparison.

- char nm\[NAME_LEN+1\]

- **Null-terminated string of the word name**. The maximum length is NAME_LEN. +1 is for \‘\\0\’.

**Typical usage (concept)**

- When defining a new word, fill in DE_T and add it to the dictionary (linked list or array).

- When searching, compare strings using ln and nm, and **jump** to xt when executing.

- If INLINE is specified, instead of calling xt, **copy sz
from the code area** and expand.

------------------------------------------------------------------------

**3)**

c

Copy Edit

typedef struct {

char \*name;

ucell value;

} NVP_T;

NVP_T is **N**ame--**V**alue **P**air, or
**a table of name-value pairs**.

- **Example of use**

- To make a **group of constants** (file modes, key codes, OS
flag values, etc.) on the C side referable by **symbol names** on the Forth side.

- Example:

c

Copy Edit

static const NVP_T file_modes\[\] = {

{\“r\”, F_READ}, // C-side values as ucell

{\“w\”, F_WRITE},

{\“rb\”, F_READ_BIN},

{\“wb\”, F_WRITE_BIN},

{NULL, 0} // End

};

- The outer interpreter or initialization code scans this array and registers **constant names → numeric values** in the Forth
environment.

--------------------------------

**Reference: Related typedef (heading organization)**

The implementation is roughly as follows (please refer to dwc-vm.h
for the exact definitions).

c

Copy Edit

typedef intptr_t cell; // Signed integer for pointer width

typedef uintptr_t ucell; // Unsigned integer for pointer width

typedef unsigned char byte;

- This allows “pointers, integers, and addresses” to be safely exchanged with the **same size** (4 bytes in a 32-bit
environment, 8 bytes in a 64-bit environment).

------------------------------------------------------------------------

**Example usage (very short example)**

c

Copy Edit

// Add a new word to the dictionary (rough example)

DE_T \*de = alloc_de();

de-\>xt = (ucell)code_ptr; // Execution location

de-\>sz = code_len_in_cells; // Size for inline copying

de-\>fl = FLAG_INLINE; // Example: inline attribute

de-\>ln = (byte)strlen(\“SQUARE\”);

strcpy(de-\>nm, \“SQUARE\”);

dict_add(de);

// Pour constants from the name-value table

for (const NVP_T *p = file_modes; p-\>name; ++p) {

forth_define_constant(p-\>name, p-\>value);

}

------------------------------------------------------------------------

If necessary, we will analyze the **bit allocation** of fl and the
\*\* units (cells/bytes) of sz together with the actual #define
and usage locations (add-word/inline expansion processing), and track them with diagrams. Shall we delve deeper into that next?

Information source

Start of form

End of form
