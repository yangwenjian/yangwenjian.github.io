


=================================
VIM
=================================

启动YouCompleteMeServer命令，ycmd每次会在：

::

    /usr/bin/python2 /home/knight/.vim/bundle/YouCompleteMe/python/ycm/../../third_party/ycmd/ycmd --port=59982 --options_file=/tmp/tmpavY1mN --log=info --idle_suicide_seconds=10800 --stdout=/tmp/ycm_temp/server_59982_stdout.log --stderr=/tmp/ycm_temp/server_59982_stderr.log

不过我这次安装似乎并不成功，OpenSuse利用自动安装命令"./install.sh --clang-completer"，安装后生成的libclang.so有问题，每次启动都不成功，错误日志显示"libclang.so"版本有问题。
这个原因是因为cmake版本或者python版本等原因造成的，从官网上下载clang的最新"libclang.so"，就可以使用了。
