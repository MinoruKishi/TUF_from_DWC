Forth-dwc Reference Manual (Draft)

**forth-dwc Reference Manual (Draft)**

Target: CCurl/forth-dwc (Last checked: 2025-08-11) License: MIT (Implementation is
C/Forth)

------------------------------------------------------------------------

**1. Overview**

- **DWC (DWORD-Code) method** minimal Forth. Standalone execution and **embedded** implementation in C/C++
are possible.

- Implementation consists mainly of **3 files**: dwc-vm.c, dwc-vm.h, and system.c.

- **32 basic primitives** and **13 system primitives**.

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

**3. Language mode (ColorForth influence)**

DWC has 4
states (**State**), which are switched by **control characters in blank spaces**.

| **State** | **Meaning** | **Default action**|
| --- | --- | --- |
| COMPILE | Compile | Compile word/number

DEFINE Start definition Add word to dictionary and go to COMPILE

INTERPRET Interpret Execute word immediately

COMMENT Comment Ignore everything except )/))
-----------------------------------------------------------------------

**Immediate state transition words**

:, ;, \[, \], (, ), ((, ))

- : → DEFINE

- ; → EXIT Compile and INTERPRET

- \[ → INTERPRET

- \] → COMPILE

- (, (( → COMMENT

- ), )) → End comment (each to COMPILE/INTERPRET)

------------------------------------------------------------------------

**4. Code representation (DWORD-Code)**

- **0..44**: Primitive number

- **Literal**: Value with LIT_BITS set in the upper bits

- **Dictionary word**: XT (code address)

------------------------------------------------------------------------

**5. Primitive list**

**Notation**: Stack effect is (in \-- out), TOS=top of stack, NOS=next.

**5.1 DWC Primitives (0--31)**

0 exit ( \-- ) Restore RTOS to PC. Stop when PC=0.

1 lit ( \-- n ) Push next word, PC++.

2 jmp ( \-- ) Sets PC to code\[PC].

3 jmpz ( n \-- ) If TOS==0, sets PC to code\[PC]; otherwise, increments PC. Discards TOS.

4 jmpnz ( n \-- ) If TOS!=0, sets PC to code\[PC]; otherwise, increments PC. Discards TOS.

5 njmpz ( n \-- n ) (conditional branch, non-destructive) If TOS==0, set PC=code\[PC\], else
PC++.

6 njmpnz ( n \-- n ) (conditional branch, non-destructive) If TOS!=0, set PC=code\[PC\], else
PC++.

7 dup ( n \-- n n ) Duplicate.

8 drop ( n \-- ) Discard.

9 swap ( a b \-- b a ) Swap.

10 over ( a b \-- a b a ) Duplicate NOS.

11 ! ( n a \-- ) Store cell (n in a).

12 @ ( a \-- n ) Read cell.

13 c! ( b a \-- ) Store byte.

14 c@ ( a \-- b ) Read byte.

15 \>r ( n \-- ) Push to return stack.

16 r@ ( \-- n ) Copy RTOS and push to data stack.

17 r\> ( \-- n ) Pop RTOS and push to data stack.

18 \* ( a b \-- c ) Multiply.

19 + ( a b \-- c ) Add.

20 - ( a b \-- c ) Subtract.

21 /mod ( a b \-- r q ) Divide and return the remainder.

22 \< ( a b \-- f ) If a < b, set 1.

23 = ( a b \-- f ) If a = b, set 1.

24 \> ( a b \-- f ) If a > b, set 1.

25 +! ( n a \-- ) Add to the cell of a.

26 \' ( \-- a ) Get the address (dictionary) of the next word.

27 for ( n \-- ) Start FOR loop.

28 next ( \-- ) End FOR loop.

29 and ( a b \-- c ) AND.

30 or (a b \-- c) OR.

31 xor (a b \-- c) XOR.

**5.2 System Primitives (32--44)**

32 key ( \-- n ) Next key input (block).

33 ?key ( \-- n ) Key presence (1/0).

34 emit ( n \-- ) Character output.

35 ztype ( a \-- ) Output NUL-terminated string.

36 fopen ( nm md \-- h ) Open file in mode md (h=0 on failure).

37 fclose ( h \-- ) Close file.

38 fread ( a sz h \-- n ) Read.

39 fwrite ( a sz h \-- n ) Write.

40 ms ( n \-- ) Sleep for milliseconds.

41 timer ( \-- n ) Current time (implementation-dependent).

42 add-word ( \-- ) Add next word to dictionary.

43 outer ( a \-- ) Execute the outer interpreter on a.

44 system ( a \-- ) Execute \`system(a)\`.

------------------------------------------------------------------------

**6. Dictionary and language features**

**6.1 INLINE**

- When a word is **specified as INLINE**, its definition is copied and expanded instead of being called.

- Controls performance/size trade-offs.

**6.2 Transient words**

- t0--t9 are **temporary words** that are not registered in the dictionary. They are used for small factorization.

- Upper-case T0, etc., are normal words (registered in the dictionary).

------------------------------------------------------------------------

**7. Typical superwords (examples)**

Specific superwords are defined in boot.fth / editor.fth.
 Representative examples (conceptual):

- Basic I/O: . CR SPACE TYPE, etc.

- Control syntax: IF \... THEN BEGIN \... UNTIL FOR \... NEXT

- String/buffer operations: HERE ALLOT , C, etc.

* For an accurate list that matches the implementation, refer to the actual .fth file.

------------------------------------------------------------------------

**8. Embedding**

- **Embedded sample** available in system.c.

- Initialize the VM from within the C/C++ application, run outer, and use the system
hook to communicate with the OS as needed.

------------------------------------------------------------------------

**9. Sample Session**

: SQUARE ( n \-- n ) DUP \* ;

5 SQUARE . \\ → Outputs 25

\\ Comment

\[ 1 2 + \] . \\ Immediate interpretation outputs 3

------------------------------------------------------------------------

**10. Development Notes**

- It is common to design for automatic loading of boot.fth.

- editor.fth is a simple editor implementation (using key input primitives).

- Since this is a minimal core, **file operation/device-dependent terms should be extended as necessary**.

------------------------------------------------------------------------

**Appendix A: Terminology**

- **XT**: eXecution Token (execution address)

- **TOS/NOS**: Top/Next Of Stack

- **RTOS**: Return-Stack TOS

- **CELL/BYTE**: Implementation-dependent size (DWC cells follow 32-bit/64-bit environments)

------------------------------------------------------------------------

**Appendix B: Reference Implementation Files**

- dwc-vm.c / dwc-vm.h / system.c

- boot.fth / editor.fth

- dwc.sln / dwc.vcxproj / makefile

------------------------------------------------------------------------

**Update History**

- 2025-08-11 Initial version (draft)

