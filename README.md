Download Link: https://assignmentchef.com/product/solved-cs39003-assignment6
<br>
The Lexical Grammar (Assignment 3) and the Phase Structure Grammar (Assignment 4) for the language tinyC have already been defined as subsets of the C language specification from the International Standard <strong>ISO/IEC 9899:1999 (E)</strong>. Finally, three address code (TAC) structure and a further subset of tinyC has been specified (Assignment 5) for translating the input tinyC program to TAC quad array, a supporting symbol table, and other auxiliary data structures.

In this assignment you will be required to write a target code translator from the TAC quad array (with the supporting symbol table, and other auxiliary data structures) to the assembly language of x86-32. The translation is now machine-specific and your generated assembly code would be translated with the gcc assembler to produce the final executable codes for the tinyC program.

<h1>1           Scope of Target Translation</h1>

<ul>

 <li>For simplicity restrict tinyC further:

  <ol>

   <li>Skip shift and bit</li>

   <li>Support only <strong>void</strong>, <strong>int</strong>, and <strong>char </strong> Skip <strong>double </strong>type.</li>

   <li>Support only one–dimensional arrays.</li>

   <li>Support only <strong>void</strong>, <strong>int</strong>, <strong>char</strong>, <strong>void*</strong>, <strong>int*</strong>, and <strong>char* </strong>for returns types of functions.</li>

   <li>No type conversion to be supported.</li>

  </ol></li>

 <li>For input/output, provide a library (similar to that created in Assignment 2) using in-line assembly language program of x86-32 along with syscall for gcc The library will contain the following functions:

  <ol>

   <li>int printStr(char *) – prints a string of characters. The parameter is a character array terminated by ‘ ’. The return value is the number of characters printed.</li>

   <li>int printInt(int n) – prints the integer value of n (no newline). It returns the number of characters printed.</li>

   <li>int readInt(int *eP) – reads an integer (signed) and returns it. The parameter eP is for error reporting (<em>ERR </em>= 1 for error condition, and <em>OK </em>= 0 for no error).</li>

  </ol></li>

</ul>

The header file myl.h of the library will be as follows:

#ifndef _MYL_H

#define _MYL_H

#define ERR 1 #define OK 0

int printStr(char *); int printInt(int); int readInt(int *eP); // *eP is for error, if the input is non-integer

#endif

<h1>2           Design of the Translator</h1>

The steps for target code generation have been outlined in Target Code Generation lecture presentations. In this assignment, however, you do not need to deal with any machine-independent or machine-specific optimization. Hence the translation will comprise of the following major steps only:

<ol>

 <li><strong>Memory Binding</strong>: This deals with the design of the allocation schema of variables (including parameters and constants) that associates each variable to the respective address expression or register. This needs to handle the following:

  <ul>

   <li><em>Handle local variables, parameters, and return value for a function</em>. These are automatic variables and will reside in the Activation Record (AR) of the function. Various design schema for AR are possible based on the calling sequence protocol. A sample Activation Record structure along with the management protocol is shown below:</li>

  </ul></li>

</ol>

<table width="368">

 <tbody>

  <tr>

   <td width="55"><strong>Offset</strong></td>

   <td width="144"><strong>Stack Item</strong></td>

   <td width="168"><strong>Responsibility</strong></td>

  </tr>

  <tr>

   <td width="55">–ve</td>

   <td width="144">Saved Registers</td>

   <td width="168">Callee Saves &amp; Restores</td>

  </tr>

  <tr>

   <td width="55">–ve</td>

   <td width="144">Callee Local Data</td>

   <td width="168">Callee defines and uses</td>

  </tr>

  <tr>

   <td rowspan="2" width="55">0</td>

   <td width="144">Base Pointer of Caller</td>

   <td width="168">Callee Saves &amp; Restores</td>

  </tr>

  <tr>

   <td width="144">Return Address</td>

   <td width="168">Saved by call, used by ret</td>

  </tr>

  <tr>

   <td width="55">+ve</td>

   <td width="144">Return Value</td>

   <td width="168">Callee writes, Caller reads</td>

  </tr>

  <tr>

   <td width="55">+ve</td>

   <td width="144">Parameters</td>

   <td width="168">Caller writes, Callee reads</td>

  </tr>

 </tbody>

</table>

The following points may be noted:

<ul>

 <li>Offsets in the AR are with respect to the Base Pointer of Callee.</li>

 <li>Return Value can alternatively be returned through a register.</li>

 <li>The AR will be populated from the Symbol Table of the function.</li>

 <li>Symbol Tables of nested blocks will be flattened and their variables allocated within the Symbol Table (and hence the AR) of the function where they occur in. Necessary name mangling will be performed to to take care of same lexical name for different variables in different nested scopes.</li>

</ul>

<ul>

 <li><em>Handle global variables </em>(note that local static variables are not allowed in tinyC) as static and generate allocations in static area. This will be populated from global symbol table (ST.gbl).</li>

 <li><em>Generate Constants from Table of Constants </em>– handle string constants as assembler symbols in DATA SEGMENT and integer constants as parts of target code in TEXT SEGMENT.</li>

 <li><em>Register Allocations &amp; Assignment</em>: Create memory binding for variables in registers with the following considerations:

  <ul>

   <li>After a load / store the variable on the activation record and the register will have identical values.</li>

   <li>Registers can be used to store temporary computed values.</li>

   <li>Register allocations are often used to pass int or pointer parameters.</li>

   <li>Register allocations are often used to return int or pointer values.</li>

  </ul></li>

</ul>

<strong>Note: </strong><em>Refer to </em>Run-Time Environment <em>lecture presentations for details and examples on memory binding</em>.

<ol>

 <li><strong>Code Translation</strong>: This deals with the translation of 3-address quad’s to x86-32 assembly code. This needs to handle:

  <ul>

   <li><em>Generation of Function Prologue </em>– few lines of code at the beginning of a function, which prepare the stack and registers for use within the function.</li>

   <li><em>Generate Function Epilogue </em>– appears at the end of the function, and restores the stack and registers to the state they were in before the function was called.</li>

   <li><em>Map 3-address Code to Assembly </em>– to translate the function body using the following rules:

    <ul>

     <li>Choose optimized assembly instructions for every expression, assignment and control quad.</li>

     <li>Use algebraic simplification and reduction of strength for choice of assembly instructions from a quad.</li>

     <li>Use machine idioms (like <strong>inc </strong>for i++, or <strong>add reg, 1 </strong>for ++i).</li>

    </ul></li>

  </ul></li>

</ol>

<strong>Note: </strong><em>Refer to </em>Target Code Generation <em>lecture presentations for details</em>.

<ol>

 <li><strong>Target Code</strong>: Integrate all the above code into an Assembly File for gcc</li>

</ol>

<h1>3           The Assignment</h1>

<ol>

 <li>Write a target code (x86-32) translator from the 3-address quads generated from the flex and bison specifications of tinyC (with restrictions as mentioned in Section 2). Assume that the input tinyC file is lexically, syntactically, and semantically correct. Hence no error handling and / or recovery is expected.</li>

 <li>Prepare a Makefile to compile and test the project.</li>

 <li>Prepare test input files ass6 <em>roll </em>test<em>&lt;</em>number<em>&gt;</em>.c to test the target code translation and generate the translation output in ass6 <em>roll &lt;</em>number<em>&gt;</em>.asm.</li>

 <li>Name your files as follows:</li>

</ol>

<table width="437">

 <tbody>

  <tr>

   <td width="243"><strong>File</strong></td>

   <td width="194"><strong>Naming</strong></td>

  </tr>

  <tr>

   <td width="243">Flex Specification</td>

   <td width="194">ass6 <em>roll</em>.l</td>

  </tr>

  <tr>

   <td width="243">Bison Specification</td>

   <td width="194">ass6 <em>roll</em>.y</td>

  </tr>

  <tr>

   <td width="243">Data Structures (Class Definitions) andGlobal Function Prototypes</td>

   <td width="194">ass6 <em>roll </em>translator.h</td>

  </tr>

  <tr>

   <td width="243">Data Structures, Function Implementations and 3–Address Translator</td>

   <td width="194">ass6 <em>roll </em>translator.cxx</td>

  </tr>

  <tr>

   <td width="243">Target Translator and x86-64 Translator main()</td>

   <td width="194">ass6 <em>roll </em>target translator.cxx</td>

  </tr>

  <tr>

   <td width="243">Test Inputs</td>

   <td width="194">ass6 <em>roll </em>test<em>&lt;</em>number<em>&gt;</em>.c</td>

  </tr>

  <tr>

   <td width="243">3-address Test Outputs</td>

   <td width="194">ass6 <em>roll </em>quads<em>&lt;</em>number<em>&gt;</em>.out</td>

  </tr>

  <tr>

   <td width="243">Test Outputs</td>

   <td width="194">ass6 <em>roll &lt;</em>number<em>&gt;</em>.asm</td>

  </tr>

 </tbody>

</table>

<ol start="5">

 <li>Prepare a zip-archive with the name ass6 <em>roll</em>.zip containing all the files and upload to Moodle.</li>

</ol>

<h1>4           Credits</h1>

Design of Memory Binding:                                                                            <strong>15 + 5 + 5 + 5 + 10 = 40</strong>

<em>Handling of Activation Records</em>

<em>Handling of Nested Symbol Tables</em>

<em>Handling of Static Memory &amp; Binding</em>

<em>Handling of Constants</em>

<em>Handling of Register Allocation &amp; Assignment</em>

Design of Code Translation:                                                                                               <strong>5 + 5 + 10 = 20</strong>

<em>Handling of Prologue</em>

<em>Handling of Epilogue</em>

<em>Handling of Function Body</em>

Design of Target Code Management:                                                                                                           <strong>10</strong>

<em>Integration of translated codes into an assembly file</em>

Design of Test files and correctness of outputs:                                                                 <strong>10 + 10 = 20</strong>

<em>Test at least 5 i/p files covering all rules</em>

<em>Shortcoming and / or bugs, if any, should be highlighted</em>

Integrated interface of the tinyC Compiler: