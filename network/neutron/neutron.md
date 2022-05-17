neutron
===

## setup.cfg
在setup.cfg中可以看到各个函数入口位置，以下是主要的程序入口
1. neutron-server: neutron.cmd.eventlet.server:main

## api-paste.ini

## 程序
| 文件夹 | 作用 | 备注 |
| --- | --- | ---- |
| cmd | 各个程序的入口函数| |
| common | | |
| conf | 配置文件相关 | |
| extensions | | |

## 备注
1. neutron.callbacks中的代码已废弃，代码已经移动到neutron-lib包中

## neutron-server程序执行流程

### 一、程序入口
neutron.cmd.eventlet.server.main

```python
def main():
    server.boot_server(wsgi_eventlet.eventlet_wsgi_server)
```

server.boot_server函数在neutron.server模块中，位于文件__init__.py，代码如下
```python
def boot_server(server_func):
    _init_configuration()
    try:
        server_func()
    except KeyboardInterrupt:
        pass
    except RuntimeError as e:
        sys.exit(_("ERROR: %s") %e)
```
此函数首先初始化配置，然后运行从参数中传进来的函数即wsgi_eventlet.eventlet_wsgi_server，在eventlet_wsgi_server函数执行期间忽略键盘的中断事件。可见neutron-server的主要逻辑在eventlet_wsgi_server函数中

wsgi_eventlet.eventlet_wsgi_server函数位于neutron.server.wsgi_eventlet中。代码为:
```python
def eventlet_wsgi_server():
    neutron_api = service.serve_wsgi(service.NeutronApiService)
    start_api_and_prc_workers(neutron_api)
```

service.NeutronApiService是用来实现Neutron api并为外部提供api服务的service，service.serve_wsgi函数创建一个NeutronApiService并调用NeutronApiService的start函数启动此service

NeutronApiService类的创建比较简单，调用类的静态方法create即可创建一个app_name为neutron的NeutronApiService对象。NeutronApiService对象启动时，在函数start中会调用`_run_wsgi(self.app_name)`函数，代码如下。openstack wsgi比较复杂暂时没搞清楚，在此不做记录。
```python
def _run_wsgi(app_name):
    app = config.load_paste_app(app_name)
    if not app:
        LOG.error('No known API applications configured.')
        return
    return run_wsgi_app(app)
```
其中关键的代码是`config.load_paste_app(app_name)`，此函数定义在neutron.common.config.py文件中
```python
def load_paste_app(app_name):
    """Builds and returns a WSGI app from a paste config file.

    :param app_name: Name of the application to load
    """
    loader = wsgi.Loader(cfg.CONF)
    app = loader.load_app(app_name)
    return app
```

此函数调用oslo_service包中的wsgi模块中的代码。没有时间看代码，直接搜索上结论。
这块代码的作用是调用neutron的api-paste.ini配置文件创建一个wsgi的application，将资源(端口、网络、子网)映射到URL上。

### 二、api-paste.ini
```
[composite:neutron]
use = egg:Paste#urlmap
/: neutronversions_composite
/v2.0: neutronapi_v2_0

[composite:neutronapi_v2_0]
use = call:neutron.auth:pipeline_factory
noauth = cors http_proxy_to_wsgi request_id catch_errors extensions neutronapiapp_v2_0
keystone = cors http_proxy_to_wsgi request_id catch_errors authtoken keystonecontext extensions neutronapiapp_v2_0

[composite:neutronversions_composite]
use = call:neutron.auth:pipeline_factory
noauth = cors http_proxy_to_wsgi neutronversions
keystone = cors http_proxy_to_wsgi neutronversions

[filter:request_id]
paste.filter_factory = oslo_middleware:RequestId.factory

[filter:catch_errors]
paste.filter_factory = oslo_middleware:CatchErrors.factory

[filter:cors]
paste.filter_factory = oslo_middleware.cors:filter_factory
oslo_config_project = neutron

[filter:http_proxy_to_wsgi]
paste.filter_factory = oslo_middleware.http_proxy_to_wsgi:HTTPProxyToWSGI.factory

[filter:keystonecontext]
paste.filter_factory = neutron.auth:NeutronKeystoneContext.factory

[filter:authtoken]
paste.filter_factory = keystonemiddleware.auth_token:filter_factory

[filter:extensions]
paste.filter_factory = neutron.api.extensions:plugin_aware_extension_middleware_factory

[app:neutronversions]
paste.app_factory = neutron.api.versions:Versions.factory

[app:neutronapiapp_v2_0]
paste.app_factory = neutron.api.v2.router:APIRouter.factory

[filter:osprofiler]
paste.filter_factory = osprofiler.web:WsgiMiddleware.factory
```
从配置文件中可以看出这里映射了两个url: '/'和'/v2.0'。
* '/'映射到composite:neutronversions_composite
* '/v2.0'映射到composite:neutronapi_v2_0

上面两个composite经过一系列过滤后最终分别调用了neutronapiapp_v2_0和neutronversions。neutronversions被映射为neutron.api.versions:Versions.factory，而neutronapiapp_v2_0被映射为neutron.api.v2.router:APIRouter.factory。也即'/'请求的最终入口是neutron.api.versions:Versions.factory，'/v2.0'请求的最终入口是neutron.api.v2.router:APIRouter.factory。

'/'逻辑比较简单，在此不做讨论。

### 三、APIRouter类
```
class APIRouter(base_wsgi.Router):

    @classmethod
    def factory(cls, global_config, **local_config):
        if cfg.CONF.web_framework == 'pecan':
            return pecan_app.v2_factory(global_config, **local_config)
        return cls(**local_config)

    def __init__(self, **local_config):
        mapper = routes_mapper.Mapper()
        manager.init()
        plugin = directory.get_plugin()
        ext_mgr = extensions.PluginAwareExtensionManager.get_instance()
        ext_mgr.extend_resources("2.0", attributes.RESOURCE_ATTRIBUTE_MAP)

        col_kwargs = dict(collection_actions=COLLECTION_ACTIONS,
                          member_actions=MEMBER_ACTIONS)

        def _map_resource(collection, resource, params, parent=None):
            allow_bulk = cfg.CONF.allow_bulk
            controller = base.create_resource(
                collection, resource, plugin, params, allow_bulk=allow_bulk,
                parent=parent, allow_pagination=True,
                allow_sorting=True)
            path_prefix = None
            if parent:
                path_prefix = "/%s/{%s_id}/%s" % (parent['collection_name'],
                                                  parent['member_name'],
                                                  collection)
            mapper_kwargs = dict(controller=controller,
                                 requirements=REQUIREMENTS,
                                 path_prefix=path_prefix,
                                 **col_kwargs)
            return mapper.collection(collection, resource,
                                     **mapper_kwargs)

        mapper.connect('index', '/', controller=Index(RESOURCES))
        for resource in RESOURCES:
            _map_resource(RESOURCES[resource], resource,
                          attributes.RESOURCE_ATTRIBUTE_MAP.get(
                              RESOURCES[resource], dict()))
            resource_registry.register_resource_by_name(resource)

        for resource in SUB_RESOURCES:
            _map_resource(SUB_RESOURCES[resource]['collection_name'], resource,
                          attributes.RESOURCE_ATTRIBUTE_MAP.get(
                              SUB_RESOURCES[resource]['collection_name'],
                              dict()),
                          SUB_RESOURCES[resource]['parent'])

        # Certain policy checks require that the extensions are loaded
        # and the RESOURCE_ATTRIBUTE_MAP populated before they can be
        # properly initialized. This can only be claimed with certainty
        # once this point in the code has been reached. In the event
        # that the policies have been initialized before this point,
        # calling reset will cause the next policy check to
        # re-initialize with all of the required data in place.
        policy.reset()
        super(APIRouter, self).__init__(mapper)
```

factory是一个工厂方法用来构造APIRouter对象。构造对象时会调用__init__方法。
首先声明一个routes_mapper的mapper，可以通过此对象的connect方法构造URL和对应controller的映射。在此函数中将'/v2.0/'映射到Index(RESOURCES)控制器，然后分别对Core Resource和Sub Resource中的资源进行映射。当对相关资源进行操作时，会调用相关的controller去执行响应的操作。

至此neutron模块api调用大体流程已经清楚，但目前还不清楚controller具体怎么实现相关的操作。此函数中还有三个点未分析：
1. manager的初始化`manager.init()`
2. 各个控制器的创建，及其相关操作`base.create_resource()`
3. 扩展功能

### manager的初始化
manager的初始化是在`neutron.api.v2.router:APIRouter.__init__`中调用的，语句为`manager.init()`

`manager.init`函数位于neutron.manager中，通过单例模式创建NeutronManager，NeutronManager的作用是加载core plugin及其service以及其他service plugin，并将其存入directory中，供其他地方调用。

代码逻辑比较简单，分析略。

### 控制器的创建
在APIRouter中多次调用了`base.create_resource(...)`函数创建控制器。`create_resource`函数位于neutron.api.v2.base中。

### plugin
#### core plugin
core plugin中有TypeManager、ExtensionManager和MechansimManager。此三个Manager分别管理TypeDriver、ExtensionDriver和MechansimDriver。TypeDriver执行物理网络的创建；MechansimDriver执行网络、子网和端口的增加、删除及更新工作；ExtensionDriver进行core plugin的其他一些工作。



metering plugin



### RPC

plugin中有rpc server， agent通过rpc client与plugin的rpc server通信获取信息
plugin中也有rpc client，agent中起rpc server，plugin通过此rpc通知agent进行必要的操作

## 其他
### extensions、db和plugin的关系

extensions里面定义了资源类和Plugin的抽象类xxxPluginBase，在此抽象类中定义了Plugin与agent交互的接口函数

在db里面基于定义的xxxmixin类继承自xxxPluginBase，主要进行数据库的操作

plugin中各个Plugin继承自db中的xxxmixin类，plugin中会启动rpc server和client，与agent通信

extensions里面定义了xxxPluginBase，定义了Plugin与agent交互的接口函数，db里面的类继承xxxPluginBase，plugin继承db里面定义的类
