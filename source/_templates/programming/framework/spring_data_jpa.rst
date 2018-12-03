Spring-data-jpa

最佳实践
=======================================

.. code::java

	@Query(value = "select device from Device device where device.deviceType.id = :deviceTypeId and device.department.id = :deptId")
    Page<Device> findByDeviceTypeAndDepartment(@Param("deviceTypeId") int deviceTypeId, @Param("deptId") String deptId, Pageable pageable);
	
	@Query(value="select d from DeviceType d inner join d.deviceList as device where device.department.id = :deptId group by d.id ")
    Page<DeviceType> findByDeviceType(@Param("deptId") String deptId, Pageable pageable);
	
	//防止lazyload造成session关闭查不出属性
	@Query(value="select device from Device device join fetch device.deviceType where device.connectType.id = :id")
	