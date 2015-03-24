


======================================
OpenStack Nova
======================================
Nova is the most important componet in OpenStack, which includes all information and control actions of virtual machine.

Nova API:
======================================
Nova 对外提供很多虚拟机操作和管理的API，早期的版本也包含网络，镜像，块存储的API。

Nova Flavor API:
--------------------------------------
Nova flaovr 是为虚拟机提供硬件大小的模板，并且可以设置为public和private两种属性。
这个功能主要是做模板管理，之后可以通过控制模板来控制用户所建的虚拟机大小。

通过命令：
nova flavor-list可以查看所有模板。

通过命令：
nova flaovr-access-list --flavor ID可以查看某个模板的访问权限；
nova flavor-access-list --tenant ID可以查看某个租户的可访问模板列表。

Hypervisor Api:
--------------------------------------
请求hypervisor-show的api，首先请求Hypervisor列表http://ncloud-nova-1:8774/v2/0ca7e360f8fa4f21866baad39e594c2f/os-hypervisors，
返回所有hypversor基本信息，之后再请求详细信息，返回的是所有OpenStack中的hypervisors的详细信息，包括ip，硬件信息，虚拟化信息，虚拟配置（vcpu），返回的是Hypervisor数组，之后从数组中获取hypervisor-show参数对应的hypervisor信息。
具体如下：

::

    curl -i 'http://ncloud-nova-1:8774/v2/0ca7e360f8fa4f21866baad39e594c2f/os-hypervisors/detail' -X GET -H "X-Auth-Project-Id: wtq" -H "Accept: application/json" -H "X-Auth-Token: ..."


::

    {
        "hypervisors": [{
            "service": {
                "host": "ncloud-compute-16.maas",
                "id": 27
            },
            "vcpus_used": 10,
            "hypervisor_type": "QEMU",
            "local_gb_used": 180,
            "host_ip": "192.168.250.26",
            "hypervisor_hostname": "ncloud-compute-16.maas",
            "memory_mb_used": 17920,
            "memory_mb": 128903,
            "current_workload": 0,
            "vcpus": 32,
            "cpu_info": "{vendor\": \"Intel\"...\"fpu\"], \"topology\": {\"cores\": 8, \"threads\": 2, \"sockets\": 1}}",
            "running_vms": 6,
            "free_disk_gb": 1835,
            "hypervisor_version": 2000000,
            "disk_available_least": 1722,
            "local_gb": 2015,
            "free_ram_mb": 110983,
            "id": 31
        }]
    }
