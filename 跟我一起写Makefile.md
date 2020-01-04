参照[链接](https://seisman.github.io/how-to-write-makefile/recipes.html)。主要记录一些不常见和主要的用法。

## 嵌套执行make

在一些大的工程中，我们会把我们不同模块或是不同功能的源文件放在不同的目录中，我们可以在每个目录 中都书写一个该目录的Makefile，这有利于让我们的Makefile变得更加地简洁，而不至于把所有的东西全 部写在一个Makefile中，这样会很难维护我们的Makefile，这个技术对于我们模块编译和分段编译有着非 常大的好处。

例如，我们有一个子目录叫subdir，这个目录下有个Makefile文件，来指明了这个目录下文件的编译规则 。那么我们总控的Makefile可以这样书写：

```makefile
subsystem:
    cd subdir && $(MAKE)
subsystem:
    $(MAKE) -C subdir
```

定义$(MAKE)宏变量的意思是，也许我们的make需要一些参数，所以定义成一个变量比较利于维护。这两个 例子的意思都是先进入“subdir”目录，然后执行make命令。

我们把这个Makefile叫做“总控Makefile”，总控Makefile的变量可以传递到下级的Makefile中（如果 你显示的声明），但是不会覆盖下层的Makefile中所定义的变量，除非指定了 -e 参数。