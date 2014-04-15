.. niusmallnan documentation master file, created by
   sphinx-quickstart on Tue Feb 18 13:49:43 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

=======================================
Nova实现虚拟机冷迁移
=======================================
冷迁移也有称块迁移，即是在非共享存储下，将虚拟机迁移到其它宿主机上，通常为了保证数据完整性，迁移之前都需要关闭虚拟机，然后拷贝instance文件到目标宿主机上，并重新加载网络策略。与之相对的是热迁移，需要工作在共享存储上，好处是迁移的时候并不需要迁移整个instance文件，速度会很快。


Nova的支持
======================
Nova迁移虚拟机是调用通过调用libvirt实现的，毕竟libvirt已经封装了针对各个虚拟化技术的统一接口，这点通过Nova代码可知::

    # Do live migration.
    try: 
        if block_migration:
            flaglist = CONF.block_migration_flag.split(',')
        else:
            flaglist = CONF.live_migration_flag.split(',')
        flagvals = [getattr(libvirt, x.strip()) for x in flaglist]
        logical_sum = reduce(lambda x, y: x | y, flagvals)

        dom = self._lookup_by_name(instance["name"])
        dom.migrateToURI(CONF.live_migration_uri % dest,
                        logical_sum,
                        None,
                        CONF.live_migration_bandwidth)
    

这里的libvirt是对底层libvirt接口封装的Python库，live_migration_uri是nova.conf的配置::
    
    
    # Migration target URI (any included "%s" is replaced with the
    # migration target hostname) (string value)
    live_migration_uri=qemu+tcp://%s/system


基本上Nova是不需要怎么配置的，真正的配置都在libvirt上。


libvirt配置
======================
主要是libvirtd.conf的配置，它通常在/etc/libvirt 下面，一般我们启用普通的TCP端口就行了，另外值得注意的是监听的ip地址一定要和该compute-node所在网络的ip一致::

    listen_tls = 0
    listen_tcp = 1
    tcp_port = "16509"
    listen_addr = "11.11.11.103"

重启libvirtd，然后你就可以迁移你的虚机了，注意使用命令行迁移的时候一定要加--block-migrate，否则Nova会默认执行热迁移。








