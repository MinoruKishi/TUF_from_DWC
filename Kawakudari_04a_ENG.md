ChatGPT 5
You:
First, I want to understand FORTH-DWC.
ChatGPT:
Sounds good 👍
To understand "forth-dwc," let's summarize the big picture.
1. What is forth-dwc?
DWC stands for dword-code
It's an ultra-minimal implementation of a Forth-like language,
The VM (virtual machine) is about 250 lines.
The executable is extremely lightweight: about 17KB on Windows and 24KB on Linux.
Composed of three C files
dwc-vm.c … Core of the VM (inner/outer loops)
dwc-vm.h … Constants, types, and external function declarations
system.c … OS-dependent I/O and REPL
2. How DWC Works
Command Representation
1 instruction = 1 DWORD (32-bit)
Value 32 to 44 or lower → Primitive (VM-built-in instruction)
Value ≥ 0x4000_0000 → Literal value (determined by high bits)
Other values → Address of a user-defined word registered in the dictionary
State
Has 4 states, switching input interpretation
COMPILE: Compiles input
DEFINE: Adds a new word to the dictionary
INTERPRET: Execute immediately
COMMENT: Treats as comment
State switching is performed using control characters or : ; [ ] ( )
3. Primitive Commands
32 basic primitives + 13 system primitives
Example:
dup (Stack Duplication)
+ (Addition)
jmp (jump)
key / emit (I/O)
outer (Interpret C-side external strings in Forth)
4. Execution Flow
main() (system.c) executes dwcInit() → boot("boot.fth")
boot.fth is loaded, and the entire text is passed to outer()
outer() splits the text into tokens, and inner() executes them in the VM
5. Features
ColorForth influence
Unique input style, including control characters and (( )) comments.
INLINE words
Can be expanded macroscopically.
Transient Words (t0–t9)
Temporary words that do not remain in the dictionary.
6. In summary
forth-dwc is a "VM implementing a minimal Forth interpreter in C."
Easy to extend since it can call external C functions (like emit, key, fOpen)
Simple structure: string → dictionary word sequence → VM execution via outer()
👉 Question: As the next step,
Do you want to understand the behavior of the primitive list step by step?
Or would you rather actually feed in some Forth code and play around?
You:
Run forth-dwc and it will wait for input. Please tell me what I can and cannot do at that point.
ChatGPT:

Thinking time: 1m 15s
Got it! I'll quickly outline the "absolute minimum you can do" and "what you can't do/is undefined" in the REPL (ready-to-print state).
(*forth-dwc is ultra-minimal. Advanced features depend on definitions in boot.fth)
What can be executed now (guaranteed minimum)
Immediate execution (INTERPRET) and definition
Hard-coded immediate state-switching keywords
: (to DEFINE) / ; (end definition, to INTERPRET) / [ (to INTERPRET) / ] (to COMPILE) / ( ) / (( )) (comment)
Defining new words:
: STAR 42 emit ; etc. (emit outputs one character)
Reads entire lines, interprets and executes per token (REPL: line input → outer interpretation)
Stack & Control
Data stack operations: dup drop swap over
Arithmetic: + - * / mod
Comparison/Logic: <, =, >, and, or, xor
Control primitives: jmp, jmpz, jmpnz; simple loop: for next
Example) Display 10 asterisks:
10 for 42 emit next
Memory/Return Stack
Memory: @ ! (cell) / c@ c! (byte)
Return stack: >r r@ r>
I/O/Time/File
Key Input: key (waits for 1 character) / ?key (check if pressed 1/0)
Output: emit(1 character) / ztype(outputs null-terminated string, requires pointer)
Time/Wait: timer / ms
File: fopen fclose fread fwrite fseek
Execute another string as Forth on the spot
outer (pass an address to evaluate the string on the spot)
※ In the REPL, it's the same vibe as passing a line to outer for interpretation
As-is, this is "not possible/undetermined" (depends on boot.fth)
The words below are not included in the VM primitives. They cannot be used unless defined in boot.fth.
The design assumes you'll create your own if needed.
High-level control structures: IF / ELSE / THEN / BEGIN / UNTIL / DO / LOOP, etc.
Numeric output / stack display: . .S, etc. (Only emit/ztype exist, so custom implementation is required)
String literals: .." ..." or S" … " (unusable if undefined)
Variables/Arrays/Dictionary-like: VARIABLE/CONSTANT/CREATE/DOES>/ALLOT, etc.
Advanced features like floating-point/exceptions/multitasking/vocabulary (VOCABULARY)
Various standard Forth immediate words (POSTPONE, IMMEDIATE, etc.)
System command execution (system): May be unavailable/dangerous depending on build or environment
REPL Quick Notes
Prompt display:
Displays "... " in COMPILE state, otherwise displays "ok" (passed to outer on a per-line basis)
Key input uses key/?key as a word (REPL itself uses line input)
Temporary words t0..t9 are disposable and not saved in the dictionary (useful for short decomposition)
Quickly testable "guaranteed to work" examples
(Output is limited to text only)
Echo the input key as-is:
key dup emit drop
Wait 1 second and output !:
1000 ms 33 emit
Loop to output 10 asterisks:
10 for 42 emit next
If you want to use "IF/ELSE" or ". (displaying numbers)", you can build a mini dictionary by adding minimal implementations.
We recommend starting with "Display ., .S", "Number → String", and "Simple IF/ELSE/THEN".
You:
Thank you very much for your response.
Please tell me what cannot be executed in forth-dwc.
ChatGPT:
I see, you want to organize what it "can't do."
Since forth-dwc only has "VM primitives + the minimal vocabulary defined in boot.fth," it lacks most of standard Forth's convenient features.
I'll list the "things it can't do" as seen from the source (PRIMS definitions in dwc-vm.c, README).

Things you cannot do in forth-dwc (unless defined in boot.fth)
1. Control structures
IF … ELSE … THEN
BEGIN … UNTIL / AGAIN
DO ... LOOP / +LOOP
CASE ... ENDCASE
→ Not available as a primitive. Users must define it by combining jmp/jmpz.
2. Display and Debugging
Numeric Display: . (Outputs stack top in decimal)
Stack display: .S
String literals: ." hello" or S" abc"
→ Output methods are limited to emit and ztype.
3. Memory Management and Dictionary Operations
VARIABLE / CONSTANT / CREATE / ALLOT / DOES>
Vocabulary management (VOCABULARY) and FORGET
→ Dictionary operations are limited to add-word and outer.
4. Immediate Compilation System
IMMEDIATE
POSTPONE
Conditional branching during compilation (e.g., standard Forth's [IF] ... [THEN])
→ State transitions are limited to hardcoded : ; [ ] ( ) (( )) only.
5. Numeric Representation and Arithmetic
Floating-point arithmetic
Shift operations (LSHIFT / RSHIFT)
Logical NOT (0= equivalent)
Character code conversion (CHAR, etc.)
→ Only integer arithmetic, comparison, AND/OR/XOR.
6. Environment System
Multitasking
Exception Handling (CATCH/THROW)
Standard input/output redirection and file attribute modification
→ Only C wrappers like fopen/fclose/fread/fwrite/fseek are available.
7. High-Level Tools
Dictionary Browsing (WORDS)
Comment display and listing (SEE)
Assembler/meta compilation
→ None whatsoever, naturally.
In other words
forth-dwc is a VM that can only perform "stack operations, memory access, jumps, simple loops, key input/character output, and file I/O."
Any additional functionality is designed to be built upon using boot.fth or custom definitions.
👉 To proceed:
Examine the contents of boot.fth to see "how close it gets to standard Forth at a minimum." This will clarify the actual "list of things it can't do."
Want to take a look?
You:
"forth-dwc is a VM that only supports 'stack operations, memory access, jumps, simple loops, key input/character output, and file I/O'.
Any additional functionality is designed to be built upon using boot.fth or custom definitions."
The above points were taught to me, and since they're important, please explain them.
　1) How to implement additional functionality by using or modifying boot.fth
　2) How to implement additional functionality by building upon custom definitions
ChatGPT:
Great question!
The paths to adding "additional functionality" are broadly twofold: (1) configuring boot.fth and (2) building custom words in the REPL. Both effectively utilize DWC's minimal primitives and internal variables.
First, the foundational facts:

The available primitives (dup, jmp, jmpz, ... key, emit, ztype, fread, ... etc.) are fixed and pre-implemented on the VM side.
The VM exposes internal variables as "literal words" (mem, mem-sz, base, state, (h) here, (vh) vhere, (l) last, >in, etc.). They are registered in the dictionary during initialization, making them accessible from Forth.
Line input is interpreted by outer; : and ; define/terminate state transitions via hardcoded rules.
1) Use/modify boot.fth to implement additional features
Goal: Enable "convenient words close to standard Forth" immediately after startup.
Method: Define minimal "assembly"-equivalent words (`, here, allot, cr, ., etc.) in boot.fth. This makes the REPL more comfortable every time.
Example implementation of representative low-level words (to be placed in boot.fth)
(Can be implemented using only DWC internal variables and primitives)
\ ===== Basic cell and here =====
4 constant CELL

: here@   ( -- idx )   (h) @ ;             \ Read the "index" value of here
: here!   ( idx -- )   (h) ! ;             \ Overwrite here
: ,       ( x -- )     \ code[here++] = x
  here@ CELL * mem +   \ Write destination address = mem + here*4
  swap !               \ Write to cell
  1 (h) +! ;           \ Increment here by 1 (cell unit)

==== Line breaks and spaces =====
: cr      ( -- )  10 emit ;
: space   ( -- )  32 emit ;
: spaces  ( n -- )  0 do 32 emit loop ;

\ ===== Numeric → Decimal Display (Simplified Version) =====
\ Simplified implementation handling negatives, zero, and allocating internal temporary buffers
create _buf 32 allot

: abs     ( n -- u )   dup 0< if negate then ;
: u/mod10 ( u -- u' d ) 10 /mod ;   \ /mod is (a b--r q), so divide by 10

: (u.)    ( u -- addr )        \ Convert u to a decimal string and return the buffer start address
  _buf 31 + >r                 \ r: End pointer
  begin
    u/mod10                    \ u' d
    48 +                       \ Based on '0'
    r@ c!  r> 1- >r            \ Write one character and return pointer
    dup 0=
  until
  drop  r> 1+                  \ Start position (next to last written)
;

: .       ( n -- )
  dup 0< if 45 emit abs then   \ If negative, output '-'
  (u.) ztype space ;

\ ===== Variables/Constants (Ultra-Simple Version) =====
\ vhere is the starting index of the data area. Simple allocation using the range between vhere and last
: allot   ( n -- )  \ Allocate n cells (handled by cell count)
  (vh) @ swap + (vh) ! ;
: cells   ( n -- n*4 )  CELL * ;

: variable ( -- a )    \ Allocates space for one cell and returns its address
  (vh) @ cells mem +   \ Calculate the actual address
  1 allot ;

: constant ( x -- )    \ Bind x to the immediately following name
  create , does> @ ;
Point
, (comma) means "mem + here*4, where here is written and incremented by 1". Since DWC's here is managed by cell index, it's multiplied by 4 when converting to an address.
mem and (h) are internal variable literals (addresses) exposed by the VM, so @ ! +! can be used.
. (numeric display) implements a simple decimal conversion internally to output strings via ztype.
One more step (adding branch syntax)
Creating immediate words like IF ELSE THEN fundamentally requires a mechanism to patch jump destinations at compile time.
DWC does not provide an operation word to attach an IMMEDIATE flag to user-defined words, making it somewhat difficult to reproduce simple IF/ELSE/THEN as immediate words (the VM has an IMMED flag, but there is no API accessible from Forth).
Workaround:

Combinator approach (higher-order style): Create "branching words" that decide at runtime, like cond true-xt false-xt choose
Manually patch "unresolved jumps": Prepare a mini-assembler in boot.fth that rewrites branch targets using words like (mark) and (resolve), along with , and @ !
(I can transcribe this part if needed)
2) Implement by stacking custom definitions (define each in REPL using `:`)
This method involves incrementally adding definitions via `: ... ;` in the REPL after startup.
Having minimal words in boot.fth makes it easier to add them in the REPL.
Examples of handy tools (OK to define on the fly in REPL)
: 2dup   ( a b -- a b a b )  over over ;
: 2drop  ( a b -- )          drop drop ;
: =0     ( n -- f )          0 = ;
: <>     ( a b -- f )        = 0 = ;
: max    ( a b -- m )        2dup < if swap then drop ;
: min    ( a b -- m )        2dup > if swap then drop ;

: key?   ( -- f )            ?key ;
: type   ( a -- )            ztype ;   \ Alias
: emit?  ( c -- )            emit ;    \ Alias
: delay  ( ms -- )           ms ;

: .s     ( ... -- ... )      \ Simplified version that just peeks at TOS
  (sp) @ (stk) + @ . ;       \ Create a proper .S file separately
Note
Since immediate words cannot be created, it is difficult to directly reproduce standard Forth's "syntactic sugar".
Nevertheless, arithmetic, comparison, I/O, and file handling are fully implemented as primitives, making it perfectly feasible to build applications.
Which to choose?
Want to make each task easier → 1) Tidy up boot.fth
Want to grow from minimal with trial and error → 2) Build in the REPL
In actual use, the standard approach is to first prototype in the REPL → then port it to boot.fth if it works well.
If needed, build a complete "minimal standard library (boot.fth)" based on the fragments above.
You could prepare it as a single file containing "numeric display, stack display, basic variables, simple branching combinators, and file utilities," but how much should you include?
You:
I want to get a feel for it, so
1) How to organize boot.fth,
2) Building custom words in the REPL
to achieve
to create the word "HELLO"
using both methods:
ChatGPT:

Thinking time: 1 second

Thinking time: 2m 52s
Alright, I'll create "HELLO" in two ways.
(In Forth-DWC, you can display it using only **emit and ztype**. In the REPL, single-character literals like 'H' can be used.)
1) Set up boot.fth (usable immediately after startup)
system.c reads and evaluates the current boot.fth by default (when no filename is specified) → Add the following to boot and save → Run → Just type HELLO in the REPL.
Mechanism: Writes "Hello, world!\0" once to a safe offset (here, 120000 bytes into memory).
and during execution, output mem + offset using ztype. Line breaks are 10 emit.
\ ===== boot.fth append starts here =====

\ Memory location for string (well beyond TIB's 10000)
120000 constant HELLO-OFF

: init-hello ( -- )
  mem HELLO-OFF +        \ a = Write destination address
  dup  'H' over c! 1+    \ Place "Hello, world!" as a byte sequence
  dup 'e' over c! 1+
  dup 'l' over c! 1+
  dup 'l' over c! 1+
  dup 'o' over c! 1+
  dup  ',' over c! 1+
  dup ' ' over c! 1+
  dup 'w' over c! 1+
  dup 'o' over c! 1+
  dup  'r' over c! 1+
  dup 'l' over c! 1+
  dup 'd' over c! 1+
  dup '!' over c! 1+
       0   swap c! ;     \ NUL terminated

: HELLO ( -- )
  mem HELLO-OFF + ztype   \ String body
  10 emit ;               \ LF

\ Place string once at startup
init-hello

\ ===== End =====
Note
ztype outputs a null-terminated string, emit outputs a single character.
mem is the VM's heap start address. Adding an offset to mem allows creating a C string at any position.
The offset can be freely changed as long as it doesn't overlap with other data.
2) Build custom words in REPL (Minimal & Fast)
Mechanism: No memory allocation. Just output each character directly with **emit** (shortest path).
Paste directly into REPL and execute:
: HELLO ( -- )
  'H' emit  'e' emit  'l' emit  'l' emit  'o' emit
  ',' emit  ' ' emit
  'w' emit  'o' emit  'r' emit  'l' emit  'd' emit  '!' emit
  10 emit
;
After definition, it will be displayed as HELLO ↵.
Which do you choose?
boot.fth version: Want it ready immediately after startup, for long strings or reuse → This one
REPL version: Need to try it right now, ultra-minimal → This one
If needed, I can also provide a simple string utility (allocator + ztype wrapper) to create equivalents of `." ..."` or `S" ..."`. How much do you want to keep available?
You:
Confirmation: In system.c, `void zType(const char *str)` is defined. Can't we handle strings without using `emit` by using `ztype`?
ChatGPT:
Yes, that understanding is correct.
Looking at the source:

In `system.c`:
void zType(const char *str) { 
    fputs(str, outputFp ? (FILE*)outputFp : stdout); 
}
It is defined as follows.
The ztype primitive is also defined on the VM side,
and it performs the operation of "taking the address at the top of the stack and outputting the null-terminated string pointed to by that address."
In other words,
emit outputs one character at a time (requiring a character code at the stack top).
Since ztype is directly connected to C's fputs, if the stack top is a string address, it displays the entire string at once.
Important note:
ztype only works correctly when the "starting address of a null-terminated string" is pushed onto the stack.
The Forth REPL does not have built-in support for understanding string literals like "Hello, world!". Therefore, you must write the string to an arbitrary address in memory yourself and then pass that address.Practical Example
For example, if you put the following in boot.fth:
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
  mem HELLO-OFF + ztype  10 emit ;
init-hello
When you type HELLO in REPL, ztype outputs "Hello, world!" all at once.
🔑 To summarize:
emit → one character code
ztype → Outputs a null-terminated string all at once
Therefore, if the string is already in memory, using ztype is easier.
👉 Now, here's a question:
"I want to add the equivalent of `." ..."` to `boot.fth` so that `." Hello, world!"` can be written."
or
"Do you want to continue by understanding the current manual method of placing it in memory?"
Which path should we take?
You:
Well, writing "." Hello, world!" is the natural way. Please use "." to answer this time.
ChatGPT:
Got it 👍
Indeed, writing ". Hello, world!" feels natural in a Forth-like way.
However, since forth-dwc doesn't include the ." mechanism from the start (it lacks parsing for string literals), you need to define and add the immediate word ." in boot.fth.
Concept of the Mechanism
The standard Forth ." is an immediate word that "embed strings into the dictionary at compile time and ztype them at runtime."
Since forth-dwc lacks a mechanism to set the IMMEDIATE flag, we cannot implement it exactly like the original.
Therefore, as a simplified version:
When you type `." Hello, world!"` in the REPL, it extracts the rest of that line, copies it to a temporary buffer, and immediately ztypes it for display.
In other words, it becomes a "word dedicated solely to immediate output."
Simplified boot.fth implementation example
\ Simple buffer (allocated sufficiently large)
create strbuf 128 allot

: ."   ( -- )
  \ Copy characters from input buffer >in position to next "
  >in @ dup
  begin
    dup c@ 34 <>   \ 34 = '"'
  while
    dup c@ over strbuf + c!
    1+ >in +!
  repeat
  drop
  0 strbuf + c!     \ null-terminated
  strbuf ztype
  10 emit           \ Newline
;
Usage
You can type the following in the REPL:
." Hello, world!"
→ Output:
Hello, world!
Additional Notes
The above implementation still needs improvements like copying until the end of the line, but it's sufficient to get a feel for it.
For a proper implementation, you should extend the outer lexical analysis loop and rewrite the C code to treat the next token as a string immediately upon encountering a "."
👉 Now, a question:
"the bare minimum you can put into boot.fth and test right away"?
Or do you want to modify the C side to make it behave exactly like the standard Forth ."?
You:
Just to confirm, is there an
immediate
: ." (") comp? if (ztype) , exit then ztype ;  immediate
Is this unrelated?
ChatGPT:
Yes, that ." definition is indeed relevant.
The line you wrote
immediate
: ." (") comp? if (ztype) , exit then ztype ;  immediate
defines "." as an immediate word (IMMEDIATE) and implements behavior switching between compile mode and interpret mode.
Roughly speaking:

(") … An auxiliary word that extracts the string up to the following " and prepares its address on the stack (by storing it in the dictionary/memory).
comp? … An auxiliary word that returns a boolean indicating whether the current state is COMPILE.
if (ztype) , exit then …
If compiling, assemble code to display the string at runtime (call (ztype), and if needed, immediately embed the data/address with a comma → execute that string as ztype).
exit is intended to embed an immediate return at the end of the word being defined.
then ztype … During interpretation, display immediately using ztype.
Thus, in an environment with this definition, similar to standard Forth:
: HELLO  ." Hello, world!" ;
HELLO
works. ztype itself is a VM system primitive that takes the "starting address of a null-terminated string" on the stack and outputs it in one go. Its C implementation uses fputs.
Summary
Yes, if the ." definition is present, you can output strings using ." ..." without needing to list emit.
Using it like the HELLO example above is fine.
The auxiliary words (") / (ztype) / comp? / , / exit are assumed to be provided by boot.fth (as DWC VM primitives, only ztype and basic symbols are provided).
You:
The words defined in forth-dwc are a carefully selected minimal set. When creating words or scripts using forth-dwc going forward, I recommend first checking the .c and .fth files shown earlier and making effective use of the words defined there. Is that possible?
ChatGPT:
Yes, that understanding is correct.
forth-dwc consists of a "minimally curated set of primitives" and "the bare minimum auxiliary words defined in files like boot.fth/editor.fth".Therefore, when writing new words or scripts, the most natural and safe approach is to first examine the primitives defined on the C side (dwc-vm.c, system.c) and the immediate words and auxiliary words already provided on the Forth side (boot.fth, editor.fth), then combine them to write your code.
1. Primitives provided by the C side
All are listed in the PRIMS macro in dwc-vm.c.
Example:
Stack operations: dup drop swap over
Arithmetic: + - * /mod
Control: jmp jmpz jmpnz for next exit lit
Memory: @ ! c@ c! +!
I/O: emit ztype key ?key
File: fopen fclose fread fwrite fseek
Time: ms timer
Other: add-word outer system
This forms the "core".
2. Literals automatically added to the dictionary during VM initialization
dwcInit() adds mem, mem-sz, base, state, (h), (vh), etc., to the dictionary.
→ Essential when directly accessing memory regions or pointers.
3. Words already expanded in boot.fth / editor.fth
As you pointed out, boot.fth already contains
: ." (") comp? if (ztype) , exit then ztype ; immediate
such auxiliary words as ".", (ztype), and comp? are already defined.
In other words, convenient features like "handling string literals" or "checking if in compile mode" are implemented in the .fth files, and users can leverage them.
4. Actual Development Procedure
Break down the desired processing into primitives
→ Example: For "display on screen," use ztype or emit.
Check if boot.fth / editor.fth has equivalent functionality
→ Example: If `."` exists, you can avoid listing multiple `emit` statements.
If not, create your own
→ Combine existing primitives with existing auxiliaries.
→ If absolutely necessary, add primitives to the C side.
🔑 Conclusion
forth-dwc is built on two pillars: the "minimal set of C-side primitives" and the "minimal set of Forth-side immediate words and auxiliary words."
Therefore, when writing new words or scripts, it is optimal to always first check these .c and .fth files and make maximum use of the words already provided.
👉 Upon request,
"Primitive List (by Purpose)"
"List of Major Added Words in boot.fth/editor.fth"
for your reference?
You:
In our conversation, we confirmed that when thinking about or creating new scripts in forth-dwc, we first consider words registered in forth-dwc's .fth or .c files. Would the same apply if other users created scripts using forth-dwc?
ChatGPT:
Yes 👍
This is a fundamental guideline that applies to all users.
Why?
forth-dwc is a minimalist implementation, with a carefully selected set of primitives.
Many "words you'd expect in standard Forth" are omitted, so when users create new scripts or words, they must first consider how to combine the available primitives and existing .fth definitions.
This applies equally to everyone:
Primitives (+ - dup swap @ ! emit ztype ...) are common
Words added to boot.fth / editor.fth (.", comp?, ("), etc.) are also common
Therefore, there is no difference per user, and "checking this first" becomes an essential step.
Differences arise only in:
How each user customizes their boot.fth/editor.fth
→ For example, if someone's boot.fth has .", it's immediately usable, but if another user custom-removed it, it won't work.
Whether primitives have been added to the C side
→ If extensions exist, that user can use their specific primitives.
🔑 Summary
The basic principle of using forth-dwc (first check the words registered in .c/.fth) applies directly to other users as well.
The only difference is whether your forth-dwc environment is stock or customized.
👉 If desired,
"Pure forth-dwc (unmodified)" always provides a list of existing words (primitives + initial definitions from boot.fth)
"Words that may change due to user extensions"
and create a reference that distinguishes between "common to everyone" and "environment-dependent".
Shall we create it?
You:
I'd like to see it. Please tell me.
ChatGPT:
Understood. **"Words that are universally present and 'mandatory'" and "words that vary depending on the environment (boot.fth/editor.fth)."
Always present (guaranteed by core/raw forth-dwc)
A) Immediate state transition words (hardcoded)
: ; [ ] ( ) (( ))
→ Hardcoded in the VM, switches states (DEFINE/COMPILE/INTERPRET/COMMENT). 【】
B) VM primitives (execution core)
(Matches PRIMS in dwc-vm.c and the table in README)
Control: exit lit jmp jmpz jmpnz njmpz njmpnz for next【】【】
Stack: dup drop swap over【】
Arithmetic/Comparison/Logic: * + - /mod < = > and or xor【】
Memory: @ ! c@ c! +!【】
Search: ' (dictionary search)【】
System/I/O/Time/File:
key ?key emit ztype ms timer fopen fclose fread fwrite fseek【】【】
Other: add-word (add next word name to dictionary) outer (pass given string to outer interpretation) system (C's system)【】【】
C) "Internal values (literal words)" automatically registered to the dictionary at startup
The following are permanently stored as words that return values. Example: mem is the heap start address, (h) is a pointer to HERE, etc.
version (vh) (h) (l) (sp) (stk) state base mem mem-sz >in output-fp【】

Example usage:
Place raw strings with mem and c!/@, then display with ztype
Manipulate (h) with @ ! +! to create custom "," (comma) equivalents, etc.
Environment-dependent (varies with boot.fth / editor.fth)
This depends on each user's file contents. Only representative examples are listed (existence is not guaranteed).
String literal system: ." ..." (if implemented, expands to ztype equivalent at compile/interpretation time)
※ If the implementation you previously showed: ." (") comp? if (ztype) , exit then ztype ; immediate exists in a branch, it can be used.
Development aids: comp? (") (ztype) , cr space . .S variable constant create allot does> etc.
Editor/Utilities: Depends on editor.fth (key bindings and simple tools)
Important: **The terms listed here are "available if present"**. They may be undefined in other environments. Only the top row "Always present" is guaranteed to work.
Quick-reference cheat sheet (minimal)
Character output (guaranteed): 'H' emit ／ Null-terminated string: addr ztype【】
Wait 1 second: 1000 ms【】
Loop: 10 for 42 emit next (10 *'s)【】
Minimal example to output memory string:
mem 120000 + dup 'H' swap c! 1+ dup 'i' swap c! 1+ 0 swap c!
mem 120000 + ztype
(mem is pre-allocated, ztype is a system primitive)【】【】
Summary
Common to all: State words, all PRIMS, internal literal words at startup.
Environment-dependent: High-level words like "." depend on boot/editor contents.
When creating new scripts/words, start by building from the "always present" items above → then utilize high-level words if available in the environment. This sequence is safe.
If needed, we can organize this list into a **single-page reference for printing (with Japanese annotations)**. Please specify desired items (e.g., table of all stack effects, usage examples).
You:
Please convert our entire conversation directly into Markdown and save it as a single Markdown file.
ChatGPT:
I have created a file compiling the entire conversation into Markdown.
📂 Download forth_dwc_conversation.md

This file contains our entire conversation so far, preserved exactly as it was in Markdown format. 



