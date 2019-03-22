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
              handlers.py
              models.py
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

   3. In case of using model based viewsets or generics view, customization shall done using their available API as long as possible
      1. To apply custom authentication method (other than specified in `REST_FRAMEWORK.DEFAULT_AUTHENTICATION_CLASSES` settings):
         - use custom authentication classes in _authentication_classes_ attribute of the view class

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

           