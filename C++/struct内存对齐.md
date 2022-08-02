# struct内存对齐
### 为什么要有内存对齐
cpu在内存中读取数据时，只能从固定倍数的位开始读取，不能跨内存区间访问。如果不按照规则对齐，就可能出现一个数据跨越两个内存区间，需要进行两次io。  
所以为了提高内存的访问效率，引入了内存对齐。
### 内存对齐规则
[/Zp (Struct Member Alignment)](https://docs.microsoft.com/en-us/cpp/build/reference/zp-struct-member-alignment?f1url=%3FappId%3DDev12IDEF1&l=ZH-CN&k=k(VC.Project.VCCLCompilerTool.StructMemberAlignment)&rd=true&view=msvc-160)
>The compiler stores members after the first one on a boundary that's the smaller of either the size of the member type, or an N-byte boundary.  
编译器将第一个成员之后的成员存储在成员类型大小或 N 字节边界中较小的一个边界上。

也就是说，对于结构体的第一个成员之后的成员，编译器会把它放在以下两个位置（offset 距离结构体头部的偏移距离）的最小值：
1. offset 为对齐到这个成员的 size 的整数倍。
2. offset 为对齐到 N-byte 的整数倍（N 为指定的对齐值）。

注意：不同的系统架构 N 不同，N 也可以通过 pragma pack 来手动设置（不推荐）。