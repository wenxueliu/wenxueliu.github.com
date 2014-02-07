---
layout: post
category : OpenStack
tagline: "Python paste deploy use in OpenStack"
tags : [python, OpenStack]
---
{% include JB/setup %}

If the [paste deploy](http://wenxueliu.github.io/blog/01/27/2014/paste-deploy/) 
which I translate in my blog has been reviewed by you, the OpenStack config will be 
a piece of cake. The neutron config is most complex in OpenStack after I read all the config
files. An example of neutron config as follows:

	[composite:neutron]
	use = egg:Paste#urlmap
	/: neutronversions
	/v2.0: neutronapi_v2_0

	[composite:neutronapi_v2_0]
	use = call:neutron.auth:pipeline_factory
	noauth = extensions neutronapiapp_v2_0
	keystone = authtoken keystonecontext extensions neutronapiapp_v2_0

	[filter:keystonecontext]
	paste.filter_factory = neutron.auth:NeutronKeystoneContext.factory

	[filter:authtoken]
	paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory

	[filter:extensions]
	paste.filter_factory = neutron.api.extensions:plugin_aware_extension_middleware_factory

	[app:neutronversions]
	paste.app_factory = neutron.api.versions:Versions.factory

	[app:neutronapiapp_v2_0]
	paste.app_factory = neutron.api.v2.router:APIRouter.factory

--------------------------------------------------------------

	from oslo.config import cfg
	import webob.dec
	import webob.exc

	from neutron import context
	from neutron.openstack.common import log as logging
	from neutron import wsgi

	LOG = logging.getLogger(__name__)



	class NeutronKeystoneContext(wsgi.Middleware):
		"""Make a request context from keystone headers."""

		@webob.dec.wsgify
		def __call__(self, req):
		    # Determine the user ID
		    user_id = req.headers.get('X_USER_ID')
		    if not user_id:
		        LOG.debug(_("X_USER_ID is not found in request"))
		        return webob.exc.HTTPUnauthorized()

		    # Determine the tenant
		    tenant_id = req.headers.get('X_PROJECT_ID')

		    # Suck out the roles
		    roles = [r.strip() for r in req.headers.get('X_ROLES', '').split(',')]

		    # Human-friendly names
		    tenant_name = req.headers.get('X_PROJECT_NAME')
		    user_name = req.headers.get('X_USER_NAME')

		    # Create a context with the authentication data
		    ctx = context.Context(user_id, tenant_id, roles=roles,
		                          user_name=user_name, tenant_name=tenant_name)

		    # Inject the context...
		    req.environ['neutron.context'] = ctx

		    return self.application


	def pipeline_factory(loader, global_conf, **local_conf):
		"""Create a paste pipeline based on the 'auth_strategy' config option."""
		pipeline = local_conf[cfg.CONF.auth_strategy]
		pipeline = pipeline.split()
		filters = [loader.get_filter(n) for n in pipeline[:-1]]
		app = loader.get_app(pipeline[-1])
		filters.reverse()
		for filter in filters:
		    app = filter(app)
		return app

----------------------------------------------------------------------------------

	def load_paste_app(app_name):
		"""Builds and returns a WSGI app from a paste config file.

		:param app_name: Name of the application to load
		:raises ConfigFilesNotFoundError when config file cannot be located
		:raises RuntimeError when application cannot be loaded from config file
		"""

		config_path = cfg.CONF.find_file(cfg.CONF.api_paste_config)
		if not config_path:
		    raise cfg.ConfigFilesNotFoundError(
		        config_files=[cfg.CONF.api_paste_config])
		config_path = os.path.abspath(config_path)
		LOG.info(_("Config paste file: %s"), config_path)

		try:
		    app = deploy.loadapp("config:%s" % config_path, name=app_name)
		except (LookupError, ImportError):
		    msg = (_("Unable to load %(app_name)s from "
		             "configuration file %(config_path)s.") %
		           {'app_name': app_name,
		            'config_path': config_path})
		    LOG.exception(msg)
		    raise RuntimeError(msg)
		return app
