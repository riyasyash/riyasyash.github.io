---
layout: post
title: "A Simple OAuth Provider"
description: "Recently, I had to implement an OAuth Provider for one of my projects (So that people can log in to a different application using their credentials on my app), which was using JWT token-based authentication."
date: 2018-10-07 00:00:00
comments: true
image: https://cdn-images-1.medium.com/max/1600/1*qatotphqCyOKDuGbDl4Qwg.jpeg
keywords: "oauth, python, django-rest-framework, django"
category: tech
tags:
- tech
---
Recently, I had to implement an OAuth Provider for one of my projects (So that people can log in to a different application using their credentials on my app), which was using JWT token-based authentication. So I read through a lot of documentation and implementation models. Fortunately, since we were using django-rest-framework I could find a lot of good writeups. This is one among them Oauthlib.

After reading the particular writeup, I decided to implement the provider our selves (since it seemed pretty simple, 😅and it turned out simple too!)

Here, we will be implementing OAuth through Authorization Code (Yeah, there are other methods as well.) The code will be more Django specific, but you can make use of the concepts and the models to implement the same in any other languages of your choice.

## How is it working?
<br>
<figure class="image"><center>
    <img src="https://cdn-images-1.medium.com/max/1600/1*HcZ1gZ_7SbeY6V5h6N13Pw.jpeg" alt="diagram depicting the authentication flow">
    <figcaption>{{ "diagram depicting the authentication flow" }}</figcaption>
</center>
  </figure>
<br>

1. The third party app (the application to which the user has to login) redirects to our authorization page with the client_id, that they get at the time of registration and optional success and error redirect URIs.<br><br>
2. Our authorization page will validate whether a user is currently logged in or not (by checking the cookies). If there is a logged in user the page will ask the user for authorizing the access, if not the page will ask to log in with credentials. After that, the page requests our application backend to generate an authorization code for the client for the logged in user.<br><br>
3. Our application backend ensures client and user validity and generates an authorization_code which expires in 10 minutes and responds with the code and redirect_uri (if any was registered with the client).<br><br>
4. The authorization page redirects to the redirect_uri obtained from the application backend (or the success URI passed as the query parameter) with the authorization code.<br><br>
5. The third party app requests our application backend to generate a user token with the obtained authorization_code, client_id, and client_secret obtained at the time of client registration.<br><br>
6. The application backend validates the credentials and authorization code and generates a user token for the third party app.

## Let’s Begin

We need to write the CRUD APIs to register a client, update details for a client and delete a client (I will not be covering them here🤓).

We need two models to store the OAuth related data

1. Client: Which will store the details of the third party application which has to access our app’s data.
2. Authorization Code: Which will store the authorization_code, the user and client details for whom the code was generated and the expiry time of the particular authorization_code.

We need to write functions to perform the following operations

1. To generate an authorization code after validating the client.
2. To generate the actual user token after validating the authorization code and client credentials
3. Optional function to invalidate the authorization code after generating the user token

## Let’s Look at the code

```python
class AuthorizationCode(models.Model):
    client = models.ForeignKey(Client)
    user = models.ForeignKey(User)
    code = models.CharField(max_length=100, unique=True)
    expires_at = models.DateTimeField()
    class Meta:
        managed = True
    def generate_user_token(self):
        try:
            user = self.user
            token = function_to_generate_token(user) // replace with the function to generate the token
            self.invalidate()
            return token
        except:
            return False

    def invalidate(self):
        self.expires_at = datetime.datetime.now(datetime.timezone.utc)
        self.save()
```

```python
class Client(models.Model):
    client_id = models.CharField(
        max_length=100, unique=True, default=generate_token, db_index=True
    )
    redirect_uri = models.TextField(
        blank=True
    )
    error_uri = models.TextField(
        blank=True
    )
    client_secret = models.CharField(
        max_length=255, blank=True, default=generate_client_secret, db_index=True
    )
    name = models.CharField(max_length=255, blank=True)
    class Meta:
        managed = True
    @classmethod
    def generate_auth_token(cls, client_id, client_secret, authorization_code):
        try:
            client = cls.objects.get(client_id=client_id, client_secret=client_secret)
            authorization_code = client.validate_auth_code(authorization_code)
            if authorization_code:
                return authorization_code.generate_user_token()
            return False
        except Exception as e:
            return False

    def validate_auth_code(self, authorization_code):
        authorization_code = self.authorizationcode_set.filter(code=authorization_code,
                                                                       expires_at__gte=datetime.datetime.now(
                                                                           datetime.timezone.utc)).first()
        return authorization_code
```

```python
def generate_authorization_code(request, user):
    client_id = request.data.get('client_id')
    client = Client.objects.get(client_id=client_id)
    if client:
        authorization_code_data = create_authorization_code(request)
        authorization_code_data.update({'user': user.id, 'client': client.id})
        authorization_code_serializer = AuthorizationCodeSerializer(data=authorization_code_data)
        authorization_code_serializer.is_valid()
        authorization_code = authorization_code_serializer.save()
        return Response({
            'redirect_uri': client.redirect_uri,
            'authorization_code': authorization_code.code
        })
def create_authorization_code(request):
    """Generates an authorization grant represented as a dictionary."""
    now = datetime.datetime.now(datetime.timezone.utc)
    expires_at = now+datetime.timedelta(minutes=10)
    grant = {'code': generate_token(),'expires_at':expires_at}
    if hasattr(request, 'state') and request.state:
        grant['state'] = request.state
    return
```

The functions generate_client_secret() and generate_token() can be any method that generates a random string with a length satisfying OAuth specification. The function_to_generate_token() has to be replaced with whatever you are using to generate user tokens (in my case it is JWT).

## That’s it!

Now you know how an OAuth provider can be implemented!.
