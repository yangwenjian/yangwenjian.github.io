

=======================================
Spring-data-jpa
=======================================

spring jpa
=======================================

OneToOne
---------------------------------------

.. code:: java

    //Husband.class
    @OneToOne
    @JoinColumn(name="wife")
    private Wife wife;
    
    //Wife.class
    @OneToOne(mappedBy="wife")
    private Husband husband

在这个例子中，对应数据库中的体现，是husband表中有一个wife的外键，这是因为我们在定义关系的时候，指定了维护关系为Husband端；

那么，在wife中查找husband，通过双向关系mappedBy查找，必须是Husband对象中的wife属性，由JPA来查找。

OneToMany和ManyToOne
---------------------------------------
 
.. code:: java
   
    //Device.class
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "device_type", insertable = false, updatable = false)
    private DeviceType deviceType;
    
    //DeviceType.class
    @OneToMany(mappedBy = "deviceType", cascade = CascadeType.PERSIST, fetch = FetchType.LAZY)
    private List<Device> deviceList;
    
在这个例子中，我们定义了关系的维护端在Device类中，在数据库中，可以看到device表中有外键进行关联，同样是通过mappedBy进行双向关系关联；

但由于是一对多的关系，DeviceType类中，拥有Device的集合；
    
这里指定获取方式为LAZY方式，指在需要的时候进行加载该属性对象，具体实现是再通过一条select语句实现，这里由于有Hibernate连接池和Hibernate多级缓存的缘故，效率并不是很慢（未亲测）；

查询
```````````````````````````````````````
由于是LAZY加载方式，一旦Hibernate的session关闭，访问该属性会报相应的异常，于是通过如下方式进行解决:

.. code:: java
    
    @Query(value="select d from DeviceType d inner join d.deviceList as device where device.department.id = :deptId group by d.id ")
    Page<DeviceType> findByDeviceType(@Param("deptId") String deptId, Pageable pageable);

更新
```````````````````````````````````````
更新时使用Device类进行save即可


ManyToMany
---------------------------------------
.. code:: java

    //User.class
    @ManyToMany(fetch = FetchType.LAZY)
    @Cascade({org.hibernate.annotations.CascadeType.SAVE_UPDATE})
    @JoinTable(name = "cbus_user_role",
            joinColumns = {@JoinColumn(name = "user_id")},
            inverseJoinColumns = {@JoinColumn(name = "role_id")})
    private Set<RoleInfo> roles;
    
    //RoleInfo.class
    @ManyToMany(mappedBy = "roles", fetch = FetchType.LAZY)
    private Set<User> users;
    
这个例子中，Hibernate通过建立关系表cbus_user_role来实现多对多的映射关系，这里我们指定一端为维护端，另一端称为inverse端，在@JoinTable中可以清晰的看到，并同时指定了关联的列名；

这里在实际使用中，经常在findone时发生stackoverflow，反复的select user和role这两张表里的内容，第一中方式是放弃双向关系，将RoleInfo里的那内容注释掉；

后来经过google才得知，正常不会出现infinite loop这种情况，但由于我使用了lombok工具包，其中复写hashCode()方法时，是使用所有属性的hashCode进行计算，这样就会一直加载其中的属性，造成栈溢出。也会调用其中的toString()方法，也会造成溢出，目前的解决方案是重写这两个方法。

查询
```````````````````````````````````````
通过查询某一个对象，Hibernate会简单进行select，如果你调用了里面的对象属性，Hibernate会提前select其中属性进行赋值。

这其中，Hibernate虽然语句多，但也会有些优化，比较明显的是，当查处的对象和将要保存的对象一致时，不会进行insert或者update语句。

更新
```````````````````````````````````````
这里还加入了级联操作关系Cascade，和我喜欢歌星一个名，好嗓子，音域宽，长得稍微可参了点；注明是SAVE_UPDATE时会将关联的关系同步存入到关系表中，如果存储的RoleInfoId不存在，则抛出JpaObjectRetrievalFailureException；

如果将Cascade改为ALL，则不仅会更新关系表，而且会将roleinfo表都对应更新，具体的实现操作如下：

.. code::

    - insert into cbus_role_info (app_id, is_delete, role_name, role_remark, id) values (?, ?, ?, ?, ?)
    - insert into cbus_role_info (app_id, is_delete, role_name, role_remark, id) values (?, ?, ?, ?, ?)
    - delete from cbus_user_role where user_id=? and role_id=?
    - delete from cbus_user_role where user_id=? and role_id=?
    - insert into cbus_user_role (user_id, role_id) values (?, ?)
    - insert into cbus_user_role (user_id, role_id) values (?, ?)
    
删除
```````````````````````````````````````
如果删除配置端，这里删除user，则关系表中的数据也会被删除；但如果删除roleinfo，则会抛出有外键，不能删除的异常。

最佳实践
=======================================

.. code::java

	@Query(value = "select device from Device device where device.deviceType.id = :deviceTypeId and device.department.id = :deptId")
    Page<Device> findByDeviceTypeAndDepartment(@Param("deviceTypeId") int deviceTypeId, @Param("deptId") String deptId, Pageable pageable);
	
	@Query(value="select d from DeviceType d inner join d.deviceList as device where device.department.id = :deptId group by d.id ")
    Page<DeviceType> findByDeviceType(@Param("deptId") String deptId, Pageable pageable);
	
	//防止lazyload造成session关闭查不出属性
	@Query(value="select device from Device device join fetch device.deviceType where device.connectType.id = :id")
	
