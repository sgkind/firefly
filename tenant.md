Tenant App程序解读
===

> tenant app用来在mars环境中创建租户(tenant)、segment和vlan/vxlan(segment)
其主要实现的功能有：
1. 添加或删除租户，获取mars中的全部或指定租户的相关信息
2. 为指定租户添加或删除segment，获取mars中全部或指定segment的相关信息
3. 从指定租户的指定segment中删除指定交换机
4. 为指定租户的指定segment中指定的交换机添加vlan
5. 获取指定租户中指定segment的全部vlan的相关信息
6. 获取指定租户中指定segment中指定交换机的vlan的相关信息
7. 为指定租户的指定segment添加vxlan
8. 删除指定租户的指定segment中的所有或指定vxlan access
9. 删除指定租户的指定segment中的所有或指定vxlan network

## 程序主要逻辑
tenant app主要的配置数据以键值对的形式保存在netcfg中。采用REST API对tenant、segment以及与此相关的vlan、vxlan进行操作时主要是对netcfg中的数据进行读取、添加和删除操作。netcfg中保存的数据发生变更时(添加、删除、更新)会产生相关的事件。同时在TenantManager中向netcfg内注册了相关事件的监听器，相关事件发生后在事件监听器内对其进行处理。

## 保存的数据
### netcfg中保存的数据
#### tenant
key: TenantId
* String tenantId

value: TenantBasicConfig
* String type (可取的值：normal/system)

#### tenantSegment
key: TenantSegmentId
* TenantId tenantId
* SegmentId segmentId

value: TenantSegmentBasicConfig
* String type (可取的值：vlan/vxlan)
* String value (代表vlan id)
* Set<String> ipAddresses

#### tenantSegmentVlan
key: TenantSegmentDeviceId
* TenantId tenantId
* SegmentId segmentId
* DeviceId deviceId

value: TenantSegmentVlanConfig
* int vid (vlan id)
* Set<String> ports
* Set<String> logicalPorts
* Set<String> macBasedVlans

#### tenantSegmentVxlanNetwork
key: TenantSegmentPortId
* TenantId tenantId
* SegmentId segmentId
* String portName

value: TenantSegmentVxlanNetworkConfig
* int vni (VXLAN Network Identifier)
* Set<String> ipAddresses
* String uplinkSegment

#### tenantSegmentVxlanAccessNormal
key: TenantSegmentPortId
* TenantId tenantId
* SegmentId segmentId
* String portName

value: TenantSegmentVxlanAccessNormalConfig
* int vni (VXLAN Network Identifier)
* String deviceId (交换机Id)
* int port
* int vlanId

#### tenantSegmentVxlanAccessOpenstack
key: TenantSegmentPortId
* TenantId tenantId
* SegmentId segmentId
* String portName

value: TenantSegmentVxlanAccessOpenstack
* int vni (VXLAN Network Identifier)
* String serverMac
* int vlanId

#### DeviceLoopback
key: DeviceLoopbackId
* DeviceId deviceId
* int loopbackId

value: DeviceLoopbackConfig
* String ipAddress

### ConsistentMap中保存的数据
#### devicePortPvidMap  ConsistentMap<String, Integer> 
保存未打标签(untag)的端口号和逻辑端口号与vlan id的对应关系
其中:
* key为deviceIdStr/<端口号>
* value为vlan id

#### vidTenantSegmentMap    ConsistentMap<Integer, TenantSegmentId> 
保存vlan id与TenantSegment的对应关系
其中:
* key为vlan id
* value为TenantSegment

## 接口
### GET tenant/v1
获取mars中所有的租户

进行的操作：直接从netcfg中获取数据并展示给前端

### POST tenant/v1
新建一个租户

进行的操作：如果同名tenant已经存在则修改tenant的类型，否则直接向netcfg中添加一个TenantBasicConfig

### DELETE tenant/v1/{tenant_id}
删除指定的租户，并删除此租户拥有的TenantSegmentVxlanAccessNormalConfig、TenantSegmentVxlanAccessOpenStackConfig、TenantSegmentVxlanNetworkConfig和TenantSegmentVlanConfig。

进行的操作：
1. 从netcfg中删除此租户下所以TenantSegmentVxlanAccessNormalConfig
2. 从netcfg中删除此租户下所有TenantSegmentVxlanAccessOpenstackConfig
3. 从netcfg中删除此租户下所有TenantSegmentVxlanNetworkConfig
4. 从netcfg中删除此租户下所有TenantSegmentVlanConfig
5. 从netcfg中删除此租户下所有TenantSegmentBasicConfig
6. 从netcfg中删除租户

### DELETE tenant/v1
删除所有的租户，并删除所有的TenantSegmentVxlanAccessNormalConfig、TenantSegmentVxlanAccessOpenStackConfig、TenantSegmentVxlanNetworkConfig和TenantSegmentVlanConfig。

进行的操作：
1. 从netcfg中删除所有TenantSegmentVxlanAccessNormalConfig
2. 从netcfg中删除所有TenantSegmentVxlanAccessOpenstackConfig
3. 从netcfg中删除所有TenantSegmentVxlanNetworkconfig
4. 从netcfg中删除所有TenantSegmentVlanConfig
5. 从netcfg中删除所有TenantSegmentBasicConfig
6. 从netcfg中删除所有TenantBasicConfig

### GET tenant/v1/segments
获取mars中所有的segment。

进行的操作：
从netcfg中获取所有的segment，并向前端返回租户名、租户类型、segment名、segment类型、ip address及segment值(vlan)

### GET tenant/v1/{tenant_name}/segments
获取mars中指定租户的所有segment

进行的操作:
1. 从netcfg中获取所有的segment
2. 将租户名与指定租户名相同的segment的名字、类型、ip地址和value(vlan)返回给前端


### GET tenant/v1/{tenant_name}/segments/{segment_name}
获取指定租户(tenant)中的指定segment

进行的操作：
从netcfg中获取指定租户的指定segment，将此segment的名字、类型、ip地址和value(vlan)返回给前端

### POST tenant/v1/{tenant_name}/segments
向指定租户(tenant)添加一个segment

json中的参数:
1. name segment名
2. type segment type(vlan/vxlan)
3. value (当type是vlan时vlan id)
4. ip_address

进行的操作：
1. 校验参数是否正确
2. 如果segment已经存在，则更新参数，否则新建TenantSegmentBasicConfig

### DELETE tenant/v1/{tenant_name}/segments/{segment_name}
删除指定租户(tenant)中的指定segment，并删除此segment所属的TenantSegmentVlanconfig、TenantSegmentVxlanNetworkConfig、TenantSegmentVxlanAccessOpenstackConfig和TenantSegmentVxlanAccessNormalConfig

进行的操作：
1. 从netcfg中删除指定segment所属的TenantSegmentVxlanAccessNormalConfig
2. 从netcfg中删除指定segment所属的TenantSegmentVxlanAccessOpenstackConfig
3. 从netcfg中删除指定segment所属的TenantSegmentVxlanNetworkConfig
4. 从netcfg中删除指定segment所属的TenantSegmentVlanConfig
5. 从netcfg中删除指定的segment

### GET tenant/v1/{tenant_name}/segments/{segment_name}/vlan
获取指定租户(tenant)中指定segment所属的所有vlan

进行的操作：
1. 从netcfg中获取指定租户中指定segment在不同交换机上的TenantSegmentVlanConfig
2. 向前端返回TenantSegmentVlanConfig中的端口、逻辑端口和mac based vlan

### GET tenant/v1/{tenant_name}/segments/{segment_name}/device/{device_name}/vlan
获取指定租户(tenant)中指定segment中指定交换机(device)所属的所有vlan

进行的操作:
1. 从netcfg中获取指定租户中指定segment在指定交换机(Device)上的TenantSegmentVlanConfig
2. 向前端返回TenantSegmentVlanConfig中的端口、逻辑端口和mac based vlan

### POST tenant/v1/{tenant_name}/segments/{segment_name}/device/{device_id}/vlan
给指定租户指定segment向指定交换机(device)添加vlan

进行的操作：
1. 检查segment是否存在
2. 验证vlan的参数是否正确
3. 如果TenantSegmentVlanConfig已经存在则删除其中的所有port、logical port和mac based vlan，然后添加需要设置的port、logical port和mac based vlan
4. 如果TenantSegmentVlanConfig不存在，则添加新的TenantSegmentVlanConfig，然后设置其port、logical port和mac based vlan

### DELETE tenant/v1/{tenant_name}/segments/{segment_name}/device/{device_name}
从指定租户的指定segment中删除指定的交换机(device)

进行的操作：
直接从netcfg中删除于给定TenantSegmentDeviceId相关的值

### GET tenant/v1/{tenant_name}/segments/{segment_name}/vxlan
获取指定租户(tenant)中指定segment的所有vxlan

进行的操作：
1. 向前端返回指定租户中指定segment的所有TenantSegmentVxlanAccessNormalConfig的内容
2. 向前端返回指定租户中指定segment的所有TenantSegmentVxlanOpenstackConfig的内容
3. 向前端返回指定租户中指定segment的所有TenantSegmentVxlanNetworkConfig的内容

### POST tenant/v1/{tenant_name}/segments/{segment_name}/vxlan
向指定租户(tenant)的指定segment中添加添加vxlan

进行的操作：
1. 检查segment是否存在
2. 如果请求参数中有access port，则对每一个access port添加一个TenantSegmentVxlanAccessNormalConfig或TenantSegmentVxlanAccessOpenstackConfig
3. 如果请求参数中有network port，则对每一个network port添加一个TenantSegmentVxlanNetworkconfig

### DELETE tenant/v1/{tenant_name}/segments/{segment_name}/vxlan/access
删除指定租户指定segment中的vxlan access(TenantSegmentVxlanAccessNormalConfig和TenantSegmentVxlanAccessOpenstackConfig) 

进行的操作: 
1. 从netcfg中删除属于指定租户指定segment中的所有TenantSegmentVxlanAccessNormalConfig
2. 从netcfg中删除属于指定租户指定segment中的所有TenantSegmentVxlanAccessOpenstackConfig 

### DELETE tenant/v1/{tenant_name}/segments/{segment_name}/vxlan/access/{name}
删除指定租户指定segment中指定的vxlan access

进行的操作:
1. 从netcfg中删除指定的TenantSegmentVxlanAccessNormalConfig
2. 从netcfg中删除指定的TenantSegmentVxlanAccessOpenstackConfig

### DELETE tenant/v1/{tenant_name}/segments/{segment_name}/vxlan/network
删除指定租户指定segment中的vxlan network

进行的操作:
从netcfg中删除属于指定租户指定segment的所有TenantSegentVxlanNetworkConfig

### DELETE tenant/v1/{tenant_name}/segments/{segment_name}/vxlan/network/{name}
删除指定租户指定segment中指定的vxlan network

进行的操作:
从netcfg中删除指定的TenantSegmentVxlanNetworkConfig

### GET tenant/v1/pvid
获取所有的pvid(Port VLAN identifier.)

进行的操作：
1. 获取devicePortPvidMap的值
2. 将所有端口与pvid的值返回前端

### POST tenant/v1/loopback/{device_id}
为mars设置环回接口

进行的操作：
1. 判断参数是否合法
2. 判断mars是否未设置环回接口，如果已设置则返回错误信息（mars中最多只能设置一个环回接口）
3. 向netcfg中添加DeviceLoopbackConfig

### DELETE tenant/v1/loopback/{device_id}/{loopback_id}
删除指定的环回接口

进行的操作：
相关环回接口在netcfg中存在则从netcfg中删除

### GET tenant/v1/loopback
获取mars中的环回接口

进行的操作：
如果netcfg中存在环回接口，则查询后向前端返回环回接口的相关信息

## 事件监听器

### InternalSpineLeafLinkListener
处理spine和leaf交换机之间连接(SpineLeafLink)的添加、删除和更新。

1. 监听到的事件为LINK_REMOVE事件时，调用updateLink(spineLeaf, false)执行SpineLeafLink的删除操作，函数调用：`updateLink->addVlanMember->setPortVlanTagMembers/SwitchVlanConfig.addVlanMember`;
2. 监听到的事件为LINK_ADD或LINK_UPDATED事件时，调用updateLink(spineLeaf, true)执行spineLeafLink的添加或更新操作，函数调用：`updateLink->deleteVlanMeber->setPortVlanTagMembers/SwitchVlanConfig.deleteVlanMember`

```
graph TB
st(开始) --> op[操作]
```




