---
layout: post
category : OpenStack
tagline: "OS_EXT_SRV_ATTR:instance_name无法获取"
comments : false
tags : [OpenStack_E, solution]
---

{% include JB/setup %}
###版本 Openstack Essex
###问题：
    由于dashboard下的某一点击需要获取OS-EXT-SRV-ATTR:instance_name，但是在demo用户下无法获取。由于之前记得在admin下是好的。通过novaclient的调试模式来确认问题。

###问题再现
root@ubuntu1204:~# nova list

	+--------------------------------------+------+--------+-------------------+
	|                  ID                  | Name | Status |      Networks     |
	+--------------------------------------+------+--------+-------------------+
	| d6887f8f-72c3-4f1c-a946-4e9990df7c57 | test | ACTIVE |  private=10.0.0.2 |
	| 296a17c9-139d-48f6-9c77-6ddf3d33ff5d | tets | ACTIVE | private=10.0.0.11 |
	+--------------------------------------+------+--------+-------------------+
####admin 下得到的结果
root@ubuntu1204:~# nova show 296a17c9-139d-48f6-9c77-6ddf3d33ff5d

    +-------------------------------------+------------------------------------------------------------+
    |               Property              |                           Value                            |
    +-------------------------------------+------------------------------------------------------------+
    |          OS-DCF:diskConfig          |                           MANUAL                           |
    |         OS-EXT-SRV-ATTR:host        |                          aslcloud                          |
    | OS-EXT-SRV-ATTR:hypervisor_hostname |                            None                            |
    |    OS-EXT-SRV-ATTR:instance_name    |                     instance-0000000a                      |
    |        OS-EXT-STS:power_state       |                             1                              |
    |        OS-EXT-STS:task_state        |                            None                            |
    |         OS-EXT-STS:vm_state         |                           active                           |
    |              accessIPv4             |                                                            |
    |              accessIPv6             |                                                            |
    |             config_drive            |                                                            |
    |               created               |                    2014-01-21T18:51:49Z                    |
    |                flavor               |                        m1.tiny (1)                         |
    |                hostId               |  d7b7a0d031ddae12f1f7d611a20c4efb217c901e6b93c388b76307c3  |
    |                  id                 |            296a17c9-139d-48f6-9c77-6ddf3d33ff5d            |
    |                image                | cirros-0.3.0-x86_64 (2d5d60f1-d96f-4c5d-bc97-18adafcf90d1) |
    |               key_name              |                                                            |
    |               metadata              |                             {}                             |
    |                 name                |                            tets                            |
    |           private network           |                         10.0.0.11                          |
    |               progress              |                             0                              |
    |                status               |                           ACTIVE                           |
    |              tenant_id              |              aedbb3cbce1f4cc98d5630a96996cba2              |
    |               updated               |                    2014-01-21T18:52:04Z                    |
    |               user_id               |              9d37d50739424ef783a7546dd931ab83              |
    +-------------------------------------+------------------------------------------------------------+

####demo 下得到的结果
     +-------------------------------------+------------------------------------------------------------+
        |               Property              |                           Value                            |
        +-------------------------------------+------------------------------------------------------------+
        |          OS-DCF:diskConfig          |                           MANUAL                           |
        | OS-EXT-SRV-ATTR:hypervisor_hostname |                            None                            |
        |        OS-EXT-STS:power_state       |                             1                              |
        |        OS-EXT-STS:task_state        |                            None                            |
        |         OS-EXT-STS:vm_state         |                           active                           |
        |              accessIPv4             |                                                            |
        |              accessIPv6             |                                                            |
        |             config_drive            |                                                            |
        |               created               |                    2014-01-21T18:51:49Z                    |
        |                flavor               |                        m1.tiny (1)                         |
        |                hostId               |  d7b7a0d031ddae12f1f7d611a20c4efb217c901e6b93c388b76307c3  |
        |                  id                 |            296a17c9-139d-48f6-9c77-6ddf3d33ff5d            |
        |                image                | cirros-0.3.0-x86_64 (2d5d60f1-d96f-4c5d-bc97-18adafcf90d1) |
        |               key_name              |                                                            |
        |               metadata              |                             {}                             |
        |                 name                |                            tets                            |
        |           private network           |                         10.0.0.11                          |
        |               progress              |                             0                              |
        |                status               |                           ACTIVE                           |
        |              tenant_id              |              aedbb3cbce1f4cc98d5630a96996cba2              |
        |               updated               |                    2014-01-21T18:52:04Z                    |
        |               user_id               |              9d37d50739424ef783a7546dd931ab83              |
        +-------------------------------------+------------------------------------------------------------+

###跟踪及解决办法
 
####命令行调试模式 admin下调试信息
nova --debug show 296a17c9-139d-48f6-9c77-6ddf3d33ff5d 

	send: u'GET /v1.1/aedbb3cbce1f4cc98d5630a96996cba2/servers/296a17c9-139d-48f6-9c77-6ddf3d33ff5d HTTP/1.1\r\nHost: 172.30.51.32:8774\r\nx-auth-project-id: admin\r\nx-auth-token: 6f0e0b1ef03d4c19a50aefba15f1dc3f\r\naccept-encoding: gzip, deflate\r\naccept: application/json\r\nuser-agent: python-novaclient\r\n\r\n'
	reply: 'HTTP/1.1 200 OK\r\n'
	header: X-Compute-Request-Id: req-728bff03-95cc-43c5-ae26-354d4511c216
	header: Content-Type: application/json
	header: Content-Length: 1375
	header: Date: Tue, 21 Jan 2014 23:46:47 GMT

	REQ: curl -i http://172.30.51.32:8774/v1.1/aedbb3cbce1f4cc98d5630a96996cba2/servers/296a17c9-139d-48f6-9c77-6ddf3d33ff5d -X GET -H "X-Auth-Project-Id: admin" -H "User-Agent: python-novaclient" -H "Accept: application/json" -H "X-Auth-Token: 6f0e0b1ef03d4c19a50aefba15f1dc3f"

	RESP:{'status': '200', 'content-length': '1375', 'content-location': u'http://172.30.51.32:8774/v1.1/aedbb3cbce1f4cc98d5630a96996cba2/servers/296a17c9-139d-48f6-9c77-6ddf3d33ff5d', 'x-compute-request-id': 'req-728bff03-95cc-43c5-ae26-354d4511c216', 'date': 'Tue, 21 Jan 2014 23:46:47 GMT', 'content-type': 'application/json'} {"server": {"OS-EXT-STS:task_state": null, "addresses": {"private": [{"version": 4, "addr": "10.0.0.11"}]}, "links": [{"href": "http://172.30.51.32:8774/v1.1/aedbb3cbce1f4cc98d5630a96996cba2/servers/296a17c9-139d-48f6-9c77-6ddf3d33ff5d", "rel": "self"}, {"href": "http://172.30.51.32:8774/aedbb3cbce1f4cc98d5630a96996cba2/servers/296a17c9-139d-48f6-9c77-6ddf3d33ff5d", "rel": "bookmark"}], "image": {"id": "2d5d60f1-d96f-4c5d-bc97-18adafcf90d1", "links": [{"href": "http://172.30.51.32:8774/aedbb3cbce1f4cc98d5630a96996cba2/images/2d5d60f1-d96f-4c5d-bc97-18adafcf90d1", "rel": "bookmark"}]}, "OS-EXT-STS:vm_state": "active", **"OS-EXT-SRV-ATTR:instance_name": "instance-0000000a"**, "flavor": {"id": "1", "links": [{"href": "http://172.30.51.32:8774/aedbb3cbce1f4cc98d5630a96996cba2/flavors/1", "rel": "bookmark"}]}, "id": "296a17c9-139d-48f6-9c77-6ddf3d33ff5d", "user_id": "9d37d50739424ef783a7546dd931ab83", "OS-DCF:diskConfig": "MANUAL", "accessIPv4": "", "accessIPv6": "", "progress": 0, "OS-EXT-STS:power_state": 1, "config_drive": "", "status": "ACTIVE", "updated": "2014-01-21T18:52:04Z", "hostId": "d7b7a0d031ddae12f1f7d611a20c4efb217c901e6b93c388b76307c3", "OS-EXT-SRV-ATTR:host": "aslcloud", "key_name": "", "OS-EXT-SRV-ATTR:hypervisor_hostname": null, "name": "tets", "created": "2014-01-21T18:51:49Z", "tenant_id": "aedbb3cbce1f4cc98d5630a96996cba2", "metadata": {}}}

这里只显示关键请求与应答，详细见附录。显然这次请求中得到了 OS-EXT-SRV-ATTR:instance_name": "instance-0000000a。

dashboard 下经过调试几次跟踪代码，从horizon下定位代码到 novaclient/v1_1/servers.py. 用pdb调试后得到的如下信息。

    http://172.30.51.32:8774/v1.1/aedbb3cbce1f4cc98d5630a96996cba2/servers/296a17c9-139d-48f6-9c77-6ddf3d33ff5d  GET
    {'headers': {'X-Auth-Project-Id': u'aedbb3cbce1f4cc98d5630a96996cba2', 'User-Agent': 'python-novaclient', 'Accept': 'application/json', 'X-Auth-Token': u'a54366edcbf74df291b6bfbf39d9656c'}}
    
    {u'server': {u'OS-EXT-STS:task_state': None, u'addresses': {u'private': [{u'version': 4, u'addr': u'10.0.0.11'}]}, u'links': [{u'href': u'http://172.30.51.32:8774/v1.1/aedbb3cbce1f4cc98d5630a96996cba2/servers/296a17c9-139d-48f6-9c77-6ddf3d33ff5d', u'rel': u'self'}, {u'href': u'http://172.30.51.32:8774/aedbb3cbce1f4cc98d5630a96996cba2/servers/296a17c9-139d-48f6-9c77-6ddf3d33ff5d', u'rel': u'bookmark'}], u'image': {u'id': u'2d5d60f1-d96f-4c5d-bc97-18adafcf90d1', u'links': [{u'href': u'http://172.30.51.32:8774/aedbb3cbce1f4cc98d5630a96996cba2/images/2d5d60f1-d96f-4c5d-bc97-18adafcf90d1', u'rel': u'bookmark'}]}, u'OS-EXT-STS:vm_state': u'active', u'flavor': {u'id': u'1', u'links': [{u'href': u'http://172.30.51.32:8774/aedbb3cbce1f4cc98d5630a96996cba2/flavors/1', u'rel': u'bookmark'}]}, u'id': u'296a17c9-139d-48f6-9c77-6ddf3d33ff5d', u'user_id': u'9d37d50739424ef783a7546dd931ab83', u'OS-DCF:diskConfig': u'MANUAL', u'accessIPv4': u'', u'accessIPv6': u'', u'progress': 0, u'OS-EXT-STS:power_state': 1, u'metadata': {}, u'status': u'ACTIVE', u'updated': u'2014-01-21T18:52:04Z', u'hostId': u'd7b7a0d031ddae12f1f7d611a20c4efb217c901e6b93c388b76307c3', u'key_name': u'', u'name': u'zhao', u'created': u'2014-01-21T18:51:49Z', u'tenant_id': u'aedbb3cbce1f4cc98d5630a96996cba2', u'config_drive': u''}}

显然，没有OS-EXT-SRV-ATTR:instance_name字段。

经过对比两次请求，发现 X-Auth-Project-Id 和 X-Auth-Token 不一样，考虑不同的用户 token 一定不一样，于是先尝试修改 X-Auth-Project-Id，在关键点设置 pdb 调试断点后，通过调试模式下赋值的方法，发现即使修改过 X-Auth-Project-Id 仍然没有反应。所以一定是token的问题了。

由于不同的 token 导致问题，自然是服务端限制了显示。在nova目录下通过
    
    find . -type f | xargs grep "OS-EXT-SRV-ATTR“

找到了线索。只有这个文件 `nova/api/openstack/compute/contrib/extended_server_attributes.py`通过查看代码，发现在显示 OS-EXT-SRV-ATTR 相关属性的时候，调用了authorize()进行鉴权。问题自然是出现在这里，尝试将鉴权部分的代码注释后，demo 用户也可以取得 OS-EXT-SRV-ATTR。至此，问题就算解决了。但是，不得不修改源码. 后又进一步追踪authorize()的代码发现原来这个接口是读取了 policy.json 文件。

通过查看 /etc/nova/policy.json 文件找到了如下字段

    "compute_extension:extended_server_attributes": [["rule:admin_api"]],

问题的根本原因在这里。简单地将rule:admin_api去掉。后重新测试。demo可以取得OS-EXT-SRV-ATTR:instance_name。问题解决。

