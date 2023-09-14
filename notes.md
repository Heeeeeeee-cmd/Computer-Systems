## <center>proj总结</center>
### proj 0
下载 [Nand2Tetris](URL 'https://drive.google.com/open?id=1xZzcMIUETv3u3sdpM_oTJSTetpVee3KZ')之后，进入到nand2tetris/projects/00 将file文件解压为project0.**zip**文件(.rar会0分的...)submit到网站上就ok了

---

### proj 1（门电路）
- 所有逻辑都可以用与或非三种门来表达出来：$And$ $Or$ $Not$
- **基本公式**：
- 分配律：
    >$x And (y Or z) = (x And y) Or (x And z)$
    >$x Or (y And z)) = (x Or y) And (x Or z)$
- 迪摩根定律:
    >$Not(x And y) = Not(x) Or Not(y)$
    >$Not(x Or y) = Not(x) And Not(y)$
- $Nand$(与非门): $a$ $Nand$ $b$ $=$ $Not(a$ $And$ $b)$
  | a   | b   | ans |
  | --- | --- | --- |
  | 0   | 0   | 1   |
  | 0   | 1   | 1   |
  | 1   | 0   | 1   |
  | 1   | 1   | 0   |
迪摩根定律将与门和或门联系起来，所以现在我们只要两种门$ (And和Not)$就可以将所有逻辑表达式表示出来. 同时因为$a$ $And$ $a$ $=$ $a$,我们可以得到 $Not(a)=a $ $Nand$ $a$,并且由于 $Nand$ 的定义：$a$ $Nand$ $b$ $=$ $Not(a$ $And$ $b)$ 我们可以将 $Not$ 和 $And$ 用 $Nand$ 来表示，故所有逻辑门都可以用与非门来表达出来。
- 根据真值表写逻辑表达式：
    | a   | b   | c   | ans |
    | --- | --- | --- | --- |
    | 0   | 0   | 0   | 1   |
    | 0   | 0   | 1   | 0   |
    | 0   | 1   | 1   | 0   |
    | 0   | 1   | 1   | 0   |
    | 1   | 0   | 0   | 1   |
    | 1   | 0   | 1   | 1   |
    | 1   | 1   | 1   | 0   |
    | 1   | 1   | 1   | 0   |
    >将真值中出现为1的值变量用 $And$ 连接整体用 $Or$ 连接. 说不明白直接举例子...如上图：
    $ (Not(a)$ $And$ $Not(b)$ $And$ $Not(c))$ $Or$ $(a And$ $Not(b)$ $And$ $Not(c))$ $Or$ $(a$ $And$ $Not(b)$ $And$ $c)$
- 第一章的Proj没什么难点，大多都可以根据真值表手推逻辑表达式后一律 $Nand$ 要多利用 $Mux$ 和 $Dmux$ 来进行分区会使一些门写起来简单一点。
     简单说说 $Mux4Way16$
    ```c
    //根据sel的后一位用2选1选出对应的结果p1和p2
    Mux16(a = a, b = b, sel = sel[0], out = p1); 
    Mux16(a = c, b = d, sel = sel[0], out = p2);

    //根据sel的第一位用2选1选出最终结果
    Mux16(a = p1, b = p2, sel = sel[1], out = out);
    ```
    Mux4Way16同理可借用Mux4Way16按照sel后两位筛出上半区结果和下半区结果(首位为0和1)后用2选1选到out中.
---

### proj 2（简易Alu）
- 数字的表示：一般的二进制数只表示正数，那么负数该如何表示呢？我们想到可以减少正数的规模从而一些负数就可以进行表示了。最容易想到的的就是反码：利用数字最高位来表示数字的正负，但这样有个问题：那就是0的处理：例1000和0000分别表示-0和+0，但是0只有一个值不分正负，所以补码就出现了，以4位2进制举例：在计算中我们发现：$6 - 2 = 6+14($ 溢出舍去 $)$ 下面给出补码的定义：
- 
    >$code(–x) = (2^n – x)$
    
<img decoding="async" src="D:\STUDY\Computer Systems\picture\屏幕截图 2023-07-22 210213.png">

这是我觉得讲的最不错的一个点，如图所示：$2^n-1$ 就是$n$位二进制全为$1$的数字，$(2^n-1) - x$ 全1的二进制数字减掉一般的二进制表达就是该数字**按位取反**最后再$+1$就是该数字负数的补码。大学学这个的时候只记得老师讲的按位取反加一了，为啥要这么做我记得他也没讲......
- proj2可能也没啥难度，半加器就是 计算一位加法，例如$a+b=c$其中$c$为本位的结果，$carry$为进位。全加器就是计算时带上前一位的$carry$进位，结果同样为$carry$和$c$.
- 这个alu也就按照给的表格来一步步判断进行二选一计算就可以
    ``` c
    // if (zx == 1) set x = 0        // 16-bit constant
    // if (nx == 1) set x = !x       // bitwise not
    // if (zy == 1) set y = 0        // 16-bit constant
    // if (ny == 1) set y = !y       // bitwise not
    // if (f == 1)  set out = x + y  // integer 2's 
    //complement addition
    // if (f == 0)  set out = x & y  // bitwise and
    // if (no == 1) set out = !out   // bitwise not
    // if (out == 0) set zr = 1
    // if (out < 0) set ng = 1
    ```
---

### proj 3（时序电路、寄存器）
逻辑电路就是输入随着输出的变化而随之改变(无时间效应，较为理想)然而在实际上：
<img decoding="async" src="D:\STUDY\Computer Systems\picture\屏幕截图 2023-07-28 184918.png">
每次时钟周期在从$t$跳变到$t+1$时并不是瞬时的，而是如上图所示会有一个过程的。如果控制好时钟周期（中间跳变的时间不计入周期内）我们就可以认为每次都只需处理逻辑电路而不考虑时延。
- 时序电路：首先引入一个 $DDF$ 的逻辑门：逻辑意义为可以保存上一位的寄存器：$out(t)=in(t-1)$ (个人理解每次该门的逻辑为：将储存在该chip里的数据输出并且同时做输入端的逻辑处理，经过逻辑运算后储存在该chip内部) 没$DDF$的图，用register来表示一下...
<img decoding="async" src="D:\STUDY\Computer Systems\picture\微信图片_20230914193121.png">
- 有了该DDF后我们就可以在输入端进行很多操作了：
```
    If load[t] == 1 then out[t+1] = in[t]
    else out does not change (out[t+1] = out[t])
```

这就是寄存器的处理方法：如果load一直为0那么out就一直保持$out(t)=in(t-1)$ 如果load为1会将in储存在register中，下一时间将其输出。
proj4的**RAM**和**PC**还是比较难的：

- RAM8是由8个16位register组成的，所以需要用3位adress来表示具体选择的register是哪一个，在处理的时候我们可以将load按$Dmux8way$来依次分配到8个register上，只有**一个**(正确的)的register才会因为load的数值来进行输出或者保持，后再按照address的逻辑用$Mux$ 将8个结果选择正确的那一个作为$out$进行输出。**一开始我将load全部接到了8个register上，注意这是错的。因为当load为1时，上述操作会将RAM8中的8个register全部加载数值，事实上只应该有一个register在拥有该值。之后如果load为0，adress选择到了另一个register。输出端会输出该寄存器所保存的值，但事实上该寄存器不应该保存值**
- RAM64就是由8个RAM8组成的，要注意的点有两个(其中一个上面讲过了),8个RAM8组成那就是有6位address.其中前3位$addr[3..5]$是表示正确的结果在哪一个RAM8上。后面的3位 $addr[0..2]$表示正确的结果在RAM8的哪一个reg上。故在操作时我们可以先用Dmux8将addr的前三位引入到8个RAM8上，后对每个RAM8使用后两位addr输出“正确结果”（**正确的结果就在这8个RAM8的同一个reg位置上，但只有一个是正确的**）再通过addr前三位使用8选1Mux将正确的结果接到out上。以后的每个RAM$x$都可以借此思路来操作。

- 对PC的操作还是挺难的...直接上图借鉴正确做法hh
- <img decoding="async" src="D:\STUDY\Computer Systems\picture\屏幕截图 2023-07-31 101901.png">
```
Add16(a = preout, b[0] = true, b[1..15] = false, out = incout); //inc
    //Inc16(in = preout, out = incout);
    Mux16(a = preout, b = incout, sel = inc, out = tmp);  //inc 和 keep二选一  ->ans1
    Mux16(a = tmp, b = in, sel = load, out = loadout); //load ans1二选一  ->ans2
    Mux16(a = loadout, b = false, sel = reset, out = nowout); //ans2 和置0二选一 ->nowout 
    
    //本次的输出 作为输入到寄存器中
    Register(in = nowout, load = true, out = out, out = preout);
```
- 主要是最后的用register处理一下没想到...写好后应该不太逻辑不难懂。就是依次往上选择，每次都用二选一，后对本次的结果放到到reg中。
---


