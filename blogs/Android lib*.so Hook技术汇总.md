# Android lib*.so Hook技术汇总

## SO HOOK技术汇总：（限于水平，难免会有错误、疏漏之处，请各位大牛斧正）
* 1. 导入表(GOT)HOOK
    最常见也最简单，为了体系的完整性，稍微再啰嗦下

* 2. ARM Inline hook
    ARM Inline 的原理想必各位坛友都知道，就不啰嗦了。 由于ARM有ARM指令和Thumb(1、2)指令集，Inline的实现比较麻烦。鉴于在论坛没有找到比较详细的帖子介绍，这里讨论了关于这两种指令集Inline hook的实现的细节部分。

* 3. 基于Android linker的"导出表"HOOK
   ELF文件格式并没有PE那种导出表，但也可以实现类似的导出表HOOK。其复杂度和导入表相当，其效果也比较理想(性价比较高^_^)

由于排版水平太烂，直接上PDF吧。
附件包含三个：
* 1. HookSOLibrary 这三种HOOK的实现库(如果发现BUG，麻烦各位坛友发邮箱告知我下吧，最好把汇编和opcode附上，ThomasKingNew@hotmail.com)
* 2. SubstrateTest  PDF中的图来源于此，限于篇幅，有些细枝末节的地方并未提出，可动态调试这个APK查看。
* 3. SO hook技术汇总.pdf 文档 

## 上传的附件
[HookSOLibrary.zip](https://github.com/shuiyes/gitblogs.github.io/blob/master/attachments/Android%20lib*.so%20Hook%E6%8A%80%E6%9C%AF%E6%B1%87%E6%80%BB/HookSOLibrary.zip?raw=true) (9.2 KB)
[SubstrateTest.zip](https://github.com/shuiyes/gitblogs.github.io/blob/master/attachments/Android%20lib*.so%20Hook%E6%8A%80%E6%9C%AF%E6%B1%87%E6%80%BB/SubstrateTest.zip?raw=true) (3.10 MB)
[SO hook技术汇总.pdf](https://github.com/shuiyes/gitblogs.github.io/blob/master/attachments/Android%20lib*.so%20Hook%E6%8A%80%E6%9C%AF%E6%B1%87%E6%80%BB/SO hook%E6%8A%80%E6%9C%AF%E6%B1%87%E6%80%BB.pdf?raw=true) (815.9 KB)

