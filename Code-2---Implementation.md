# Code 2 - Implementation

## 1. Project Structure
   1. The project shall contain an app/directory named `goalie` as the main project
   2. Feature implementation shall be implemented in separate apps from the main app
      1. Feature app shall be named with following pattern

             goalie_<feature_name>

      2. `<feature_name>` shall contain a noun that describe the main idea about the feature
      3. A feature shall be implemented in separate app when:
         - WIP

   3. All communication between apps shall be happened through signal. Each app shall be loose-coupled, which mean there is no function call allowed between app. 

## 2. Feature App Structure
   1. An app shall be created with` django-admin createapp` command. 
   2. An app shall have following basic structure:
         
          <app_name>/
              __init__.py
              apps.py
              admin.py
              authentications.py
              handlers.py
              models.py
              permissions.py
              serializers.py
              views.py
              urls.py

## 3. View
   1. All views shall be implemented in `views.py` from the app directory
   2. View implementation shall following this precedence:
      - Inherit available viewsets classes (_ModelViewSet_, _ReadOnlyModelViewSet_, etc), together with available routers classes (_DefaultRouters_, etc)

        ```python
         ### in views.py
         from rest_framework import viewsets
         
         class UserViewSet(viewsets.ModelViewSet):
             """
             A viewset for viewing and editing user instances.
             """
             ....
         
         
         ### in urls.py
         from myapp.views import UserViewSet
         from rest_framework.routers import DefaultRouter
         
         router = DefaultRouter()
         router.register(r'users', UserViewSet, basename='user')
         urlpatterns = router.urls
         ```

      - Inherit _GenericViewSet_, with or without available mixins (_CreateModelMixin_, _ListModelMixin_, etc)

        ```python
        ### in views.py
        from rest_framework import viewsets
        
        class WriteOnlyUser(CreateModelMixin, ListModelMixin, GenericViewSet):
            """
            A viewset for creating and listing user instances.
            """
            ....
        
        
        ### in urls.py
        from myapp.views import UserViewSet
        from rest_framework.routers import DefaultRouter
        
        router = DefaultRouter()
        router.register(r'users', UserViewSet, basename='user')
        urlpatterns = router.urls
        ```

      - Inherit _GenericAPIView_, with or without available mixins (_CreateModelMixin_, _ListModelMixin_, etc)

        ```python
        ### in views.py
        from rest_framework import generics
          
        class ActivateUser(generics.GenericAPIView):
            ....
        
        ### in urls.py
        from .views import SensorView
        
        urlpatterns = [
            url(r'^(?P<_id>[a-z0-9]+)/activate/$', ActivateUser.as_view(), name='user-activate'),
        ]
        ```

      - Inherit base _APIView_ class

        ```python
        ### in views.py
        class ListUsers(APIView):
            ....
        
        ### in urls.py
        urlpatterns = [
            url(r'^', SensorView.as_view(), name='sensor-list'),
        ]
        ```

      - Function based view with _@api_view_ decorator
        ```python
        ### in views.py
        from rest_framework.decorators import api_view
        
        @api_view(('POST', 'GET'))
        def accept_invitation(request, key):
            ....
        
        ### in urls.py
        urlpatterns = [
            url(r'^accept-invitation/(?P<key>\w+)/?$',
                accept_invitation,
                name='accept-invite'),
        ]
        ```

   3. In case of using GenericAPIView or it's derivative (all generics view, ModelViewSet, etc), customization shall be done using their available API as long as possible

      1. To apply custom **authentication classes** (other than specified in `REST_FRAMEWORK.DEFAULT_AUTHENTICATION_CLASSES` settings):
         - override _authentication_classes_ attribute of the view class

           ```python
           from knox import views as knox_views
           from .auth import BasicAuthentication
           
           class LoginView(knox_views.LoginView):
           
               authentication_classes = (BasicAuthentication,)
           ```

         - override _get_authenticators_ method of the view class

           ```python
           class LoginView(knox_views.LoginView):
           
               def get_authenticators(self):
                   if self.request.method == 'GET':
                       return [TokenAuthentication()]
                   else:
                       return [BasicAuthentication()]
           ```

      2. To apply custom **permission classes** (other than specified in `REST_FRAMEWORK.DEFAULT_PERMISSION_CLASSES` settings):

         - override _permission_classes_ attribute of the view class

            ```python
            class NetworkMemberViewSet(GoalieLoggingMixin, viewsets.ModelViewSet):

                permission_classes = (permissions.IsAuthenticated, IsMemberOfNetwork)
            ```

         - override _get_permission_classes_ method of the view class (usually preferred when permission is different depend on action or request method)

           ```python
           class NetworkViewSet(viewsets.ModelViewSet):
               
               def get_permissions(self):
                   if self.action == 'destroy':
                       permission_classes = [permissions.IsAuthenticated, IsAllowedToDelete]
                   else:
                       permission_classes = [permissions.IsAuthenticated]
                   return [permission() for permission in permission_classes]
            ```

      2. To define data set that will be returned in response (in order):
         - override _queryset_ attribute in view class with Model class of the data
         - override _filter_backends_ attribute in view class with available generic filter
         - when operate with single object where primary key value provided in URL (like `/network/<pk>`) (this is the case when using ModelViewSet, or GenericAPIView with RetrieveMixins/UpdateMixins/DeleteMixins):
           - to use a field other than `pk` in model as primary key, override _lookup_field_ attribute of the view class
           - to use a kwargs other than `pk` in URL to get primary key value, override _lookup_url_kwargs_  attribute of the view class
         - override _get_queryset_ method in view class, in case we need some custom filtering or different queryset depend on request (authenticated user, URL, query params, etc)
         - to use custom method when operate with single object (for example, with multiple key), override _get_object_ method of the view class

      3. To define serializer class that will be use to serialize data set in response (in order):
         - override _serializer_class_ attribute
         - override _get_serializer_class_ method, in case we need more dynamic (for example, different serializer depend on request)

      4. To provide additional context for serializer (other than _view_ and _request_ objects):
         - override _get_serializer_context_ method

      5. To use (custom) pagination:
         - override _pagination_backend_ attribute to use available pagination classes

      6. To implement create action (in order)
         - inherit CreateModelMixin (either directly together with GenericAPIView, or indirectly from available _ModelViewSet_, _CreateAPIView_) and customization shall be done using available API
           1. To add some side effect or provide extra value to serializer, override _perform_create_ method
           2. To add some custom header in success response, override _get_success_header_ method
           3. View shall not alter the response body from serializer
         - in case of using _ModelViewSet_ or available _views_ that inherit _CreateModelMixin_:
           - override _create_ method
         - in case of using _APIView_, or _GenericAPIView_ without _CreateModelMixin_:
           - override _post_ method

      7. To implement update action (in order)
         - inherit UpdateModelMixin (either directly together with GenericAPIView, or indirectly from available _ModelViewSet, _UpdateAPIView_) and customization shall be done using available API
           1. To add some side effect or provide extra value to serializer, override _perform_update_ method
         - in case of using _ModelViewSet_ or available _views_ that inherit _UpdateModelMixin_:
           - override _update_ method
         - in case of using _APIView_, or _GenericAPIView_ without _CreateModelMixin_:
           - override _put_ method for full update, and _patch_ method for partial update

      8. To implement delete action (in order)
         - inherit DestroyModelMixin (either directly together with GenericAPIView, or indirectly from available _ModelViewSet, _DestroyAPIView_) and customization shall be done using available API
           1. to use different instance method to perform delete, or provide more paremeter to instance.delete , override _perform_destroy_ method
         - in case of using _ModelViewSet_ or available _views_ that inherit _DestroyModelMixin_:
           - override _destroy_ method
         - in case of using _APIView_, or _GenericAPIView_ without _DestroyModelMixin_:
           - override _delete_ method