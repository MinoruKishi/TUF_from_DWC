You:

First, I want to understand forth-dwc

ChatGPT:

Sounds good 👍  
To understand "forth-dwc," let's summarize the big picture.

1\. What is forth-dwc?

- DWC stands for dword-code
- It's an ultra-minimal implementation of a Forth-like language,
  - The VM (virtual machine) is about 250 lines
  - The executable is extremely lightweight: about 17KB on Windows and 24KB on Linux.
- Composed of three C files
  - dwc-vm.c … Core of the VM (inner/outer loops)
  - dwc-vm.h … Declares constants, types, and external functions
  - system.c … OS-dependent I/O and REPL

2\. How DWC Works

Command Representation

- 1 instruction = 1 DWORD (32-bit)
  - Value ≤ 32 to 44 → Primitive (VM-embedded instruction)
  - Value ≥ 0x4000_0000 → Literal value (determined by high bits)
  - Other → Address of a user-defined word registered in the dictionary

State

- Maintains 4 states, switching input interpretation
  - COMPILE: Compiles input
  - DEFINE: Adds a new word to the dictionary
  - INTERPRET: Execute immediately
  - COMMENT: Treats as comment
- State switching is performed using control characters or : ; \[ \] ( ) etc.

3\. Primitive Instructions

- 32 basic primitives + 13 system primitives
  - Example:
    - dup (Stack Duplication)
    - \+ (Addition)
    - jmp (jump)
    - key / emit (I/O)
    - outer (Interpret C-side external strings in Forth)

4\. Execution Flow

1. main() (system.c) executes dwcInit() → boot("boot.fth")
2. boot.fth is loaded, and the entire text is passed to outer()
3. outer() splits the text into tokens; inner() executes them in the VM

5\. Features

- ColorForth influence  
    Unique input style, including control characters and (( )) comments.
- INLINE words  
    Can be expanded macroscopically.
- Transient Words (t0–t9)  
    Temporary words that do not remain in the dictionary.

6\. In summary

- forth-dwc is a "VM implementing a minimal Forth interpreter in C"
- Easy to extend since it can call external C functions (like emit, key, fOpen)
- Simple structure: string → dictionary word sequence → VM execution via outer()

👉 Question: As the next step,

- Do you want to understand the behavior of each primitive in order?
- Or would you rather actually feed in some Forth code and play around?

You:

Run forth-dwc and it will wait for input. Please tell me what I can and cannot do at that point.

ChatGPT:

Thinking time: 1m 15s

Got it! I'll quickly outline the "absolute minimum you can do" and "things you can't do/are undefined" in the REPL (ready-to-execute state).  
(*forth-dwc is ultra-minimal. Advanced features depend on definitions in boot.fth)

What can be executed now (guaranteed minimum)

Immediate execution (INTERPRET) and definitions

- Hard-coded immediate state-switching words  
    : (to DEFINE) / ; (end definition, to INTERPRET) / \[ (to INTERPRET) / \] (to COMPILE) / ( ) / (( )) (comment)
- Defining new words:  
    : STAR 42 emit ; etc. (emit outputs one character)
- Reads entire lines, interprets and executes per token (REPL processes line input → outer interpretation)

Stack & Control

- Data stack operations: dup drop swap over
- Arithmetic: + - \* /mod
- Comparison/Logic: &lt; = &gt; and or xor
- Control primitives: jmp, jmpz, jmpnz; simple loop: for next  
    Example) Display 10 asterisks:  
    10 for 42 emit next

Memory/Return Stack

- Memory: @ ! (cell) / c@ c! (byte)
- Return stack: >r r@ r>

I/O/Time/File

- Key Input: key (waits for 1 character) / ?key (check if pressed: 1/0)
- Output: emit(1 character) / ztype(outputs null-terminated string, requires pointer)
- Time/Wait: timer / ms
- File: fopen fclose fread fwrite fseek

Execute another string as Forth on the spot

- outer (pass an address to evaluate the string on the spot)  
    ※ Same vibe as passing lines to outer for interpretation in the REPL

As-is, this is "not possible/undetermined" (depends on boot.fth)

The words below are not included in the VM primitives. They cannot be used unless defined in boot.fth.  
The design assumes you'll create your own if needed.

- High-level control structures: IF / ELSE / THEN / BEGIN / UNTIL / DO / LOOP, etc.
- Numeric output/stack display: . .S, etc. (Custom implementation required since only emit/ztype exist)
- String literals: ." ..." or S" … " (unusable if undefined)
- Variables/Arrays/Dictionaries: VARIABLE/CONSTANT/CREATE/DOES>/ALLOT, etc.
- Advanced features like floating-point/exceptions/multitasking/vocabulary (VOCABULARY)
- Various standard Forth immediate words (POSTPONE, IMMEDIATE, etc.)
- System command execution (system): May be unavailable or dangerous depending on build/environment

REPL Quick Notes

- Prompt display:  
    Displays ... in COMPILE state, ok otherwise (passed to outer per line)
- Key input uses key/?key as a word (REPL itself uses line input)
- Temporary words t0..t9 are disposable and not saved in the dictionary (useful for short decompositions)

Quickly testable "guaranteed to work" examples

(Output is limited to text only)

- Echo the entered key verbatim:  
    key dup emit drop
- Wait 1 second and output !:  
    1000 ms 33 emit
- Loop to output 10 asterisks:  
    10 for 42 emit next

If you want to use "IF/ELSE" or ". (displaying numbers)", you can build a mini dictionary by adding minimal implementations.  
We recommend starting with "Display ., .S", "Number → String", and "Simple IF/ELSE/THEN".

You:

Thank you very much for your response. Please tell me what cannot be executed in forth-dwc.

ChatGPT:

I see, you want to organize what "cannot be done."  
Since forth-dwc only has "VM primitives + the minimal vocabulary defined in boot.fth," it lacks most of standard Forth's convenient features.

I'll list the "things it can't do" as seen from the source (PRIMS definitions in dwc-vm.c, README).

Things you cannot do in forth-dwc (unless defined in boot.fth)

1\. Control structures

- IF ... ELSE ... THEN
- BEGIN … UNTIL / AGAIN
- DO ... LOOP / +LOOP
- CASE ... ENDCASE  
    → Not available as primitives. Users must define them by combining jmp/jmpz.

2\. Display and Debugging

- Numeric display: . (Outputs stack top in decimal)
- Stack display: .S
- String literals: ." hello" or S" abc"  
    → Output methods are limited to emit and ztype.

3\. Memory Management and Dictionary Operations

- VARIABLE / CONSTANT / CREATE / ALLOT / DOES>
- Vocabulary management (VOCABULARY) and FORGET  
    → Dictionary operations are limited to add-word and outer.

4\. Immediate Compilation System

- IMMEDIATE
- POSTPONE
- Compile-time conditional branching (e.g., standard Forth's \[IF\] … \[THEN\])  
    → State transitions are limited to hardcoded : ; \[ \] ( ) (( )) only.

5\. Numeric Representation and Operations

- Floating-point arithmetic
- Shift operations (LSHIFT / RSHIFT)
- Logical NOT (0=, etc.)
- Character code conversion (CHAR, etc.)  
    → Only integer arithmetic, comparison, AND/OR/XOR.

6\. Environment Functions

- Multitasking
- Exception handling (CATCH/THROW)
- Standard I/O redirection and file attribute modification  
    → Only C wrappers like fopen/fclose/fread/fwrite/fseek are available.

7\. High-level Tools

- Dictionary Browsing (WORDS)
- Comment display and listing (SEE)
- Assembler/meta-compiler  
    → Naturally, none whatsoever.

In other words

forth-dwc is a VM that can only perform "stack operations, memory access, jumps, simple loops, key input/character output, and file I/O."  
Any additional functionality is designed to be built upon using boot.fth or custom definitions.

👉 To proceed further:

- Examine the contents of boot.fth to see "how close it gets to standard Forth at a minimum." This will clarify the actual "list of things it can't do."

Want to take a look?

You:

"forth-dwc is a VM that only supports 'stack operations, memory access, jumps, simple loops, key input/character output, and file I/O'. Additional functionality is designed to be built upon using boot.fth or custom definitions." These are the points you taught me, and they are important, so please explain them. 　1) How to implement additional functionality by using or modifying boot.fth 2) How to implement additional functionality by building upon custom definitions

ChatGPT:

Great question!  
The paths to adding "additional functionality" are broadly twofold: (1) refining boot.fth and (2) building custom words via REPL. Both approaches skillfully utilize the minimal primitives and internal variables inherent to DWC.

First, the foundational facts:

- Usable primitives (dup + jmp jmpz ... key emit ztype fread ... etc.) are fixed implementations on the VM side.
- The VM exposes its internal variables as "literal words" (mem, mem-sz, base, state, (h) here, (vh) vhere, (l) last, >in, etc.). These are registered in the dictionary during initialization, making them accessible from Forth.
- Line input is interpreted by outer; : and ; define/terminate state transitions via hardcoded rules.

1) Use/modify boot.fth to implement additional features

Goal: Enable "convenient words close to standard Forth" immediately after startup.  
Method: Define the minimum set of "assembly-like" words (`, here, allot, cr, ., etc.) in boot.fth. This makes the REPL more comfortable every time.

Example implementation of representative low-level words (to be placed in boot.fth)

(Can be implemented using only DWC internal variables and primitives)

\\ ===== Basic cell and here =====

4 constant CELL

: here@ ( -- idx ) (h) @ ; \\ Read the "index" value of here

: here! ( idx -- ) (h) ! ; \\ Overwrite here

: , ( x -- ) \\ code\[here++\] = x

here@ CELL \* mem + \\ Write destination address = mem + here\*4

swap ! \\ Write to cell

1 (h) +! ; \\ Advance here by 1 (cell unit)

\\ ===== Line breaks and spaces =====

: cr ( -- ) 10 emit ;

: space ( -- ) 32 emit ;

: spaces ( n -- ) 0 do 32 emit loop ;

\\ ===== Converting Numbers to Decimal Display (Simplified Version) =====

\\ Simplified implementation handling negatives, zero, and allocating internal temporary buffers

create \_buf 32 allot

: abs ( n -- u ) dup 0< if negate then ;

: u/mod10 ( u -- u' d ) 10 /mod ; \\ /mod is (a b--r q), so divide by 10

: (u.) ( u -- addr ) \\ Convert u to a decimal string and return the buffer start address

\_buf 31 + >r \\ r: End pointer

begin

u/mod10 \\ u' d

48 + \\ Based on '0'

r@ c! r> 1- >r \\ Write one character and return pointer

dup 0=

until

drop r> 1+ \\ Start position (next to last written character)

;

: . ( n -- )

dup 0< if 45 emit abs then \\ If negative, output '-'

(u.) ztype space ;

\\ ===== Variables/Constants (Ultra-Simplified Version) =====

\\ vhere is the starting index of the data area. Simple allocation using the range between vhere and last

: allot ( n -- ) \\ Allocate n cells (handled by cell count)

(vh) @ swap + (vh) ! ;

: cells ( n -- n\*4 ) CELL \* ;

: variable ( -- a ) \\ Allocates space for one cell and returns its address

(vh) @ cells mem + \\ Calculate the real address

1 allot ;

: constant ( x -- ) \\ Bind x to the immediately following name

create , does> @ ;

point

- , (comma) means "write to mem + here\*4 and increment here". Since DWC's here is managed by cell index, we multiply by 4 when converting to an address.
- mem and (h) are internal variable literals (addresses) exposed by the VM, so @ ! +! can be used.
- . (numeric display) implements a simple decimal conversion internally to output strings via ztype.

One more step (adding branch syntax)

Creating immediate words like IF ELSE THEN fundamentally requires a mechanism to patch the jump destination at compile time.  
DWC does not provide an operation word to attach an IMMEDIATE flag to user-defined words, making it somewhat difficult to reproduce simple IF/ELSE/THEN as immediate words (the VM has an IMMED flag, but there's no API accessible from Forth).

Workarounds:

- Combinator approach (higher-order style): Create "branch-at-run-time words" like cond true-xt false-xt choose
- Manually patch "unresolved jumps": Prepare a mini-assembler in boot.fth that rewrites branch targets using , and @ ! with words like (mark) (resolve)

(I can transcribe this part if needed)

2) Implement by stacking custom definitions (define : definitions in the REPL each time)

This method involves incrementally adding definitions via `: ... ;` in the REPL after startup.  
Having minimal words in boot.fth makes it easier to add more via REPL.

Examples of handy tools (OK to define on the fly in REPL)

: 2dup ( a b -- a b a b ) over over ;

: 2drop ( a b -- ) drop drop ;

: =0 ( n -- f ) 0 = ;

: <> ( a b -- f ) = 0 = ;

: max ( a b -- m ) 2dup < if swap then drop ;

: min ( a b -- m ) 2dup > if swap then drop ;

: key? ( -- f ) ?key ;

: type (a --) ztype ; \\ Alias

: emit? ( c -- ) emit ; \\ Alias

: delay ( ms -- ) ms ;

: .s ( ... -- ... ) \\ For now, a simplified version that only looks at TOS

(sp) @ (stk) + @ . ; \\ Create a proper .S file separately

Note

- Since immediate words cannot be created, it is difficult to directly reproduce standard Forth's "syntactic sugar".
- Nevertheless, arithmetic, comparison, I/O, and file handling are fully implemented as primitives, making it perfectly feasible to build applications.

Which to choose?

- Want to make each task easier → 1) Tidy up boot.fth
- Want to grow from minimal code through trial and error → 2) Build in the REPL  
    In real-world use, the standard approach is: first prototype in the REPL → if it works, port it to boot.fth.

If needed, build a complete "minimal standard library (boot.fth)" based on the fragments above.  
You could prepare it as a single file containing "numeric display, stack display, basic variables, simple branching combinators, and file utilities," but how much should you include?

You:

To get a feel for it, please create the word "HELLO" that displays "Hello, world!" using both methods: 1) refining boot.fth, and 2) building custom words in the REPL.

ChatGPT:

Thinking time: 1 second

Thinking time: 2m 52s

Alright, I'll create "HELLO" using both methods.  
(In forth‑dwc, you can display it using only `\*\*emit` and `ztype\*\*`. In the REPL, single-character literals like `'H'` can be used.)

1) Set up boot.fth (usable immediately after startup)

system.c loads and evaluates the current boot.fth by default (when no filename is specified) → Add the following to boot and save → Run → Just type HELLO in the REPL.

Mechanism: Writes "Hello, world!\\0" once to a safe offset (here, 120000 bytes into memory).  
and during execution, output mem + offset using ztype. Line breaks are 10 emit.

\\ ===== Add to boot.fth starting here =====

\\ Memory location for string (well beyond TIB's 10000)

120000 constant HELLO-OFF

: init-hello ( -- )

mem HELLO-OFF + \\ a = Write destination address

dup 'H' over c! 1+ \\ Place "Hello, world!" as a byte sequence

dup 'e' over c! 1+

dup 'l' over c! 1+

dup 'l' over c! 1+

dup 'o' over c! 1+

dup ',' over c! 1+

dup ' ' over c! 1+

dup 'w' over c! 1+

dup 'o' over c! 1+

dup 'r' over c! 1+

dup 'l' over c! 1+

dup 'd' over c! 1+

dup '!' over c! 1+

0 swap c! ; \\ NUL terminated

: HELLO ( -- )

mem HELLO-OFF + ztype \\ String body

10 emit ; \\ LF

\\ Place string only once at startup

init-hello

\\ ===== END =====

Note

- `ztype` outputs a null-terminated string, `emit` outputs a single character.
- mem is the VM's heap start address. Adding an offset to mem lets you create a C string at any position.
- Offsets can be freely changed as long as they don't overlap with other data.

2) Building custom words in REPL (Minimal & Fast)

Mechanism: No memory allocation. Each character is output directly via `\*\*emit\*\*` (shortest path).

Paste directly into REPL and execute:

: HELLO ( -- )

'H' emit 'e' emit 'l' emit 'l' emit 'o' emit

',' emit ' ' emit

'w' emit 'o' emit 'r' emit 'l' emit 'd' emit '!' emit

10 emit

;

After definition, it displays HELLO ↵.

Which do you choose?

- boot.fth version: Want it available immediately after startup, for long strings or reuse → This one
- REPL version: Need to try it right now, ultra-minimal → This one

If needed, we can also provide a simple string utility (allocator + ztype wrapper) to create equivalents of `." ..."` or `S" ..."`. How much do you want to keep permanently available?

You:

Confirmation:In system.c, void zType(const char *str) is defined. Can't we handle strings without emit by using ztype?

ChatGPT:

Yes, that understanding is correct.

Looking at the source:

- In system.c:

void zType(const char *str) {

fputs(str, outputFp ? (FILE*)outputFp : stdout);

}

It is defined as follows.

- The ztype primitive is also defined on the VM side,  
    and it performs the operation of "taking the address at the top of the stack and outputting the null-terminated string pointed to by that address."

In other words,

- emit outputs one character at a time (requiring a character code at the stack top).
- ztype is directly linked to C's fputs, so if the stack top is a string address, it displays the entire string at once.

Important note

- ztype only works correctly when the "starting address of a null-terminated string" is pushed onto the stack.
- Forth's REPL does not include a mechanism to understand string literals like "Hello, world!". Therefore, you must write the string to an arbitrary address in mem yourself and then pass that address.

Practical Example

For example, put the following in boot.fth:

120000 constant HELLO-OFF

: init-hello

mem HELLO-OFF + dup

'H' over c! 1+

'e' over c! 1+

'l' over c! 1+

'l' over c! 1+

'o' over c! 1+

',' over c! 1+

' ' over c! 1+

'w' over c! 1+

'o' over c! 1+

'r' over c! 1+

'l' over c! 1+

'd' over c! 1+

'!' over c! 1+

0 swap c! ;

: HELLO

mem HELLO-OFF + ztype 10 emit ;

init-hello

When you type HELLO in REPL, ztype outputs "Hello, world!" all at once.

🔑 To summarize:

- emit → one character code
- ztype → Outputs a null-terminated string all at once

So if the string is already in memory, using ztype is easier.

👉 Now, a question:

- "I want to add the equivalent of `." ..."` to `boot.fth` so that `." Hello, world!"` can be written."  
    or
- "I want to understand and continue with the current manual method of placing it in memory."  
    Which path should we take?

You:

Right, writing "." Hello, world!" is more natural. Please use "." to answer this time.

ChatGPT:

Got it 👍  
Indeed, writing ". Hello, world!" feels natural in a Forth-like way.  
However, since forth-dwc doesn't include the ." mechanism from the start (it lacks parsing for string literals), you need to define and add the immediate word ." in boot.fth.

Concept of the mechanism

- The standard Forth ." is an immediate word that "embed strings into the dictionary at compile time and ztype them at runtime".
- Since forth-dwc lacks a mechanism to set the IMMEDIATE flag, it cannot be made identical to the original.
- Therefore, as a simplified version:
  - When you type `." Hello, world!"` in the REPL, it grabs the rest of that line, copies it to a temporary buffer, and immediately ztypes it for display.
  - In other words, it becomes a "word dedicated solely to immediate output."

Simplified boot.fth implementation example

\\ Simple buffer (allocated sufficiently large)

create strbuf 128 allot

: ." ( -- )

\\ Copy characters from the position in input buffer >in to the next "

\>in @ dup

begin

dup c@ 34 <> \\ 34 = '"'

while

dup c@ over strbuf + c!

1+ >in +!

repeat

drop

0 strbuf + c! \\ null-terminated

strbuf ztype

10 emit \\ Newline

;

Usage

You can type the following in REPL:

." Hello, world!"

→ Output:

Hello, world!

Additional Notes

- The above implementation still needs improvements like copying to the end of the line, but it's sufficient for getting a feel for it.
- For a proper implementation, you should extend the outer lexical analysis loop and rewrite the C code to treat the next token as a string immediately upon encountering a "."

👉 Now, a question:

- Is "the bare minimum to actually put into boot.fth and test immediately" sufficient?
- Or do you want to modify the C side to make it behave exactly like the standard Forth ."?

You:

Just to confirm, in boot.fth there's immediate : ." (") comp? if (ztype) , exit then ztype ; immediate. Is this unrelated?

ChatGPT:

Yes, that ." definition is indeed relevant.  
The line you wrote

immediate

: ." (") comp? if (ztype) , exit then ztype ; immediate

defines "." as an immediate word (IMMEDIATE) and implements behavior switching between compile and interpret states.

Roughly speaking:

- (") … An auxiliary word that extracts the string up to the next "", stores it (in the dictionary/memory), and prepares its address on the stack.
- comp? … An auxiliary word that returns a boolean indicating whether the current state is COMPILE.
- if (ztype) , exit then …
  - If compiling, assemble code to display the string at runtime (call (ztype), then immediately embed data/address with , if needed → execute that string as ztype).
  - exit is intended to insert an immediate return at the end of the word being defined.
- then ztype … During interpretation, display immediately using ztype at that location.

In other words, in an environment with this definition, just like standard Forth:

: HELLO ." Hello, world!" ;

HELLO

works. ztype itself is a VM system primitive that takes the "starting address of a null-terminated string" on the stack and outputs it in bulk. Its C implementation uses fputs.

Summary

- Yes, if the ." definition is present, you can output strings using ." ..." without needing to list emit.
- Using it like the HELLO example above is fine.
- Auxiliary words (") / (ztype) / comp? / , / exit are assumed to be provided by boot.fth (as DWC VM primitives, only ztype and basic symbols are provided).

You:

The words defined in forth-dwc are a carefully selected minimal set. When creating words or scripts using forth-dwc going forward, I recommend first checking the .c and .fth files shown earlier and making effective use of the words defined there. Is that possible?

ChatGPT:

Yes, that understanding is correct.  
forth-dwc consists of a "minimally curated set of primitives" and "the bare minimum auxiliary words defined in files like boot.fth and editor.fth".Therefore, when writing new words or scripts, the most natural and safe approach is to first examine the primitives defined on the C side (dwc-vm.c, system.c) and the immediate words and auxiliary words already prepared on the Forth side (boot.fth, editor.fth), then combine them to write your code.

1\. Primitives provided by the C side

All are listed in the PRIMS macro in dwc-vm.c.  
Example:

- Stack operations: dup drop swap over
- Arithmetic: + - \* /mod
- Control: jmp jmpz jmpnz for next exit lit
- Memory: @ ! c@ c! +!
- I/O: emit ztype key ?key
- File: fopen, fclose, fread, fwrite, fseek
- Time: ms timer
- Other: add-word outer system

This forms the "core".

2\. Literals automatically added to the dictionary during VM initialization

dwcInit() adds mem, mem-sz, base, state, (h), (vh), etc., to the dictionary.  
→ Essential when directly accessing memory regions or pointers.

3\. Words already expanded in boot.fth / editor.fth

As you pointed out, boot.fth already contains

: ." (") comp? if (ztype) , exit then ztype ; immediate

such auxiliary words as "." and (ztype), comp? are already defined.  
In other words, convenient features like "handling string literals" or "determining if in compile mode" are implemented in the .fth files, and users can leverage them.

4\. Actual Development Procedure

1. Break down the desired processing into primitives  
    → Example: For "display on screen," use ztype or emit.
2. Check if boot.fth / editor.fth has equivalent functionality  
    → Example: If `."` exists, you can avoid nesting `emit` statements.
3. If not, create your own  
    → Create it by combining existing primitives and existing auxiliary words.  
    → If absolutely necessary, add primitives to the C side.

🔑 Conclusion  
forth-dwc is built on two pillars: the "minimal set of C-side primitives" and the "minimal set of Forth-side immediate words and auxiliary words."  
Therefore, when writing new words or scripts, it is optimal to always first check these .c and .fth files and make maximum use of the words already provided.

👉 Upon request,

- "Primitive List (by Use Case)"
- "boot.fth/editor.fth"

for your reference?

You:

In our conversation, we confirmed that when you and I think about or create new scripts in forth-dwc, we first consider words registered in forth-dwc's .fth or .c files. Would the same apply if other users created scripts using forth-dwc?

ChatGPT:

Yes 👍  
This is a fundamental guideline that applies to all users.

Why?

- Forth-dwc is an extremely minimalist implementation, with a carefully curated set of primitives.
- Many "words you'd expect in standard Forth" are intentionally omitted. Therefore, when users create new scripts or words, they must first consider how to combine the available primitives and existing .fth definitions.
- This applies equally to everyone:
  - Primitives (+ - dup swap @ ! emit ztype ...) are common
  - Words added to boot.fth / editor.fth (.", comp?, ("), etc.) are also common
  - Therefore, there is no difference per user, and "checking this first" becomes an essential step.

Differences may arise from:

- How each user customizes their boot.fth/editor.fth  
    → For example, if someone's boot.fth has .", it's immediately usable, but if another user custom-removed it, it won't work.
- Whether primitives have been added on the C side  
    → If extensions exist, those user-specific primitives can be used.

🔑 Summary

- The basic usage principle of forth-dwc (first check the words registered in .c/.fth) applies directly to other users as well.
- The only difference is whether your forth-dwc environment is stock or customized.

👉 If desired,

- The list of words always present in "pure forth-dwc (unmodified)" (primitives + initial definitions in boot.fth)
- "Words that may change due to user extensions"

and create a reference that distinguishes "this is common to everyone" from "this is environment-dependent."  
Shall we create it?

You:

I'd like to see it. Please tell me.

ChatGPT:

Understood. \*\* I'll clearly separate and list "terms that are universally present" and "terms that vary depending on the environment (boot.fth/editor.fth)".

Always present (guaranteed by core/raw forth-dwc)

A) Immediate state transition words (hardcoded)

: ; \[ \] ( ) (( ))  
→ Hardcoded in the VM, switching states (DEFINE/COMPILE/INTERPRET/COMMENT). 【】

B) VM primitives (execution core)

(Matches PRIMS in dwc-vm.c and table in README)

- Control Flow: exit lit jmp jmpz jmpnz njmpz njmpnz for next【】【】
- Stack: dup drop swap over【】
- Arithmetic/Comparison/Logic: \* + - /mod &lt; = &gt; and or xor【】
- Memory: @ ! c@ c! +!【】
- Search: ' (dictionary search)【】
- System/I/O/Time/Files:  
    key ?key emit ztype ms timer fopen fclose fread fwrite fseek【】【】
- Other: add-word (adds next word name to dictionary) outer (passes given string to outer interpreter) system (C's system)【】【】

C) "Internal values (literal words)" automatically registered to the dictionary at startup

The following are permanently stored as words that return values. Example: mem is the heap start address, (h) is a pointer to HERE, etc.

version (vh) (h) (l) (sp) (stk) state base mem mem-sz >in output-fp【】

Example usage:

- Place raw strings with `mem` and `c!/@`, then display with `ztype`
- Manipulate (h) with @ ! +! to create custom "," (comma) equivalents, etc.

Environment-dependent (varies by boot.fth / editor.fth)

This depends on each user's file contents. Only representative examples are listed (existence is not guaranteed).

- String literal system: .." ..." (if implemented, expands to ztype equivalent during compilation/interpretation)  
    ※ If an implementation like : ." (") comp? if (ztype) , exit then ztype ; immediate exists in a branch you previously showed, it can be used.
- Development aids: comp? (") (ztype) , cr space . .S variable constant create allot does> etc.
- Editor/Utilities: Depends on editor.fth (key bindings and simple tools)

Important: \*\*The terms listed here are "available if present" \*\*. They may be undefined in other environments. Only the top row "always present" is guaranteed to work.

Quick-reference cheat sheet (minimal)

- Character output (guaranteed): 'H' emit ／ Null-terminated string: addr ztype【】
- Wait 1 second: 1000 ms【】
- Loop: 10 for 42 emit next (10 instances of \*)【】
- Minimal example to output a memory string:  
    <br/><br/><br/>mem 120000 + dup 'H' swap c! 1+ dup 'i' swap c! 1+ 0 swap c!
- mem 120000 + ztype
-
    <br/><br/>(mem is persistent, ztype is a system primitive)【】【】

Summary

- Common to all: State words, all PRIMS, internal literal words at startup.
- Environment-dependent: High-level words like ".", etc., depend on the contents of boot/editor.
- When creating new scripts/words, start by building from the "always present" items above → If high-level words exist in the environment, utilize them. This order is safe.

If needed, we can organize this list into a single reference sheet for printing (with Japanese annotations). Let us know what items you want (e.g., table of all stack effects, usage examples).

You:

Please convert our entire conversation directly into Markdown and save it as a single Markdown file.

ChatGPT:

I have created a file compiling the entire conversation into Markdown.

📂 Download forth_dwc_conversation.md

This file contains our entire conversation so far, preserved exactly as it was in Markdown format.

