![[Pasted image 20250731193828.png]]makeBaseApp.pl -t ioc basic 
makeBaseApp.pl -i -t ioc basic
- **makeBaseApp.pl**：EPICS 自带的 Perl 脚本，用于生成应用开发的基本目录结构和模板文件。
- **-t ioc**：指定要创建的是 “ioc”（Input Output Controller）类型的应用模板。
- **basic**：你要创建的应用名，最后会生成一个叫 `basic` 的目录（或相关文件）。这条命令会**创建一个名为 `basic` 的 IOC 应用的基础框架**，生成 Makefile、src 目录、以及一些模板文件，方便你后续开发。
- - **-i**：表示_只生成应用实例（instance）相关的目录和文件_。  
    具体来说，它会在你的当前目录下，基于之前创建的 “basic” 应用，再生成一层实例化目录和文件。  
    你可以把它理解为“为 basic 这个 IOC 应用创建一个新实例的配置环境”。
- 其它部分和上面一样。

#### 总结一句话：

> 这条命令**只会为 `basic` 应用生成应用实例相关的目录结构和配置文件**（比如 IOC 配置、启动脚本等），而不会再生成应用代码的框架（那些文件已经由第一条命令生成了）。
![[Pasted image 20250731200842.png]]- **make**  
    是 Linux 下的自动化构建工具，会读取当前目录下的 `Makefile` 文件，按里面的规则执行任务。
    
- **clean**  
    是 `Makefile` 里的一个目标，意思是**删除所有由编译过程生成的临时文件、目标文件（.o）、可执行文件等**，让目录恢复到“未编译”状态。
    
- **uninstall**  
    也是 `Makefile` 里的目标，意思是**把之前通过 `make install` 安装到系统或指定目录的程序或库全部删除**。
make clean
make uninstall
