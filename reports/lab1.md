# LAB1报告

注意点有几个，第一是不要产生递归调用，可能会导致trace自己trace自己。

第二就是放的位置，如果注入到syscall crate里就显得很臃肿混乱，我选择注入到trap_handler里，结果还挺合适，再具体调用之前直接update就不会忘记trace的调用。

还有trace自身有一个获取过程，我搞了第二个get函数专门获取，不增加，感觉还是很清爽的。

## 简答题

1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容（运行 三个 bad 测例 (ch2b_bad_*.rs) ）， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。

答：rustsbi 0.2.0-alpha.2。判例ch2b_bad_address.rs会触发Trap::Exception(Exception::StorePageFault)，当用户程序试图访问无效地址 0x0 时，RISC-V CPU 会触发页错误异常。ch2b_bad_instructions.rs和ch2b_bad_register.rs是让CPU检测到特权指令之后trap进IllegalInstruction异常里，ch2b_bad_instructions.rs 测试内核是否能捕获用户态执行 sret 指令，ch2b_bad_register.rs 测试内核是否能捕获用户态访问特权寄存器。

2. 深入理解 trap.S 中两个函数 __alltraps 和 __restore 的作用，并回答如下问题:

    1. L40：刚进入 __restore 时，sp 代表了什么值。请指出 __restore 的两种使用情景。

    内核栈区。从内核态切换到用户态都会调用__restore，__alltraps是陷入系统态，二者互为两面。启动第一个用户程序和从内核态返回到用户态都是__restore的活。

    2. L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。

    ld t0, 32*8(sp)
    csrw sstatus, t0
    这是sstatus寄存器，控制CPU状态（SPP：指示之前的特权级（0=用户态，1=监督者态），SPIE：指示之前是否启用了中断，SIE：控制是否启用中断）

    ld t1, 33*8(sp)
    csrw sepc, t1
    保存发生异常时的指令地址，决定 sret 后返回到用户态的哪条指令

    ld t2, 2*8(sp)
    csrw sscratch, t2
    sscratch 是一个通用寄存器，在 rCore 中用于保存用户栈指针

    3. L50-L56：为何跳过了 x2 和 x4？

    ld x1, 1*8(sp)
    ld x3, 3*8(sp)
    .set n, 5
    .rept 27
    LOAD_GP %n
    .set n, n+1
    .endr
    4. L60：该指令之后，sp 和 sscratch 中的值分别有什么意义？

    ```asm
    # now sp->kernel stack, sscratch->user stack
    csrrw sp, sscratch, sp
    ```

    这是__alltraps后面的，进入了系统态，sp指向内核栈区，sscratch是保存用户栈指针的。

    5. __restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？

    6. L13：该指令之后，sp 和 sscratch 中的值分别有什么意义？

    ```asm
    csrrw sp, sscratch, sp
    # now sp->kernel stack, sscratch->user stack
    ```
    跟4一样

    7. 从 U 态进入 S 态是哪一条指令发生的？

    __alltraps进去的

## 荣誉准则

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

无

2. 此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

[5exercise](https://learningos.cn/rCore-Tutorial-Guide-2025S/chapter3/5exercise.html)
[6answer](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/6answer.html)

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。