==============
Authentication
==============

By using a custom authentication backend, you can make use of Django's
authentication framework while storing users in Neo4j.

Add :mod:`neo4django.auth` in your ``INSTALLED_APPS`` setting, and add::

    AUTHENTICATION_BACKENDS = ('neo4django.auth.backends.NodeModelBackend',)

in your settings.py.

To create a new user, use something like::
    
    user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')

Login, reset password, and other included auth views should work as expected.
In your views, :attr:`~Request.user` will contain an instance of 
:class:`neo4django.auth.models.User` for authenticated users.

Referencing Users
=================

Other models are free to reference users. Consider::

    from django.contrib.auth import authenticate

    from neo4django.db import models
    from neo4django.auth import User

    class Post(models.NodeModel):
        title = models.StringProperty()
        author = models.Relationship(User, rel_type='written_by', single=True,
                                     related_name='posts')

    user = authenticate(username='john', password='johnpassword')

    post = Post()
    post.title = 'Cool Music Post'
    post.author = user
    post.save

    assert list(user.posts.all())[0] == post


Customizing Users
=================

Swappable user models are in the works, but until then users can be customized
by subclassing::

    from neo4django.db import models
    from neo4django.auth import User

    class TwitterUser(User):
        follows = models.Relationship('self', rel_type='follows',
                                      related_name='followed_by')

    jack = TwitterUser()
    jack.username = 'jack'
    jack.email = 'jack@example.com'
    jack.set_password("jackpassword')
    jack.save()

    jim = TwitterUser()
    jim.username = 'jim'
    jim.email = 'jim@example.com'
    jim.set_password('jimpassword')
    jim.follows.add(jack)
    jim.save()

The caveats are, first, that :class:`~User` manager shortcuts, like
:func:`~create_user`, aren't available, and that :func:`~authenticate` and other
included functions to work with users will return the wrong model type. This is
fairly straightforward to handle, though, using the included convenience method
:meth:`~neo4django.db.models.NodeModel.from_model`::

    from django.contrib.auth import authenticate

    user = authenticate(username='jim', password='jimpassword')
    twitter_user = TwitterUser.from_model(user)

Permissions
===========

Because neo4django doesn't support :mod:`django.contrib.contenttypes` or an
equivalent, user permissions are not supported. Object-specific or
contenttypes-style permissions would be a great place to `contribute <https://github.com/scholrly/neo4django>`_.
