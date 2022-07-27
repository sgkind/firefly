选项 -rdynamic 用来通知链接器将所有符号添加到动态符号表中
（目的是能够通过使用 dlopen 来实现向后跟踪）


-rdynamic
Pass the flag ‘-export-dynamic’ to the ELF linker, on targets that support
it. This instructs the linker to add all symbols, not only used ones, to the
dynamic symbol table. This option is needed for some uses of dlopen or to
allow obtaining backtraces from within a program.


[What does the /ALTERNATENAME linker switch do? - The Old New Thing (microsoft.com)](https://devblogs.microsoft.com/oldnewthing/20200731-00/?p=104024)

```flowchart

```


