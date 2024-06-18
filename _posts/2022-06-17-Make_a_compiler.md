---
title: "Creating a simple C compiler"
date: 2022-06-17
---

## Task 1

### Sample program

task1.c

```c
main()
{
    int sum;
    int i;
    sum = 0;
    for (i = 0; i <= 10; i = i + 1) {
        sum = sum + i;
    }
    put_int(sum);
}
```

Here is a simple program that computes the sum of integers from 0 to 10. Variable `sum` and `i` is defined. In a `for loop`, `i` is incremented and added to `sum`. Lastly, `sum` is printed on the screen.

### `tlc` output

```
FuncTab
 main #1

SymTab
id(1)
 sum #1, offset(-8)
 i #2, offset(-4)
root
 func[ identifier(r0)(main)] ()
  l(3): declaration( identifier(r0)(sum))
  l(4): declaration( identifier(r0)(i))
  l(5): stm_asign( exp_asign(r0)( identifier(r0)(sum) const_int(r1)(0)))
  l(8): for( exp_asign(r0)( identifier(r0)(i) const_int(r1)(0)) lte(r0)( identifier(r0)(i) const_int(r1)(10)) exp_asign(r1)( identifier(r1)(i) add(r0)( identifier(r0)(i) const_int(r1)(1)))
   l(8): list(
    l(7): stm_asign( exp_asign(r1)( identifier(r1)(sum) add(r0)( identifier(r0)(sum) identifier(r1)(i))))
   )
  )
  l(9): stm_asign( call(r0)( identifier(r0)(put_int) ( identifier(r0)(sum))))
```

The output from `tlc` captures some important information. 

- `FuncTab` prints the functions in the code. The sample code only has a main function, thus there is just one entry in `FuncTab`. 

- `SymTab` is symbol table, contains every symbol in the code. 
  
  - The variables used is shown in `id`, and the corresponding offsets are printed. The offset is 4 bytes apart, as the size of an integer is 4 bytes.
  
  - The structure of the code is shown in `root`. 
    
    - `func[]` shows the entry of a function. The name of the function is printed in the brackets. 
    
    - `l()` prints the line number of the corresponding code in the C code.
    
    - `declaration()` denotes where a variable is defined.
    
    - The log that start with `stm_` is a statement, and `exp_` is an expression. In `l(5)`, `sum` is assigned value `0` in `exp_asign()`, inside of `stm_asign()`. There is two layer because `sum = 0` is an expression, and `sum = 0;` is a statement.
    
    - `for()` correspond to the for loop in the code. In the brackets is the loop conditions.
      
      - `lte()` compares two variables. It stands for "less than or equal" (≤). 
      
      - `list()` here contains the body of the loop. It usually represents code that resides inside of a code block `{}`.
    
    - `call()` in `l(9)` is a function call to `put_int`. The function arguments are in the brackets. This line is also inside of `stm_asign()` because it is a statement.

### Assembly

task1.s

```nasm
    .text
    .p2align 2
    .global    _main
_main:
    stp    x29, x30, [sp, -32]!
    add    x29, sp, 32
    mov    w9, 0
    str    w9, [x29, -8]
    mov    w9, 0
    str    w9, [x29, -4]
.L0:
    ldr    w8, [x29, -4]
    mov    w9, 10
    cmp    w8, w9
    b.gt    .L1
    ldr    w8, [x29, -8]
    ldr    w9, [x29, -4]
    add    w8, w8, w9
    str    w8, [x29, -8]
    ldr    w8, [x29, -4]
    mov    w9, 1
    add    w8, w8, w9
    str    w8, [x29, -4]
    b    .L0
.L1:
    sub    sp, sp, #16
    str    w9, [sp, 4]
    str    w10, [sp, 0]
    ldr    w8, [x29, -8]
    mov    w0, w8
    bl    put_int
    ldr    w9, [sp, 4]
    ldr    w10, [sp, 0]
    add    sp, sp, 16
_END_main:
    ldp    x29, x30, [sp], 32
    ret

    .text
    .p2align 2
.LC0:
    .string "%d\n"
    .text
    .p2align 2
put_int:
    sub    sp, sp, #32
    stp    x29, x30, [sp, #16]
    add    x29, sp, #16
    stur    w0, [x29, #-4]
    ldur    w9, [x29, #-4]
    mov    x8, x9
    adrp    x0, .LC0@PAGE
    add    x0, x0, .LC0@PAGEOFF
    mov    x9, sp
    str    x8, [x9]
    bl    _printf
    ldp    x29, x30, [sp, #16]
    add    sp, sp, #32
    ret
```

The above is the assembly output of `tlc` on `arm64` platform. The `_main` block corresponds to the main function. The variables are defined and assigned to zero. `L0` block contains the for loop. `L1` block is where for loop exists, and calls `put_int`. Function `put_int` is hard coded in the assembly. It organize the input into appropriate format and calls `printf` from system library.

## Task 2

### `while` `do-while` difference

`while` loop checks the condition first, and then execute the body. It is entry controlled loop and the body may not be executed. 

```c
while (expression) {
    statements;
}
```

`do-while` loop executes the body, then check the conditions. It is exit controlled loop and the body is at least executed once. 

```c
do {
    statements
} while (expression);
```

### Sample Program

task2.c

```c
main()
{
    // the following code increments `a` until it reaches 10
    int a;
    a = 0;

    // do-while loop
    do {
        a = a + 1;
    } while (a < 10);

    /* while loop version
    while (a < 10) {
        a = a + 1;
    }*/
    put_int(a);
}
```

#### Output of `tlc` without modification

Since the syntax analyzer do not understand `do` keyword, an syntax error is emitted for the `do-while` loop:

`[error 1] line 5: syntax error`

The program output for the while version of the sample code is as follows:

```
FuncTab
 main #1

SymTab
id(1)
 a #1, offset(-4)
root
 func[ identifier(r0)(main)] ()
  l(3): declaration( identifier(r0)(a))
  l(4): stm_asign( exp_asign(r0)( identifier(r0)(a) const_int(r1)(0)))
  l(7): while( lt(r0)( identifier(r0)(a) const_int(r1)(10))
   l(7): list(
    l(6): stm_asign( exp_asign(r1)( identifier(r1)(a) add(r0)( identifier(r0)(a) const_int(r1)(1))))
   )
  )
  l(8): stm_asign( call(r0)( identifier(r0)(put_int) ( identifier(r0)(a))))
```

Assembly output of the code is:

```nasm
    .text
    .p2align 2
    .global    _main
_main:
    stp    x29, x30, [sp, -32]!
    add    x29, sp, 32
    mov    w9, 0
    str    w9, [x29, -4]
.L0:
    ldr    w8, [x29, -4]
    mov    w9, 10
    cmp    w8, w9
    b.ge    .L1
    ldr    w8, [x29, -4]
    mov    w9, 1
    add    w8, w8, w9
    str    w8, [x29, -4]
    b    .L0
.L1:
    sub    sp, sp, #16
    str    w9, [sp, 4]
    str    w10, [sp, 0]
    ldr    w8, [x29, -4]
    mov    w0, w8
    bl    put_int
    ldr    w9, [sp, 4]
    ldr    w10, [sp, 0]
    add    sp, sp, 16
_END_main:
    ldp    x29, x30, [sp], 32
    ret

    .text
    .p2align 2
.LC0:
    .string "%d\n"
    .text
    .p2align 2
put_int:
    sub    sp, sp, #32
    stp    x29, x30, [sp, #16]
    add    x29, sp, #16
    stur    w0, [x29, #-4]
    ldur    w9, [x29, #-4]
    mov    x8, x9
    adrp    x0, .LC0@PAGE
    add    x0, x0, .LC0@PAGEOFF
    mov    x9, sp
    str    x8, [x9]
    bl    _printf
    ldp    x29, x30, [sp, #16]
    add    sp, sp, #32
    ret
```

The `while` loop is executed in the label `.L0`. First, `a < 10` is computed in `cmp w8, w9`. If `a` is found to be smaller, the body is executed. Else, jump to `.L1`, which exits the loop. At the end of `.L0` block, `b .L0` tells the CPU to jump back to the beginning of the block, which makes a loop.

In the `do-while` version of the assembly, the body should be executed first. The assembly for the loop will resemble the following.

```nasm
.L0:
    ldr    w8, [x29, -4] ; load a
    mov    w9, 1
    add    w8, w8, w9    ; add 1 to a
    str    w8, [x29, -4] ; store a
    mov    w9, 10
    cmp    w8, w9        ; compare a & 10
    b.ge    .L1          ; exit if false
    b    .L0             ; loop
```

### Modify source code

The code for `dowhile` is mostly similar to `while` loop. The difference is mainly the order of statement and expression, and the order of execution.

#### tl_gram.y

```c
...
iteration_statement
    : TOKEN_WHILE TOKEN_LPAREN expression TOKEN_RPAREN statement
    { $$ = act_while_stm($3, $5); }
    | TOKEN_FOR TOKEN_LPAREN expression TOKEN_SEMICOLON expression TOKEN_SEMICOLON expression TOKEN_RPAREN statement
    { $$ = act_for_stm($3, $5, $7, $9); }
/** REPORT3
    このあたりにdo-while文のルールを追加する
    Add a rule for do-while statement
 */
    | TOKEN_DO statement TOKEN_WHILE TOKEN_LPAREN expression TOKEN_RPAREN TOKEN_SEMICOLON
    { $$ = act_dowhile_stm($2, $5); }
...
```

The semantics of `dowhile` is similar to that of `while`; the order of the statements are different and `TOKEN_DO` is added is the front.

#### parse_action.c

```c
...
/* REPORT3
   ここにdo-while文のアクション関数を記述する
   Add an action function for do-while statement
*/
AST_Node*
act_dowhile_stm(AST_Node *s, AST_Node *e)
{
    AST_Node *ret = create_AST_Stm(AST_STM_DOWHILE, yylineno);
    ret->child[0] = s;
    ret->child[1] = e;
    if (s != NULL) {
        s->parent = ret;
    }
    if (e != NULL) {
        e->parent = ret;
    }
    return ret;
}
...
```

The function here is mostly same as `while`. The order of `s` and `e` is reversed.

#### parse_action.h

```c
...
extern AST_Node  *act_while_stm(AST_Node *e, AST_Node *s);
extern AST_Node  *act_for_stm(AST_Node *e1, AST_Node *e2, AST_Node *e3, AST_Node *s);
/* REPORT3
   ここにアクション関数のプロトタイプ宣言を追加する
   Add function prototype declaration(s)
*/
extern AST_Node  *act_dowhile_stm(AST_Node *s, AST_Node *e);
extern AST_Node  *act_return_stm(AST_Node *e);
...
```

This is the header file that contains function prototype.

#### cg.c

```c
...
    case AST_STM_DOWHILE:
    /* REPORT3
       このあたりにdo-while文ノード用のレジスタ割り付け巡回処理を追加する
       Add traverse code for register assignment to do-while statement
    */
        traverse_ast_stm(s->child[0], pass);
        traverse_ast_exp(s->child[1], pass);
        break;
...
```

```c
void
gen_stm_dowhile(FILE *out, AST_Node *s)
{
    /* REPORT3
       ここにdo-while文のコード生成処理を追加する
       Add code-generation code for do-while
    */
    int  l_begin, l_exit;

    l_begin = get_label();
    l_exit = get_label();
    gen_label_stm(out, l_begin);
    gen_stm(out, s->child[0]);
    gen_exp(out, s->child[1]);
    gen_stm_rel(out, s->child[1], l_exit);
    gen_insn_jmp(out, gen_label(l_begin));
    gen_label_stm(out, l_exit);
}
```

Here, the logic of `dowhile` is defined. The order of execution is slightly different from `while`: function body is executed first, then the loop condition is examined.

#### ast.c

```c
    ...
    case AST_STM_DOWHILE:
        /* REPORT3
         * このあたりにdo-whileノード用のダンプ処理を追加する
         * Add output dump code for do-while here
         */
        dump_ast_stm(s->child[1]);
        indent_count++;
        fputs("\n", stderr);
        dump_ast_exp(s->child[0]);
        indent_count--;
        indent();
        break;
    ...
```

The log output is defined here. Modified slightly to make the output clear.

### Output of `tlc` with modification

`tlc` log:

```
FuncTab
 main #1

SymTab
id(1)
 a #1, offset(-4)
root
 func[ identifier(r0)(main)] ()
  l(3): declaration( identifier(r0)(a))
  l(4): stm_asign( exp_asign(r0)( identifier(r0)(a) const_int(r1)(0)))
  l(7): dowhile( lt(r0)( identifier(r0)(a) const_int(r1)(10))
   l(7): list(
    l(6): stm_asign( exp_asign(r1)( identifier(r1)(a) add(r0)( identifier(r0)(a) const_int(r1)(1))))
   )
  )
  l(8): stm_asign( call(r0)( identifier(r0)(put_int) ( identifier(r0)(a))))
```

Assembly output:

```nasm
    .text
    .p2align 2
    .global    _main
_main:
    stp    x29, x30, [sp, -32]!
    add    x29, sp, 32
    mov    w9, 0
    str    w9, [x29, -4]
.L0:
    ldr    w8, [x29, -4]
    mov    w9, 1
    add    w8, w8, w9
    str    w8, [x29, -4]
    ldr    w8, [x29, -4]
    mov    w9, 10
    cmp    w8, w9
    cset    w8, lt
    b.ge    .L1
    b    .L0
.L1:
    sub    sp, sp, #16
    str    w9, [sp, 4]
    str    w10, [sp, 0]
    ldr    w8, [x29, -4]
    mov    w0, w8
    bl    put_int
    ldr    w9, [sp, 4]
    ldr    w10, [sp, 0]
    add    sp, sp, 16
_END_main:
    ldp    x29, x30, [sp], 32
    ret

    .text
    .p2align 2
.LC0:
    .string "%d\n"
    .text
    .p2align 2
put_int:
    sub    sp, sp, #32
    stp    x29, x30, [sp, #16]
    add    x29, sp, #16
    stur    w0, [x29, #-4]
    ldur    w9, [x29, #-4]
    mov    x8, x9
    adrp    x0, .LC0@PAGE
    add    x0, x0, .LC0@PAGEOFF
    mov    x9, sp
    str    x8, [x9]
    bl    _printf
    ldp    x29, x30, [sp, #16]
    add    sp, sp, #32
    ret
```

In block `.L0`, the body of the loop is executed first, and then the loop condition is examined. The rest of the assembly is strictly the same.

Program output:

```
./task2
10
```

### Discussion

Closely examining the assembly code generated, one could notice that currently `tlc` is only translating the `c` code into assembly at a very surface level, line by line. The generated assembly has many repeated or redundant segments that could be optimized. For example, the `do-while` loop generated by `tlc` is

```nasm
.L0:
    ldr    w8, [x29, -4]
    mov    w9, 1
    add    w8, w8, w9
    str    w8, [x29, -4]
    ldr    w8, [x29, -4]
    mov    w9, 10
    cmp    w8, w9
    cset    w8, lt
    b.ge    .L1
    b    .L0
```

It could be greatly optimized into

```nasm
    ldr w8, [x29, -4]
    mov w9, 1
    mov w10, 10
.L0:
    add w8, w8, w9
    cmp w9, w10
    b.ge    .L1
    b    .L0
```

10 lines of code in `.L0` could be reduced to 4. In modern compilers, optimization is done extensively, and the resulting assembly is often more efficient than a handwritten counterpart.

## Task 3

### Line Comment

Any code without comments is essentially a disaster. Thus I sincerely believe comment is the first thing that needs to be supported. Thankfully, line comment is a rather simple functionality to support.

```
"//".*"\n" ;
```

By adding above to the end of `tl_lex.l`, any character after `//` is not processed by the parser. Comment lines are ignored, thus will not appear in the log.

### Pointer Support

Pointer is an object that stores memory address. Using pointers, programmers can work with the memory directly, and pass arguments to functions by reference.

#### How pointers should be handled

To add the pointer support to `tlb`, it is first necessary to understand how pointers are handled on the assembly level. Thus, I took assembly generated from `clang`  as reference and came up with the following conclusion.

First, not all registers are the same. The `w` registers currently used for arithmetic calculation is 32 bit. It can count up to $2^{32}$, but is still not sufficient to store the memory address of a modern computer. Thus, the 64 bit `x` registers needs to be used for any pointer operations. The `x` and `w` registers occupy the same space, and is related to each other in the following way.

![register_x_w.png](/assets/register_x_w.png)

To make pointers functional, the following syntax needs to be supported: 

- `*p` loading and storing

- `&v` loading. 

```nasm
ldr x8, [sp]    ; load the pointer into register x8
ldr w9, [x8]    ; load the value from the pointer
```

In `clang`, when `*p` is invoked, it is converted into the assembly above (registers used will depend on the code). The process is similar to loading a normal variable, the difference is that the loading happens twice. Note that the register that stores the pointer temporarily is a 64 bit `x` register

```nasm
ldr x9, [sp]    ; load the pointer into register x9
str w8, [x9]    ; store value into the address stored in pointer
```

When a value is assigned to `*p`, the assembly above is generated. This is a two step process as well. The pointer is loaded, and then a value is stored into the address stored in the pointer.

```nasm
add x9, sp, #4
```

`&v` gives the address of the variable. The address of a variable is calculated by adding the base address with the corresponding offset. In the assembly above, `sp` stores the base address, and `#4` is the offset. The resulting address is stored into `x9`.

#### Code modification

The important changes to the program is shown below. Some not so crucial changes, such as the header files and adding to type enum, is abbreviated.

##### tl_gram.y

Now we understand how pointers should be translated, the lex file is first modified to accommodate with the added pointer grammar.

```diff
diff --git a/tl_gram.y b/tl_gram.y
index 12a0fa0..9f5c85a 100644
--- a/tl_gram.y
+++ b/tl_gram.y
@@ -61,6 +61,7 @@ extern int  yylineno;
 %token  TOKEN_LBRACE
 %token  TOKEN_RBRACE
 %token  TOKEN_SEMICOLON
+%token  TOKEN_AMPERSAND

 %token  TOKEN_ELSE
 %token  TOKEN_FOR
@@ -106,6 +107,10 @@ expression
 identifier
        : TOKEN_ID
        { $$ = act_ID($1); }
+       | TOKEN_ASTERISK TOKEN_ID
+       { $$ = act_ID_P($2); }
+       | TOKEN_AMPERSAND TOKEN_ID
+       { $$ = act_ID_A($2); }

 primary_expression
        : identifier
@@ -180,10 +185,8 @@ assignment_expression
        { $$ = act_expr_n2(AST_EXP_ASGN, $1, $3); }

 declaration
-       : TOKEN_INT identifier_list TOKEN_SEMICOLON
-       { $$ = act_dec_int($2); }
+       : TOKEN_INT identifier_list TOKEN_SEMICOLON
+       { $$ = act_dec($2); }

 identifier_list
        : identifier
```

- Ampersand symbol `&` is added. 

- `*p` pointer and `&v` address is added to identifier

- Function name for declaring variables is changed (as integer is not the only variable type anymore)

##### parse_action.c

The input is converted into  `AST_Node` with corresponding types in `parse_action.c`.

```diff
diff --git a/parse_action.c b/parse_action.c
index 63a6ddd..b963f58 100644
--- a/parse_action.c
+++ b/parse_action.c
@@ -36,6 +36,34 @@ act_ID(char *id)
     return ret;
 }

+AST_Node*
+act_ID_P(char *id)
+{
+    char* str;
+    AST_Node *ret;
+    ret = create_AST_Exp(AST_EXP_IDENT_P);
+    if ((str = strdup(id)) == NULL) {
+        fprintf(stderr, "Not enough memory for strdup.\n");
+        abort();
+    }
+    ret->str = str;
+    return ret;
+}
+
+AST_Node*
+act_ID_A(char *id)
+{
+    char* str;
+    AST_Node *ret;
+    ret = create_AST_Exp(AST_EXP_IDENT_A);
+    if ((str = strdup(id)) == NULL) {
+        fprintf(stderr, "Not enough memory for strdup.\n");
+        abort();
+    }
+    ret->str = str;
+    return ret;
+}
+
 AST_Node*
 act_const_int(int c)
 {
@@ -89,12 +117,17 @@ act_expr_n2(int ope, AST_Node *n1, AST_Node *n2)
 }

 AST_Node*
-act_dec_int(AST_Node *d)
+act_dec(AST_Node *d)
 {
     AST_Node *n;
     AST_Node *ret = create_AST_Stm(AST_STM_DEC, yylineno);
     for (n = d; n != NULL; n = n->child[0]) {
-        if (append_sym(TYPE_INT, SYM_AUTOVAR, n->str) == 0) {
+        int type = TYPE_NONE;
+        if (n->sub_kind == AST_EXP_IDENT)
+            type = TYPE_INT;
+        else if (n->sub_kind == AST_EXP_IDENT_P)
+            type = TYPE_PTR;
+        if (append_sym(type, SYM_AUTOVAR, n->str) == 0) {
             fprintf(stderr, "Duplicate variable declaration: %s\n", n->str);
             yynerrs++;
         }
```

- The parser for `*` and `&` is added with the corresponding types. 

- Parser logic for declaring variables is updated to record the type of the variables

##### cg.c

```diff
diff --git a/cg.c b/cg.c
index 6709a54..3edf1fa 100644
--- a/cg.c
+++ b/cg.c
@@ -504,6 +504,10 @@ gen_exp(FILE *out, AST_Node *e)
         gen_exp_asgn(out, e);
     } else if (e->sub_kind == AST_EXP_IDENT) {
         gen_exp_ident(out, e);
+    } else if (e->sub_kind == AST_EXP_IDENT_P) {
+        gen_exp_ident_p(out, e);
+    } else if (e->sub_kind == AST_EXP_IDENT_A) {
+        gen_exp_ident_a(out, e);
     } else if (e->sub_kind == AST_EXP_CNST_INT) {
         gen_exp_cnst(out, e);
     } else if (e->sub_kind == AST_EXP_CALL) {
@@ -517,10 +521,19 @@ void
 gen_exp_asgn(FILE *out, AST_Node *e)
 {
     gen_exp(out, e->child[1]);
-    if (e->child[0]->sub_kind != AST_EXP_IDENT) {
+    int wide;
+    if (e->child[0]->symtab->type == TYPE_PTR) {
+        wide = 1;
+    } else {
+        wide = 0;
+    }
+    if (e->child[0]->sub_kind == AST_EXP_IDENT) {
+        gen_insn_store_lvar(out, wide, e->child[1]->reg, e->child[0]->symtab->offset);
+    } else if (e->child[0]->sub_kind == AST_EXP_IDENT_P) {
+        gen_insn_store_pvar(out, e->child[1]->reg, e->child[0]->symtab->offset);
+    } else {
         errexit("Invalid destination operand for assign.", __FILE__, __LINE__);
     }
-    gen_insn_store_lvar(out, e->child[1]->reg, e->child[0]->symtab->offset);
 }
```

- add logic to generate different assembly for pointers and addresses
- use `x` register if type is pointer

##### arch_arm64.c

```diff
diff --git a/arch_arm64.c b/arch_arm64.c
index d1acd3d..9d1ee17 100644
--- a/arch_arm64.c
+++ b/arch_arm64.c
@@ -63,6 +63,7 @@ const char CALL_OP[]      =  "bl";


 char reg_name[][10] = {"w8", "w9", "w10"};
+char reg_x_name[][10] = {"x8", "x9", "x10"};
 char param_reg_name[][10] = {"NULL", "w0", "w1", "w2", "w3", "w4", "w5",
                              "w6", "w7" };

@@ -176,8 +177,14 @@ arch_assign_memory(SymTab *symtab)
         if (t->kind == SYM_ARG) {
             t->argid = ++id_arg;
         } else if (t->kind == SYM_AUTOVAR) {
-            t->offset = ++id_var;
-            t->argid = 0;
+            if (t->type == TYPE_INT) {
+                t->offset = ++id_var;
+                t->argid = 0;
+            } else if (t->type == TYPE_PTR) {
+                id_var += 2;
+                t->offset = id_var;
+                t->argid = 0;
+            }
         }
     }
     poffset = (id_arg > 8) ? (id_arg-8)*(-8) : 0;
@@ -268,6 +275,21 @@ gen_exp_ident(FILE *out, AST_Node *idnt)
             reg_name[idnt->reg], idnt->symtab->offset);
 }

+void
+gen_exp_ident_p(FILE *out, AST_Node *idnt)
+{
+    fprintf(out, "\tldr\tx11, [x29, %d]\n"
+                 "\tldr\t%s, [x11]\n",
+            idnt->symtab->offset, reg_name[idnt->reg]);
+}
+
+void
+gen_exp_ident_a(FILE *out, AST_Node *idnt)
+{
+    fprintf(out, "\tadd\t%s, x29, %d\n",
+            reg_x_name[idnt->reg], idnt->symtab->offset);
+}
+
 int
 gen_call_prologue(FILE *out, AST_Node *e, int *padsize, int *framesize)
 {
@@ -342,9 +364,18 @@ gen_call_epilogue(FILE *out, AST_Node *e, int padsize, int framesize)
   store local variable
 */
 void
-gen_insn_store_lvar(FILE* out, int reg, int offset)
+gen_insn_store_lvar(FILE* out, int wide, int reg, int offset)
+{
+    char (*reg_a)[10];
+    if (wide) reg_a = reg_x_name;
+    else reg_a = reg_name;
+    fprintf(out, "\tstr\t%s, [x29, %d]\n", reg_a[reg], offset);
+}
+void
+gen_insn_store_pvar(FILE* out, int reg, int offset)
 {
-    fprintf(out, "\tstr\t%s, [x29, %d]\n", reg_name[reg], offset);
+    fprintf(out, "\tldr\tx11, [x29, %d]\n"
+                 "\tstr\t%s, [x11]\n", offset, reg_name[reg]);
 }
```

- `register_x_name`: 64bit registers for pointer operation

- `arch_assign_memory`: assign 8 bytes of memory if the type of the variable is pointer (otherwise the pointer value might be overwritten by other variables)

- `gen_exp_ident_p`: generate expression for pointer. The temporary value is stored in `x11`, so that it does not overwrite other loaded variables.

- `gen_exp_ident_a`: generate expression for address. The output is stored in `x` registers (does not fit in `w`).

- `gen_insn_store_lvar`: store value into a variable. If the variable is a pointer, the `x` registers are used.

- `gen_insn_store_pvar`: store value into a pointer. Since only integer will be stored, `w` registers are used.

#### Testing & result

To examine the function of the modified program, a sample program is written and tested.

```c
main ()
{
    int i, *j;
    i = 1;
    j = &i;
    i = i + 1;
    *j = *j + 1;
    put_int(i);
}
```

The program above defines integer `i` and a pointer `j`. `j` points to `i`. `i` is incremented by 1, and then `j` increments its value by 1. In the end, `i` should become 3.

```
FuncTab
 main #1

SymTab
id(1)
 i #1, offset(-12)
 j #2, offset(-4)
root
 func[ identifier(r0)(main)] ()
  l(3): declaration( identifier(r0)(i identifier(r0)(*j)))
  l(4): stm_asign( exp_asign(r0)( identifier(r0)(i) const_int(r1)(1)))
  l(5): stm_asign( exp_asign(r0)( identifier(r0)(j) identifier(r1)(&i)))
  l(6): stm_asign( exp_asign(r1)( identifier(r1)(i) add(r0)( identifier(r0)(i) const_int(r1)(1))))
  l(7): stm_asign( exp_asign(r1)( identifier(r1)(*j) add(r0)( identifier(r0)(*j) const_int(r1)(1))))
  l(8): stm_asign( call(r0)( identifier(r0)(put_int) ( identifier(r0)(i))))
```

The log from `tlc` correctly captures everything in the test code. It also marks pointers with `*` and addresses with `&`. The log also correctly captures that `j` is a pointer that is 8 bytes in size (memory address is 8 bytes long in 64-bit system).

```
➜  test git:(master) ✗ ./task3
3
```

The output of `tlc` is compiled into a binary and executed. The result is 3, which is the correct answer. 

I have tested with other C code as well, and have found no major problems.

#### Issues encountered in the process

In the process of writing code, I have encountered numerous problems and bugs. Some are errors from `tlb`, and some are more thorny seg-fault from the generated assembly. Thankfully, using debugger and comparing the assembly to that generated by `clang` helped me solve many problems quickly. Here are some big issues I had.

- `gcc` refuse to compile assembly to binary. This is because wrong registers are used to store memory addresses.

- Variables are randomly set to 0. This is because 4 byte offset is mistakenly used for pointers. When writing to a pointer, integer variable stored nearby is overwritten.

- `Invalid statement kind` error from `tlb`. This can occur when the `sub_kind` of a `AST_Node` is wrong. This is sometimes hard to debug as the frame is usually buried in a pile of recursion.

#### Existing Issues

Pointers currently only work in function body; strange problems can occur if passed into functions. For example, sometimes when I attempt to print pointers using `put_int()`, it works without issue. But sometimes the program will emit `segmentation fault`. I examined the binary using `lldb` and found that this might be due to some registers changing value when calling a function. However this does not explain why sometimes passing pointer to function works. 

For now, I will leave this issue for next time.
