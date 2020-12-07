# Developers Guide
## Introduction
The OCL API is implemented as a series of modifications to the [Django REST Framework](http://www.django-rest-framework.org/).  These modifications were necessary to allow for the nesting of resources, and are outlined below.  This guide is intended to be used as a supplement to the Django REST Framework [Tutorials](http://www.django-rest-framework.org/#tutorial), and is structured in a similar fashion.

### Serializers

Most `CREATE` and `UPDATE` serializers are implemented as standard `rest_framework.serializers.Serializer` classes.  For these operations, the list of fields specified in the class definition describes the set of fields that may be set (or modified) by the corresponding operation.  For most resource types, some subset of fields will be unmodifiable, so the `UPDATE` view will be distinct from the `CREATE` view. For example, consider:

    class OrganizationUpdateSerializer(serializers.Serializer):
        name = serializers.CharField(required=False)
        company = serializers.CharField(required=False)
        website = serializers.CharField(required=False)

which is distinct from:

    class OrganizationCreateSerializer(serializers.Serializer):
        id = serializers.CharField(required=True, validators=[RegexValidator(regex=NAMESPACE_REGEX)], source='mnemonic')
        name = serializers.CharField(required=True)
        company = serializers.CharField(required=False)
        website = serializers.CharField(required=False)

because the `id` field is unmodifiable.

#### HyperlinkedResourceSerializer

The next most-widely used serializer is the homegrown `oclapi.serializers.HyperlinkedResourceSerializer`.  It is nearly identical to the REST Framework's `rest_framework.serializers.HyperlinkedModelSerializer`, except that it uses a different field type to render the identity ('url') field.

    class HyperlinkedModelSerializer(ModelSerializer):
        ...
        def get_default_fields(self):
            fields = super(HyperlinkedModelSerializer, self).get_default_fields()

            if self.opts.view_name is None:
                self.opts.view_name = self._get_default_view_name(self.opts.model)

            if 'url' not in fields:
                url_field = HyperlinkedIdentityField(
                    view_name=self.opts.view_name,
                    lookup_field=self.opts.lookup_field
                )
                ret = self._dict_class()
                ret['url'] = url_field
                ret.update(fields)
                fields = ret

        return fields

Note that the `HyperlinkedModelSerializer` uses the REST Framework's `rest_framework.relations.HyperlinkedIdentityField` to render its 'url' field.  This field type merely uses `django.core.urlresolvers.reverse` to generate a URL for the object in question, but this implementation of `reverse`does not account for the nesting of resources.

`HyperlinkedResourceSerializer` instead uses a `HyperlinkedResourceIdentifyField`:

    class HyperlinkedResourceSerializer(serializers.Serializer):
        ...
        def get_default_fields(self):
            fields = super(HyperlinkedResourceSerializer, self).get_default_fields()

            if self.opts.view_name is None:
                self.opts.view_name = self._get_default_view_name(self.opts.model)

            if 'url' not in fields:
                url_field = HyperlinkedResourceIdentityField(
                    view_name=self.opts.view_name,
                )
                ret = self._dict_class()
                ret['url'] = url_field
                ret.update(fields)
                fields = ret

        return fields

`HyperlinkedResourceIdentityField` uses `oclapi.utils.reverse_resource` to generate the resource URL, which, unlike `django.core.urlresolvers.reverse`, _does_ account for resource nesting.

The `OrganizationDetailSerializer` is an example of a `HyperlinkedResourceSerializer`:

    class OrganizationDetailSerializer(HyperlinkedResourceSerializer):
        type = serializers.CharField(source='resource_type')
        uuid = serializers.CharField(source='id')
        id = serializers.CharField(source='mnemonic')
        name = serializers.CharField()
        company = serializers.CharField()
        website = serializers.CharField()
        members = serializers.IntegerField(source='num_members')
        publicSources = serializers.IntegerField(source='public_sources')
        createdOn = serializers.DateTimeField(source='created_at')
        updatedOn = serializers.DateTimeField(source='updated_at')

So, in addition to the fields listed above, `OrganizationDetailSerializer` will use `HyperlinkedResourceIdentityField` (which uses `oclapi.utils.reverse_resource`) to render the resource URL for the organization in question.

`HyperlinkedResourceSerializer` is one example of a serializer that renders a **derived field** ('url') in addition to the set of **declared fields**.  Here are some others:

#### HyperlinkedSubResourceSerializer

A sub-resource is a resource that exists only within the context of another resource.  For example, a Source only exists within the context of a User or an Organization.  That User or Organization is referred to as the Source's parent, or owner.

A `HyperlinkedSubResourceSerializer` is used to render the detail view of a sub-resource.  It inherits from `HyperlinkedResourceSerializer`, so it injects a 'url' field, but in addition to that, it also derives and renders an 'ownerUrl' field, which is the resource URL for the sub-resource's parent.

    class HyperlinkedSubResourceSerializer(HyperlinkedResourceSerializer):

        def get_default_fields(self):
            default_fields = super(HyperlinkedSubResourceSerializer, self).get_default_fields()
            parent_resource = self.object.parent if hasattr(self.object, 'parent') else self.object.versioned_object.parent
            default_fields.update({
                'ownerUrl': HyperlinkedResourceOwnerField(view_name=self._get_default_view_name(parent_resource))
            })
            return default_fields

Note that `HyperlinkedSubResourceSerializer` introduces a new field type: `HyperlinkedResourceOwnerField`.  As its name implies, this field type uses `oclapi.utils.reverse_resource` to render the resource URL for the object's parent.

#### ResourceVersionSerializer

Many resource types support versioning, so we use the `ResourceVersionSerializer` to render the detail for a particular version of a resource. `ResourceVersionSerializer` injects 2 fields: 'url' and 'versioned_resource_url'.

The 'url' field is of yet another field type: `HyperlinkedResourceVersionIdentityField`.  This differs from `HyperlinkedResourceIdentifyField` in that it uses `reverse_resource_version`, rather than `reverse_resource`, to render the resource URL (which includes a version identifier at the end).

The 'versioned_resource_url' field is of another type as well: `HyperlinkedVersionedResourceIdentityField`.  (Note the subtle difference in naming from the previous field type.)  Much like `HyperlinkedResourceOwnerField`, this field user `oclapi.utils.reverse_resource` to render the resource URL for a related object, but in this case, the related object is the versionless descriptor of this versioned object; not its parent.  The name 'versioned_resource_url' may be overridden.

### Views
The primary function of a view in the Django REST Framework is to determine the **scope** in which to apply a **serializer**.  Scope is determined by specifying or calculating an initial **QuerySet**.

#### BaseAPIView
The starting point for most views is the `oclapi.views.BaseAPIView`.  This view is an extension of `rest_framework.generics.GenericAPIView` that adds the following bits of functionality:

1. **A post-initialize hook** - You implement the post-initialize hook by overriding the `initialize` method.  This prevents you from having to override `views.GenericAPIView.initial`, which can cause problems if not done carefully.
1. **Decoupling of the URL keyword argument from the lookup field name** - In the base framework, the URL keyword argument and the model instance lookup field are assumed to be the same (e.g. 'pk').  This works for non-nested URLs, but breaks down when a URL specifies multiple nested empties with (potentially) the same lookup field.  The introduction of a `pk_field` allows the lookup field to be distinguished from the URL kwarg.
1. **Performs a soft delete on `destroy()` rather than a hard one** - By default, the `destroy` method on a generic API view performs a hard delete (`Object.delete()`).  OCL API doesn't do any hard deletes, rather it marks the resource as 'inactive'.

#### PathWalkerMixin

Another commonly used base class for OCL API views is the `PathWalkerMixin`.  This class contains methods that can be used recursively to obtain information about parent objects in a nested URL:

1. `get_parent_in_path`: returns the portion of the URL that corresponds to the current resource's parent.
1. `get_object_for_path`: walks up the URL to determine the topmost object, and then resolves the nested object relative to its ancestors.

#### SubResourceMixin

A `SubResourceMixin` inherits from both `BaseAPIView` and `PathWalkerMixin`.  In the post-initialize step provided by `BaseAPIView`, it calls the methods provided by `PathWalkerMixin` to determine the parent (owner) resource and its URL path.  It also ensures that the base QuerySet is limited to objects that are sub-resources of the parent resource.

#### VersionedResourceChildMixin

A further specialization of the `SubResourceMixin` is the `VersionedResourceChildMixin`.  It also determines the version of the parent resource and its class name.  It uses the combination of all these attributes to limit the QuerySet to objects that are sub-resources of a particular version of a parent resource.

#### ResourceVersionMixin

A `ResourceVersionMixin` is similar to a `SubResourceMixin` in that it inherits from both `BaseAPIView` and `PathWalkerMixin`, except that it uses the path-walker methods to infer the version-less base object for a versioned resource.

### Putting it all Together

In this section, we will examine how the components described above are composed to render the CRUD views for concept sources.

#### SourceBaseView

All of the Source CRUD views inherit from the `SourceBaseView`.  `SourceBaseView` inherits from `SubResourceMixin`, which means it is composed of a `BaseAPIView` and a `PathWalkerMixin`.

    class SourceBaseView(SubResourceMixin):
        lookup_field = 'source'
        pk_field = 'mnemonic'
        model = Source

The `lookup_field` is 'source', which means the URL keyword argument named 'source' will specify the identifying field for a given concept source.  The `pk_field` is 'mnemonic', which means that identifying field is 'mnemonic'.  In other words, a URL where the 'source' keyword argument is equal to 'foo' will specify a concept source with a mnemonic equal to 'foo'.

#### SourceRetrieveUpdateDestroyView

In `ocl.urls`, we have:

    url(r'^orgs/', include('orgs.urls')),
    url(r'^users/', include('users.urls')),

In `orgs.urls` (and similarly in `users.urls`):

    url(r'^(?P<org>[a-zA-Z0-9\-\.]+)/sources/', include('sources.urls')),

In `sources.urls`:

    url(r'^(?P<source>[a-zA-Z0-9\-\.]+)/$', SourceRetrieveUpdateDestroyView.as_view(), name='source-detail'),

So a URL that matches the pattern:

    r'^orgs/(?P<org>[a-zA-Z0-9\-\.]+)/sources/(?P<source>[a-zA-Z0-9\-\.]+)/$'

will resolve to the `SourceRetrieveUpdateDestroyView`.

In addition to `SourceBaseView`, `SourceRetrieveUpdateDestroyView` also inherits from `rest_framework.generics.RetrieveAPIView`, `rest_framework.generics.UpdateAPIView`, and `rest_framework.generics.DestroyUpdateAPIView`.  This means that the view will respond to:

1. `GET` with the method `retrieve`
1. `PUT` with the method `update`
1. `DELETE` with the method `destroy`

based on these three parent classes, respectively.  This is (almost) all the functionality it needs.  By virtue of inheriting from `SubResourceMixin`, its parent resource - either a user or an organization - will be inferred from the URL path, and this particular resource will be resolved based on its mnemonic, relative to its parent.

Now, we just need a serializer to tell us what fields to render.  Consider the request:

    GET /orgs/WHO/sources/ICD-10/

The URL matches the pattern above, so the request will be handled by the `SourceReadUpdateDestroyView`.  Now let's examine what happens in there:

    def initial(self, request, *args, **kwargs):
        if 'GET' == request.method:
            self.permission_classes = (CanViewCollection,)
            self.serializer_class = SourceDetailSerializer
        else:
            self.permission_classes = (CanEditCollection,)
            self.serializer_class = SourceUpdateSerializer
        super(SourceRetrieveUpdateDestroyView, self).initial(request, *args, **kwargs)

We can ignore the permissioning bit for now.  The key point here is that, if the request method is a `GET` (as it is in our example), the `SourceDetailSerializer` will be used to render the result.  The `SourceDetailSerializer` looks like this:

    class SourceDetailSerializer(HyperlinkedSubResourceSerializer):
        type = serializers.CharField(required=True, source='resource_type')
        uuid = serializers.CharField(required=True, source='id')
        id = serializers.CharField(required=True, source='mnemonic')
        shortCode = serializers.CharField(required=True, source='mnemonic')
        name = serializers.CharField(required=True)
        fullName = serializers.CharField(source='full_name')
        sourceType = serializers.CharField(required=True, source='source_type')
        publicAccess = serializers.CharField(source='public_access')
        defaultLocale = serializers.CharField(source='default_locale')
        supportedLocales = serializers.CharField(source='supported_locales')
        website = serializers.CharField()
        description = serializers.CharField()
        owner = serializers.CharField(source='parent_resource')
        ownerType = serializers.CharField(source='parent_resource_type')
        versions = serializers.IntegerField(source='num_versions')
        createdOn = serializers.DateTimeField(source='created_at')
        updatedOn = serializers.DateTimeField(source='updated_at')

and will render this:

    {
        type: "Source"
        uuid: "5289334c1d1e986dfbe55f30"
        id: "ICD-10"
        shortCode: "ICD-10"
        name: "International Classification of Diseases, v10"
        fullName: null
        sourceType: "dictionary"
        publicAccess: "View"
        defaultLocale: "en"
        supportedLocales: null
        website: null
        description: null
        owner: "WHO"
        ownerType: "Organization"
        versions: 1
        createdOn: "2013-11-17T16:21:16.303"
        updatedOn: "2013-11-17T16:21:16.303"
        url: "/orgs/WHO/sources/ICD-10/"
        ownerUrl: "/orgs/WHO/"
    }

Note that the last two fields, `url` and `ownerUrl`, are not declared, but rather they are derived from the URL specified by virtue of the fact that this is a `HyperlinkedSubResourceSerializer`.

### Users & Roles

#### Authentication
The OCL API uses a token-based authentication scheme, as provided by the Django REST Framework (described [here](http://www.django-rest-framework.org/api-guide/authentication#tokenauthentication)).  Tokens are automatically generated and assigned to new users when they are created via the `POST /users` operation.  They are stored in a Mongo collection called `authtoken_token` and are also accessible via the Django admin interface.

#### Permissions
The permissioning system employed by the OCL API is also derived from the one provided by the Django REST Framework (described [here](http://www.django-rest-framework.org/tutorial/4-authentication-and-permissions#adding-required-permissions-to-views)).  By default, all API endpoints are protected by the `IsAuthenticated` permission class.

From `settings.py`

    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ]

In other words, a valid authentication token must be provided as a minimal requirement to access any OCL API endpoint.  Permissions may be overridden for a particular view (or a particular operation on a view) by specifying a value for the `permission_classes` instance variable.

Some other permission classes provided by the OCL API are `HasOwnership` and `HasPrivateAccess`.  These are defined in `oclapi.permissions`:

    class HasOwnership(BasePermission):
        """
        The request is authenticated, and the user is a member of the referenced organization
        """
        def has_object_permission(self, request, view, obj):
            if request.user.is_staff:
                return True
            if request.user.is_authenticated and hasattr(request.user, 'get_profile'):
                userprofile = request.user.get_profile()
                if isinstance(obj, UserProfile):
                    return obj == userprofile
                elif isinstance(obj, Organization):
                    return userprofile.id in obj.members
                return True
            return False

    class HasPrivateAccess(BasePermission):
        """
        Current user is authenticated as a staff user, or is designated as the referenced object's owner,
        or belongs to an organization that is designated as the referenced object's owner.
        """
        def has_object_permission(self, request, view, obj):
            if request.user.is_staff:
                return True
            if request.user == obj.owner:
                return True
            if request.user.is_authenticated and hasattr(request.user, 'get_profile'):
                profile = request.user.get_profile()
                if obj.parent_id in profile.organizations:
                    return True
            return False

`HasPrivateAccess` is used to restrict many views, such as the `SourceBaseView`:

    class SourceBaseView(SubResourceMixin):
        lookup_field = 'source'
        pk_field = 'mnemonic'
        model = Source
        queryset = Source.objects.filter(is_active=True)
        permission_classes = (HasPrivateAccess,)

This means you cannot perform any operations on a Source unless you are one of the following:

1. The User who is the owner of that Source
2. A member of the Organization that is the owner of the Source
3. An admin user

The usage of `HasOwnership` is a bit more subtle.  Consider the `OrganizationDetailView`:

    class OrganizationDetailView(mixins.UpdateModelMixin,
                                 OrganizationBaseView):
        serializer_class = OrganizationDetailSerializer
        queryset = Organization.objects.filter(is_active=True)

        def initial(self, request, *args, **kwargs):
            if (request.method == 'DELETE') or (request.method == 'POST'):
                self.permission_classes = (HasOwnership, )
            super(OrganizationDetailView, self).initial(request, *args, **kwargs)

In some cases (i.e. the `GET` operation), the base permissions will suffice, but in others (i.e. `DELETE` and `POST`), we override the base permissions with the `HasOwnership` class.  This is to ensure that no one outside of an organization may modify or delete it.
