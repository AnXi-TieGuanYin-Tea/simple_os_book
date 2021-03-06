# 【实现】设置栈

只有设置好的合适大小和地址的栈内存空间（简称栈空间），才能有效地进行函数调用。这里为了减少汇编代码量，我们就通过C代码来完成显示。由于需要调用C语言的函数，所以需要自己建立好栈空间。设置栈的代码如下：

	movl    $start, %esp
    
由于start位置（0x7c00）前的地址空间没有用到，所以可以用来作为bootloader的栈，需要注意栈是向下长的，所以不会破坏start位置后面的代码。在后面的小节还会对栈进行更加深入的讲解。我们可以通过用gdb调试bootloader来进一步观察栈的变化：

**【实验】用gdb调试bootloader观察栈信息 **

1. 开两个窗口；在一个窗口中，在proj1目录下执行命令make；
2. 在proj1目录下执行 “qemu -hda bin/ucore.img -S -s”,这时会启动一个qemu窗口界面，处于暂停状态，等待gdb链接；
3. 在另外一个窗口中，在proj1目录下执行命令 gdb obj/bootblock.o；
4. 在gdb的提示符下执行如下命令，会有一定的输出：
```
        (gdb) target remote :1234   #与qemu建立远程链接
        (gdb) break bootasm.S:68    #在bootasm.S的第68行“movl $start, %esp”设置一个断点
        (gdb) continue              #让qemu继续执行  
```
这时qemu会继续执行，但执行到bootasm.S的第68行时会暂停，等待gdb的控制。这时可以在gdb中继续输入如下命令来分析栈的变化：
```   
        (gdb) info registers esp
        esp            0xffd6   0xffd6    #没有执行第68行代码前的esp值
        (gdb) si                          #执行第68行代码
        69        call bootmain
        (gdb) info registers esp
        esp            0x7c00   0x7c00   #当前的esp值，即栈顶
        (gdb) si
        bootmain () at boot/bootmain.c:87    #执行call汇编指令
        87      bootmain(void) {
        (gdb) info registers esp
        esp            0x7bfc   0x7bfc    #当前的esp值0x7bfc, 0x7bfc处存放了bootmain函数的返回地址0x7c4a，这可以通过下面两个命令了解  
        (gdb) x /4x 0x7bfc                  
        0x7bfc: 0x00007c4a      0xc031fcfa      0xc08ed88e      0x64e4d08e
        (gdb) x /4i 0x7c40
           0x7c40 <protcseg+14>:        mov    $0x7c00,%esp
           0x7c45 <protcseg+19>:        call   0x7c6c <bootmain>
           0x7c4a <spin>:       jmp    0x7c4a <spin>
           0x7c4c <gdt>:        add    %al,(%eax)
```

##【提示】

在proj1中执行
```	
    make debug
```        
   则自动完成上述大部分前期工作，即qemu和gdb的加载，且gdb会自动建立于qemu的联接并设置好断点。具体实现可参看proj1的Makefile中于debug相关的内容和tools/gdbinit中的内容。
