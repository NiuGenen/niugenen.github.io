# 总结

## 安全

Nightingale【2017】memvisor【2012/2014】Multimem【2012】CrossIF【2010】system call check【2009】

## 静态翻译器

BabelFish【2017】

## 翻译器

DistriBit【2012/2010】，MTCrossBit【2011/2009/2008】，GXBit【2011/2005】，CacheBit【2009】，vBTrans【2008】Co-design CrossBit【2008】CrossBit【2007】

## 优化

Fast Return【2010/2009】热代码集中【2011/2010】HW-profile【2009】Condition Codes【2009】硬件查找翻译，与执行分离【2009/2008】profile hot path【2008】Code Cache替换【2008】

## 特定问题

大小端【2011】调试Guest【2008】浮点【2008】



# 简单列表

2017

- 【安全】Nightingale，针对VM解释器的代码保护方案，用DBT来简化VM
- 【BT】BabelFish，轻量级静态二进制翻译器，将MIPS转换为LLVM-IR

2014

- 【安全】memvisor，对内存进行备份用于恢复，通过静态二进制翻译的方式处理访存来备份

2012

- 【安全】Multimem，同样通过静态二进制翻译器实现备份
- 【安全】Memvisor
- 【BT】DistriBit，分布式的CrossBit，两层TCache设计（全局+本地）

2011

- 【特定问题】针对大小端的处理，两种方式，byte swap 和 address swizzling
- 【优化】执行第一次收集数据，然后静态分析将热代码集中（提高局部性）
- 【BT】MTCrossBit，扩展CrossBit支持多线程，一个执行，一个优化，两个TCache，ASLC机制同步
- 【BT】GXBit，扩展CrossBit进行自动并行化，生成PTX代码来运行在GPU上

2010

- 【优化】热代码集中
- 【优化】跳转分析，Fast Return
- 【安全】基于CrossBit实现CrossIF构建了BufferSageTy，利用污点分析来检测缓冲区溢出
- 【BT】DistriBit，分布式的CrossBit，两层TCache设计（全局+本地），通讯协议C/S架构

2009

- 【BT】CacheBit，支持模拟Cache的行为
- 【优化】Fast Return
- 【BT】MTCrossBit
- 【优化】Profile，硬件计数，软件在block开头写SPC告诉底层硬件
- 【优化】先执行一次收集信息来优化
- 【安全】system call 的规则检测
- 【优化】Condition Codes
- 【优化】硬件实现虚拟机协处理器，进行翻译、查找等工作，软件只负责执行，消除了上下文切换
- 【BT】中间表示的设计，参考了LLVA和VCODE

2008

- 【BT】早期的MTCrossBit
- 【特定问题】支持DBT的调试器，在中间语言中新增Break指令来添加断点
- 【优化】多种热路径识别算法
- 【特定问题】支持浮点运算，添加中间语言指令
- 【优化】多种不同的Code Cache替换算法分析
- 【BT】vBTrans，在XEN中运行IA32EL，添加MMU、中断、IO的处理，支持系统态
- 【BT】Co-design CrossBit，硬件翻译、查找，执行与翻译分离（09年那篇的前身？）

2007

- 【优化】可加载的优化器，中间语言层面的优化
- 【BT】CrossBit

