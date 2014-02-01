---
layout: post
category : OpenStack
tagline: "nova instance create"
tags : [OpenStack]
---
{% include JB/setup %}

##Nova创建实例代码分析

####版本 ：Havana
####说明 ：
            --> 为调用下一流程
            Appendix 为变量的内容
            
###主要API解析
run_instances(self, context, kwargs): 

    kwargs{'max_count' : int
           'min_count' : int
           'kernel_id' : str
           'ramdisk_id': str
           'key_name'  : str
           'user_data' : str
           'security_group' : str
           'placement'      : {}
           'block_device_mapping' :{}
            }

    1. 初始化参数:
    _resv_id_from_token(context, client_token)
    _format_run_instances(context, resv_id)
    _get_image(context, kwargs['ramdisk_id'])
    parse_block_device_mapping(bdm)
    __get_image_state(image)
    flavor_obj.Flavor.get_by_name(context,kwargs.get('instance_type',None))

    2. 传递参数，调用关键函数 
    from nova.compute import api as compute_api
    compute_api.create(context,
            instance_type=obj_base.obj_to_primitive(flavor),
            image_href=image_uuid,
            max_count=int(kwargs.get('max_count', min_count)),
            min_count=min_count,
            kernel_id=kwargs.get('kernel_id'),
            ramdisk_id=kwargs.get('ramdisk_id'),
            key_name=kwargs.get('key_name'),
            user_data=kwargs.get('user_data'),
            security_group=kwargs.get('security_group'),
            availability_zone=kwargs.get('placement', {}).get('availability_zone'),
            block_device_mapping=kwargs.get('block_device_mapping', {}))


###1._get_image(self, context, ec2_id):
   
    Function 
        根据ec2_id获取指定参数的实例的元数据

    Workflow:
        internal_id = ec2utils.ec2_id_to_id(ec2_id)
        image_service.show(context, internal_id) :关键函数
        image_type = ec2_id.split('-')[0]

    return image

####1.1 image_service.show(context, internal_id)
    1）image_uuid :根据给定的image_id值，查询数据库，找到匹配的表信息，获取它的S3Image.uuid并返回，赋值给image_uuid 
    2）直接得到service 
       或 
       尝试一定次数调用glance客户端的get方法，获取image_id指定的镜像元数据（此时为JSON格式）,把glance下载的镜像数据转换为python可处理的字典格式； 
    3）根据元数据中的glance id值查找到匹配的S3格式镜像数据信息，进一步获取的S3数据库id值赋值给image_copy['id']、image_copy['properties']['kernel_id']和image_copy['properties']['ramdisk_id']； 更新image当中的相关属性，返回更新后的image数据；

根据配置参数确定最大链接访问glance客户端的次数，在不超过最大次数的情况下，尝试确定正确的客户端版本，建立glance客户端类的对象，通过glance客户端中images.py中的get方法，获取指定的镜像元数据；

###2. _parse_block_device_mapping()
  
    解析块设备映射bdm；
    这个方法主要完成的就是读取数据库，获取相应数据更新上述实例数据中的'snapshot_id'和'volume_id'值；
       ( {'device_name': '/dev/fake0',
          'ebs': {'snapshot_id': 'snap-12345678',
                   'volume_size': 1}},
          {'device_name': '/dev/fake0',
           'snapshot_id': '00000000-1111-2222-3333-444444444444',
           'volume_size': 1,
           'delete_on_termination': True})


###3. compute_api.create()
        1. 根据给定的参数是否满足policy来决定是否有create的资格
        2. neutron启动多个实例时，处理端口的分配问题。
        3. 准备实例，并且发送实例的信息和要运行实例的请求消息到远程调度器scheduler;
        实现实例的简历和运行，由调度器决定，这部分代码实际上只是实现请求消息的发送；  
        返回一个元组（实例或者是reservation_id的元组），元组里面的实例可以是“None”或者是实例字典的一个列表，这要取决于是否等待scheduler返回的信息；  
     

    

####3.1 _create_instance()
    1. 初始化传入参数
    2. 验证传入参数的有效性,返回要建立实例的各类信息  
    3. 
        Verify all the input parameters regardless of the provisioning
        strategy being performed and schedule the instance(s) for
        creation.

    1.  (base_options,max_net_count) = validate_and_build_base_options()
    2. _check_and_transform_bdm(self, base_options, image_meta, min_count,
                                 max_count, block_device_mapping, legacy_bdm):
    3. _provision_instances(self, context, instance_type, min_count,
            max_count, base_options, boot_meta, security_groups,block_device_mapping):
    4. _build_filter_properties(self, context, scheduler_hints, forced_host,forced_node, instance_type):
    5. _record_action_start(self, context, instance, action):
		获取每个实例启动的一些信息，并在数据库中建立新的条目，把这些信息写入数据库。
    6. compute_task_api.build_instances()
        ComputeTaskAPI(*args, **kwargs)

    return (instances, reservation_id)
	
#####3.1.1 compute_task_api.build_instances()
      实际调用:
        if 
            nova.compute.api 的 API中指定了compute_task_api
        else 
             --> self._compute_task_api = nova.conductor.ComputeTaskAPI()
             if oslo.config.cfg.CONF.conductor.use_local
                 //nova.conductor.api
                 -->
                 nova.conductor.api.LocalComputeTaskAPI().build_instances(ARGS1)
                 --> nova.utils.spawn_n(
                                    utils.ExceptionHelper(mananger.ComputeTaskManager()).build_instances(), 
                                    ARGS)
                 -->
                     eventlet.spawn_n(utils.ExceptionHelper(mananger.ComputeTaskManager().build_instances(), ARGS))
                 -->
                     nova.utils.ExceptionHelper(mananger.ComputeTaskManager().build_instances(), ARGS)
                 --> nova.conductor.mananger.ComputeTaskManager().build_instances(ARGS)
                 --> nova.scheduler.rpcapi.run_instances(ARGS)
                 -->
                     cctxt = nova.rpcclient.RpcProxy().get_client()
                     nova.rpcclient.prepare().cast(cctxt, 'run_instance', ARGS)

                 --> 
                     nova.openstack.common.rpc.proxy.cast_to_server()  OR
                     nova.openstack.common.rpc.proxy.fanout_cast_to_server() OR
                     nova.openstack.common.rpc.proxy.fanout_cast() : Broadcast a remote method invocation with no return
                 -->
                     importutils.import_module(oslo.config.cfg.CONF.rpc_backend).fanout_cast_to_server()
                     importutils.import_module(oslo.config.cfg.CONF.rpc_backend).cast_to_server()
                     importutils.import_module(oslo.config.cfg.CONF.rpc_backend).fanout_cast()
                 --> 
                     nova.openstack.common.rpc.impl_kombu.fanout_cast_to_server()
                     nova.openstack.common.rpc.impl_kombu.cast_to_server()
                     nova.openstack.common.rpc.impl_kombu.fanout_cast()
                 -->
                     nova.openstack.common.rpc.amqp.cast_to_server(conf, context,
                     server_params, topic, msg,
                     rpc_amqp.get_connection_pool(conf, Connection))

                     nova.openstack.common.rpc.amqp.fanout_cast(conf, context,
                     server_params, topic, msg,
                     rpc_amqp.get_connection_pool(conf, Connection))

                     nova.openstack.common.rpc.amqp.fanout_cast_to_server(conf,
                     context, server_params, topic, msg,
                     rpc_amqp.get_connection_pool(conf, Connection))

                 --> 
                     """Sends a message on a fanout exchange without waiting for a response."""
                     nova.openstack.common.rpc.amqp.fanout_cast((conf,
                     context, server_params, topic, msg,
                     rpc_amqp(conf, kombu.Connection.BrokerConnection()))

                     """Sends a message on a fanout exchange to a specific server."""
                     nova.openstack.common.rpc.amqp.fanout_cast_to_server((conf,
                     context, server_params, topic, msg,
                     rpc_amqp(conf, kombu.Connection.BrokerConnection()))

                    """Sends a message on a topic to a specific server."""
                     nova.openstack.common.rpc.amqp.cast_to_server((conf,
                     context, server_params, topic, msg,
                     rpc_amqp(conf, kombu.Connection.BrokerConnection()))

                 -->
                 nova.openstack.common.rpc.amqp.ConnectionContext.fanout_send()
                 nova.openstack.common.rpc.amqp.ConnectionContext.topic_send()

                 -->
                 nova.openstack.common.rpc.common.Connection.fanout
                 nova.openstack.common.rpc.amqp.get_connection_pool(conf, kombu.Connection.BrokerConnection())

                 --> kombu.Connection.BrokerConnection.pool

         else 
             --> nova.conductor.api.ComputeTaskAPI().build_instances(ARGS)
             --> nova.rpcapi.ComputeTaskAPI().build_instances(ARGS)




##整个创建实例的主线代码调用

    nova.api.ec2.cloud.run_instances(context, kwargs)
    --> 
        nova.compute.api.create(ARGS)
    --> 
        nova.compute.api._create_instance(ARGS)

        -->     
            nova.compute.api.compute_task_api(self)
        -->
            nova.conductor.ComputeTaskAPI()
        --> 
            nova.conductor.api.ComputeTaskAPI(args, kwargs)     
            OR
            nova.conductor.api.LocalComputeTaskAPI(args, kwargs)
    -->
        nova.conductor.api.ComputeTaskAPI.build_instances(ARGS1)
        OR
        nova.conductor.api.LocalComputeTaskAPI.build_instances(ARGS1)


    1. nova.conductor.api.ComputeTaskAPI.build_instances(ARGS1)
     -->
        nova.conductor.rpcapi.ComputeTaskAPI.build_instances(ARGS1)
        -->
            cctxt = nova.rpcclient.RpcProxy().get_client(namespace='compute_task').prepare()
            cctxt = nova.rpcclient.RPCClient('compute_task',None).prepare(version=1.5)
     -->
        nova.rpcclient.RPCClient.prepare.cast(context, 'build_instance', ARGS.1-context)
     -->
        nova.openstack.common.rpc.proxy.fanout_cast_to_server(context,
                                            'build_instances', msg, kwargs)
    OR
        nova.openstack.common.rpc.proxy.cast_to_server(context, msg, 
                                            'build_instances', kwargs)
    OR
        nova.openstack.common.rpc.proxy.fanout_cast(context, msg, 
                                            'build_instances', kwargs)

     -->
         importutils.import_module(oslo.config.cfg.CONF.rpc_backend).fanout_cast_to_server()
    OR
         importutils.import_module(oslo.config.cfg.CONF.rpc_backend).cast_to_server()
    OR
         importutils.import_module(oslo.config.cfg.CONF.rpc_backend).fanout_cast()

     -->
        """Sends a message on a topic to a specific server."""
         nova.openstack.common.rpc.amqp.cast_to_server(CONF, context,'build_instances', topic, msg,
                                                        rpc_amqp.get_connection_pool(conf, Connection))
    OR
         """Sends a message on a fanout exchange to a specific server."""
         nova.openstack.common.rpc.amqp.fanout_cast_to_server(conf, context, 'build_instances', topic, msg,
                                                        rpc_amqp.get_connection_pool(conf, Connection))
    OR
         """Sends a message on a fanout exchange without waiting for a response."""
         nova.openstack.common.rpc.amqp.fanout_cast(conf, context,topic, msg,
                                                        rpc_amqp.get_connection_pool(conf, Connection))

     -->
        POOL = rpc_amqp.get_connection_pool(conf, Connection)
        nova.openstack.common.rpc.amqp.ConnectionContext(CONF,POOL).fanout_send('build_instances', msg)
    OR
        nova.openstack.common.rpc.amqp.ConnectionContext(CONF,POOL).topic_send('build_instances', msg)

     --> 
        Connection = nova.openstack.common.rpc.amqp.get_connection_pool(conf,Connection).get()
        Connection.fanout_cast('build_instances')
        Connection.topic_send('build_instances')

        Connection =
        nova.openstack.common.rpc.amqp.get_connection_pool(conf,Connection).connection_cls(CONF,
                'build_instances')
        Connection.topic_send('build_instances')
        Connection.fanout_cast('build_instances')

     -->
        nova.openstack.common.rpc.impl_kombu.Connection.fanout_send('build_instances', msg)
        nova.openstack.common.rpc.impl_kombu.Connection.topic_send('build_instances', msg)

     -->
        nova.openstack.common.rpc.impl_kombu.Connection.publisher_send(TopicPublisher,'build_instances', msg)
        nova.openstack.common.rpc.impl_kombu.Connection.topic_send(FanoutPublisher,'build_instances', msg)

    2. nova.conductor.api.LocalComputeTaskAPI().build_instances(ARGS1)
     --> nova.utils.spawn_n(
                        utils.ExceptionHelper(mananger.ComputeTaskManager()).build_instances(), 
                        ARGS1)
     -->
         eventlet.spawn_n(utils.ExceptionHelper(mananger.ComputeTaskManager().build_instances(), ARGS))
     -->
         nova.utils.ExceptionHelper(mananger.ComputeTaskManager().build_instances(), ARGS)
     --> nova.conductor.mananger.ComputeTaskManager().build_instances(ARGS)
     --> nova.scheduler.rpcapi.run_instances(ARGS)
         -->
            cctxt = nova.rpcclient.RpcProxy().get_client()
            cctxt = nova.rpcclient.RPCClient()
     -->
        nova.rpcclient.RPCClient.prepare().cast(context,'run_instance', ARGS2)

     --> 
         nova.openstack.common.rpc.proxy.cast_to_server(context, 'run_instances', 
                                                                msg,{})  
    OR
         nova.openstack.common.rpc.proxy.fanout_cast_to_server(context,'run_instances',
                                                                msg,{}) 
    OR
         nova.openstack.common.rpc.proxy.fanout_cast(context, msg, {}) 
     -->
         importutils.import_module(oslo.config.cfg.CONF.rpc_backend).fanout_cast_to_server(CONF,context,'build_instances',topic,msg)
    OR
         importutils.import_module(oslo.config.cfg.CONF.rpc_backend).cast_to_server(CONF,context,'build_instances', topic,msg)
    OR
         importutils.import_module(oslo.config.cfg.CONF.rpc_backend).fanout_cast(CONF,context,topic,msg)
     --> 
         nova.openstack.common.rpc.impl_kombu.fanout_cast_to_server(CONF,context,'build_instances',topic,msg)
    OR
         nova.openstack.common.rpc.impl_kombu.cast_to_server(CONF,context,'build_instances',topic,msg)
    OR
         nova.openstack.common.rpc.impl_kombu.fanout_cast(CONF,context,topic,msg)
     -->
        """Sends a message on a topic to a specific server."""
         nova.openstack.common.rpc.amqp.cast_to_server(CONF, context,'build_instances', topic, msg,
                                                        rpc_amqp.get_connection_pool(conf, Connection))
    OR
         """Sends a message on a fanout exchange to a specific server."""
         nova.openstack.common.rpc.amqp.fanout_cast_to_server(conf, context, 'build_instances', topic, msg,
                                                        rpc_amqp.get_connection_pool(conf, Connection))
    OR
         """Sends a message on a fanout exchange without waiting for a response."""
         nova.openstack.common.rpc.amqp.fanout_cast(conf, context,topic, msg,
                                                        rpc_amqp.get_connection_pool(conf, Connection))


     --> 
         """Sends a message on a fanout exchange without waiting for a response."""
         nova.openstack.common.rpc.amqp.fanout_cast((conf,
         context, server_params, topic, msg,
         rpc_amqp(conf, kombu.Connection.BrokerConnection()))

         """Sends a message on a fanout exchange to a specific server."""
         nova.openstack.common.rpc.amqp.fanout_cast_to_server((conf,
         context, server_params, topic, msg,
         rpc_amqp(conf, kombu.Connection.BrokerConnection()))

        """Sends a message on a topic to a specific server."""
         nova.openstack.common.rpc.amqp.cast_to_server((conf,
         context, server_params, topic, msg,
         rpc_amqp(conf, kombu.Connection.BrokerConnection()))

     -->
        nova.openstack.common.rpc.amqp.ConnectionContext.fanout_send()
        nova.openstack.common.rpc.amqp.ConnectionContext.topic_send()

     -->
     nova.openstack.common.rpc.common.Connection.fanout
     nova.openstack.common.rpc.amqp.get_connection_pool(conf, kombu.Connection.BrokerConnection())

     --> kombu.Connection.BrokerConnection.pool

##Appendix
###ARGS:
    context = <nova.context.RequestContext object at 0x447d050>  
			{
			user_id = afc380206e2549ad930396d9050d20cf  
			project_id = 0e492e86f22e4d19bd523f1e7ca64566  
			roles = [u'admin', u'KeystoneAdmin', u'KeystoneServiceAdmin']  
			read_deleted = no  
			remote_address = 172.21.6.145  
			timestamp = 2013-06-23 16:36:37.399405  
			request_id = req-f0255b14-833d-4fff-b973-23c35f70ddda  
			auth_token = 7e7bb3cf84ab43269010bb55410064b3  
			service_catalog = [{u'endpoints': [
								{u'adminURL': u'http://172.21.5.161:8776/v1/0e492e86f22e4d19bd523f1e7ca64566', 
								u'region': u'RegionOne', 
								u'id': u'753a1ad55e91469794e2eb7ac4c3df92', 
								u'internalURL': u'http://172.21.5.161:8776/v1/0e492e86f22e4d19bd523f1e7ca64566', 
								u'publicURL': u'http://172.21.6.145:8776/v1/0e492e86f22e4d19bd523f1e7ca64566'}], 
								u'endpoints_links': [], 
								u'type': u'volume',
								u'name': u'cinder'}]  
			instance_lock_checked = False  
			quota_class = None  
			user_name = admin  
			project_name = admin  
			is_admin = True  
			}
    instance_type = {
			'memory_mb': 2048L, 'root_gb': 20L, 'deleted_at': None, 'name': u'm1.small', 'deleted': 0L, 
			'created_at': None, 'ephemeral_gb': 0L, 'updated_at': None, 'disabled': False, 'vcpus': 1L, 
			'extra_specs': {}, 'swap': 0L, 'rxtx_factor': 1.0, 'is_public': True, 'flavorid': u'2', 
			'vcpu_weight': None, 'id': 5L}  
    image_href = 20612b24-c980-4900-b270-8e6b66e5f72f  
    kernel_id = None  
    ramdisk_id = None  
    min_count = 1  
    max_count = 1  
    display_name = test3  
    display_description = test3  
    key_name = oskey  
    key_data = None  
    security_group = ['default']  
    availability_zone = None  
    user_data = None  
    metadata = {}  
    injected_files = []  
    admin_password = Piu4aSSNmSNk  
    block_device_mapping = []  
    access_ip_v4 = None  
    access_ip_v6 = None  
    requested_networks = None  
    config_drive = None  
    auto_disk_config = None  
    scheduler_hints = {}  
    legacy_bdm = True
###ARGS.1
        (context,
        instances=instances, 
        image=boot_meta,
        filter_properties=filter_properties,
        admin_password=admin_password,
        injected_files=injected_files,
        requested_networks=requested_networks,
        security_groups=security_groups,
        block_device_mapping=block_device_mapping,
        legacy_bdm=False)

###ARGS.2
        (context,
         instances = [nova.compute.api._apply_instance_name_template(),
            )

###ARGS2
    msg_kwargs = {'request_spec': request_spec,
                  'admin_password': admin_password,
                  'injected_files': injected_files,
                  'requested_networks': requested_networks,
                  'is_first_time': is_first_time,
                  'filter_properties': filter_properties}


###msg
    (
        'method'  : nova.rpcclient.RPCClient.prepare()
        'namespce': 'run_instances' OR 'build_instances'
        'args'    : ARGS1
        'version' : str
        'oslo.version' : 2.0
        'oslo.message' : msg of json 
        )
        
