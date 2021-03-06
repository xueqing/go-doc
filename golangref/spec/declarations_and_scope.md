# 声明和范围

- [声明和范围](#声明和范围)

一个声明将一个非空的标识符和一个常量、类型、变量、函数、标签或包绑定。程序中的每个标识符都要被声明。同一个区块中的标识符不能被声明两次，且在文件和包的区块中不能声明标识符。

空白标识符可以在一个声明中像其他标识符一样使用，但是它不引入绑定因此没有被声明。在包区块，标识符 `init` 只能用于 `init` 函数的声明，而且像空白标识符一样没有引入新的绑定。

```txt
Declaration   = ConstDecl | TypeDecl | VarDecl .
TopLevelDecl  = Declaration | FunctionDecl | MethodDecl .
```

一个声明的标识符的范围是源文本的范围中，这个范围中该标识符表示指定的常量、类型、变量、函数、标签或包。

Go 使用区块定义词法作用域：

1. 预先声明的标识符的范围是全局区块(universe block)
2. 表示指定的常量、类型、变量或函数(而不是方法)的标识符，声明在顶层的(函数之外)范围是包区块
3. 一个导入包的包名的范围是包含这个导入声明的文件区块
4. 表示一个方法的接收者、函数的参数或结果变量的标识符的范围是函数体
5. 一个声明在函数内部的常量或变量标识符的范围起始于 ConstSpec 或 VarSpec(short 变量声明是 ShortVarSpec)的末尾，终止于最内部包含区块的末尾
6. 定义在函数内部的类型标识符的范围起始于 TypeSpec 的标识符，终止于最内部包含区块的末尾

一个区块声明的标识符可在一个更内部的区块重新声明。
