


=========================================
Git 
=========================================
Git是当前最火的版本管理工具，GitHub也成为程序员最喜欢的代码仓库管理网站。

Introduction
=========================================
Git是一种分布式代码管理工具，其特点是本地有一份远程仓库的clone，即使远程服务器挂了也不会丢失代码。

Git是直接记录所有文件的快照，并非比较差异进行记录。

Git有三种状态：已提交，已修改，已暂存。

常用命令：

::

    add        Add file contents to the index
    bisect     Find by binary search the change that introduced a bug
    branch     List, create, or delete branches
    checkout   Checkout a branch or paths to the working tree
    clone      Clone a repository into a new directory
    commit     Record changes to the repository
    diff       Show changes between commits, commit and working tree, etc
    fetch      Download objects and refs from another repository
    grep       Print lines matching a pattern
    init       Create an empty Git repository or reinitialize an existing one
    log        Show commit logs
    merge      Join two or more development histories together
    mv         Move or rename a file, a directory, or a symlink
    pull       Fetch from and integrate with another repository or a local branch
    push       Update remote refs along with associated objects
    rebase     Forward-port local commits to the updated upstream head
    reset      Reset current HEAD to the specified state
    rm         Remove files from the working tree and from the index
    show       Show various types of objects
    status     Show the working tree status
    tag        Create, list, delete or verify a tag object signed with GPG









Git Exercise
=========================================
Git ignore文件的写法：

::

    # 此为注释 – 将被 Git 忽略
    # 忽略所有 .a 结尾的文件
    *.a
    # 但 lib.a 除外
    !lib.a
    # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
    /TODO
    # 忽略 build/ 目录下的所有文件
    build/
    # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
    doc/*.txt
    # ignore all .txt files in the doc/ directory
    doc/**/*.txt
