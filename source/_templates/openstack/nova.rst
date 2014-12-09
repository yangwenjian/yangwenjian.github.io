


======================================
OpenStack Nova
======================================

Nova API:
======================================

Hypervisor Api:
--------------------------------------
请求hypervisor-show的api，首先请求Hypervisor列表http://ncloud-nova-1:8774/v2/0ca7e360f8fa4f21866baad39e594c2f/os-hypervisors，
返回所有hypversor基本信息，之后再请求详细信息，返回的是所有OpenStack中的hypervisors的详细信息，包括ip，硬件信息，虚拟化信息，虚拟配置（vcpu），返回的是Hypervisor数组，之后从数组中获取hypervisor-show参数对应的hypervisor信息。
具体如下：

::

    curl -i 'http://ncloud-nova-1:8774/v2/0ca7e360f8fa4f21866baad39e594c2f/os-hypervisors/detail' -X GET -H "X-Auth-Project-Id: wtq" -H "Accept: application/json" -H "X-Auth-Token: ..."

返回的是所有OpenStack中的hypervisors的详细信息，包括ip，硬件信息，虚拟化信息，虚拟配置（vcpu），返回的是Hypervisor数组，然后具体如下：

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
