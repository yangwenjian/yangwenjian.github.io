


==============================
OSLO
==============================
在Oslo项目出现之前，Openstack的每个组件都要自己实现处理配置文件的代码，这样就导致了一系列重复代码的产生，这是程序员所不能忍受的，因此Oslo就解决这类问题。

Oslo需要解决的几点问题：

1) command line option parsing;
2) common command line options;
3) configuration file parsing;
4) option value lookup.

Oslo.conf在使用时，需要先声明配置项的名称、定义类型、帮助文字、缺省值等，然后再按照事先声明的配置项，对CLI或conf中的内容进行解析。

::

    common_opts = [
        cfg.StrOpt('bind_host',
            default='0.0.0.0',
            help='IP address to listen on'),
        cfg.IntOpt('bind_port',
            default=9292,
            help='Port number to listen on')
    ]
