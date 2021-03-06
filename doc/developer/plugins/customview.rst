.. highlight:: python
   :linenothreshold: 5

.. _`customview`:

Creating custom views
=====================

This page describes how to provide a custom view from within your plugin. Before you start
reading this page, please read and understand how :ref:`URL handling <urlconf>` works in
pretalx.

Orga panel views
----------------

If you want to add a custom view to the organiser area of an event, just register an URL in your
``urls.py`` that lives in the ``/orga/`` subpath::

    from django.conf.urls import url

    from . import views

    urlpatterns = [
        url(r'^orga/event/(?P<event>[^/]+)/mypluginname/',
            views.admin_view, name='backend'),
    ]

It is required that your URL parameters is called ``event``.

You can then implement the view as you would normally do. Our middleware will automatically
detect the ``/orga/`` subpath and will ensure the following things if this is an URL with
the ``event`` parameter:

* The user is logged in
* The ``request.event`` attribute contains the current event
* The user has permission to view the current event

If you want to require specific permission types, we provide you with a decorator or a mixin for
your views::

    from pretalx.common.mixins.views import PermissionRequired

    class AdminView(PermissionRequired, View):
        permission_required = 'can_view_orders'

        ...


There is also a signal that allows you to add the view to the event sidebar navigation like this::


    from django.core.urlresolvers import resolve, reverse
    from django.dispatch import receiver
    from django.utils.translation import ugettext_lazy as _
    from pretalx.orga.signals import nav_event


    @receiver(nav_event, dispatch_uid='friends_tickets_nav')
    def navbar_info(sender, request, **kwargs):
        url = resolve(request.path_info)
        if not request.user.has_perm('can_see_orga_area', request.event):
            return []
        return [{
            'label': _('My plugin view'),
            'icon': 'heart',
            'url': reverse('plugins:myplugin:index', kwargs={
                'event': request.event.slug,
            }),
            'active': url.namespace == 'plugins:myplugin' and url.url_name == 'view',
        }]


Frontend views
--------------

Frontend views work pretty much exactly like organiser area views. Take care that your url starts
with ``/(P?P<event>[\]+)/``.
You can then implement a view as you would normally do. It will be automatically ensured that:

* The requested event exists
* The requested event is visible (either by being public, or by being viewed by an organiser)
* The event is accessed via the domain it should be accessed
* The ``request.event`` attribute contains the correct ``Event`` object
* Your plugin is enabled
* The locale is set correctly


.. _Django REST Framework: http://www.django-rest-framework.org/
.. _ViewSets: http://www.django-rest-framework.org/api-guide/viewsets/
.. _Routers: http://www.django-rest-framework.org/api-guide/routers/
