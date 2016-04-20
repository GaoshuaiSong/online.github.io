
内联函数


1、引入内联函数是为了解决函数调用效率的问题

2、由于函数之间的调用，会从一个内存地址调到另外一个内存地址，当函数调用完毕之后还会返回原来函数执行的地址。函数调用会有一定的时间开销，引入内联函数就是为了解决这一问题。

UIKIT_STATIC_INLINE UIEdgeInsets UIEdgeInsetsMake(CGFloat top, CGFloat left, CGFloat bottom, CGFloat right) {
    UIEdgeInsets insets = {top, left, bottom, right};
    return insets;
}

UIKIT_STATIC_INLINE UIOffset UIOffsetMake(CGFloat horizontal, CGFloat vertical) {
    UIOffset offset = {horizontal, vertical};
    return offset;
}
...

#define UIKIT_STATIC_INLINE static inline

这个函数是一个静态的内联函数！

那么引用内联函数到底有什么区别呢？


“为了解决函数调用效率的问题”


如何解决呢?

#ifndef inline_inline_h
#define inline_inline_h

static inline int add(int a, int b){
    return a+b;
}
#endif

#import <Foundation/Foundation.h>
#import "inline.h"

int main(int argc, const char * argv[]) {
    int c = add(1, 2);
    return 0;
}



查看main.m的汇编文件，如下：

    .section    __TEXT,__text,regular,pure_instructions
    .globl  _main
    .align  4, 0x90
_main:                                  ## @main
Lfunc_begin0:
    .loc    2 14 0                  ## /Users/fenglihai/Desktop/inline/inline/main.m:14:0
    .cfi_startproc
## BB#0:
    pushq   %rbp
Ltmp0:
    .cfi_def_cfa_offset 16
Ltmp1:
    .cfi_offset %rbp, -16
    movq    %rsp, %rbp
Ltmp2:
    .cfi_def_cfa_register %rbp
    ##DEBUG_VALUE: main:argc <- EDI
    ##DEBUG_VALUE: main:argv <- RSI
Ltmp3:
    ##DEBUG_VALUE: main:c <- 3
    xorl    %eax, %eax
    .loc    2 17 5 prologue_end     ## /Users/fenglihai/Desktop/inline/inline/main.m:17:5
Ltmp4:
    popq    %rbp
    retq
    
    
    
#ifndef notInline_Header_h
#define notInline_Header_h

 int add(int a, int b){
    return a+b;
}

#endif


#import <Foundation/Foundation.h>
#import "Header.h"

int main(int argc, const char * argv[]) {
    int c = add(1,2);
    return 0;
}


查看main.m的汇编文件，如下：

    .section    __TEXT,__text,regular,pure_instructions
    .globl  _add
    .align  4, 0x90
_add:                                   ## @add
Lfunc_begin0:
    .file   3 "/Users/fenglihai/Desktop/notInline/notInline" "Header.h"
    .loc    3 12 0                  ## /Users/fenglihai/Desktop/notInline/notInline/Header.h:12:0
    .cfi_startproc
## BB#0:
    pushq   %rbp
Ltmp0:
    .cfi_def_cfa_offset 16
Ltmp1:
    .cfi_offset %rbp, -16
    movq    %rsp, %rbp
Ltmp2:
    .cfi_def_cfa_register %rbp
    movl    %edi, -4(%rbp)
    movl    %esi, -8(%rbp)
    .loc    3 13 5 prologue_end     ## /Users/fenglihai/Desktop/notInline/notInline/Header.h:13:5
Ltmp3:
    movl    -4(%rbp), %esi
    addl    -8(%rbp), %esi
    movl    %esi, %eax
    popq    %rbp
    retq
Ltmp4:
Lfunc_end0:
    .cfi_endproc

    .globl  _main
    .align  4, 0x90
_main:                                  ## @main
Lfunc_begin1:
    .loc    2 12 0                  ## /Users/fenglihai/Desktop/notInline/notInline/main.m:12:0
    .cfi_startproc
## BB#0:
    pushq   %rbp
Ltmp5:
    .cfi_def_cfa_offset 16
Ltmp6:
    .cfi_offset %rbp, -16
    movq    %rsp, %rbp
Ltmp7:
    .cfi_def_cfa_register %rbp
    subq    $32, %rsp
    movl    $1, %eax
    movl    $2, %ecx
    movl    $0, -4(%rbp)
    movl    %edi, -8(%rbp)
    movq    %rsi, -16(%rbp)
    .loc    2 13 13 prologue_end    ## /Users/fenglihai/Desktop/notInline/notInline/main.m:13:13
Ltmp8:
    movl    %eax, %edi
    movl    %ecx, %esi
    callq   _add
    xorl    %ecx, %ecx
    movl    %eax, -20(%rbp)
    .loc    2 14 5                  ## /Users/fenglihai/Desktop/notInline/notInline/main.m:14:5
    movl    %ecx, %eax
    addq    $32, %rsp
    popq    %rbp
    retq
    
上面2段代码可以看出，只有add函数的定义不一样，一个是加了static inline修饰，而另外一个没有。再对比一下汇编代码，发现的确有很大的不一样呀！！！给人第一感觉，有用static inline修饰的汇编之后的代码比没有static inline修饰的的汇编之后的代码简洁的多了！！

其次，在没有调用static inline修饰add函数的main.m汇编代码中，add函数是有单独的汇编代码的！ 
而使用内联函数的main.m汇编代码中，仅仅只有main函数的汇编代码！ 

对比两者的mian.m的汇编代码，可以发现，没有使用`static inline修饰的内联函数的mian函数汇编代码中，会出现 call 指令！这就是区别！调用call指令就是就需要： 
- (1)将下一条指令的所在地址（即当时程序计数器PC的内容）入栈 
- (2)并将子程序的起始地址送入PC（于是CPU的下一条指令就会转去执行子程序）。
