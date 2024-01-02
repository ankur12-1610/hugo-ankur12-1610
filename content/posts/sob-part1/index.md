---
title: "Access tokens for Specter's REST API: Part 1 | Summer of Bitcoin '22"
read_time: true
date: 2022-07-03
cover:
    image: "https://cdn.hashnode.com/res/hashnode/image/upload/v1656862908396/J9knlsb49.png?w=1600&h=840&fit=fill&fill=blur&auto=compress,format&format=webp"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "sob-part1"
    relative: false # To use relative path for cover image, used in hugo Page-bundles
tags:
 - Summer of Bitcoin
 - Flask Framework
 - Bitcoin
 - REST API
 - Open Source
 - JWT
---

### Abstract

[Specter Desktop](https://github.com/cryptoadvance/specter-desktop) is a desktop GUI for Bitcoin Core optimized to work with hardware wallets.

![specter.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656859075053/CpG22i_7I.png )

Specter already has a REST API system, but the authorization is currently done by `HTTPBasicAuth` and to improve the security, `HTTPTokenAuth` is really necessary. **Access tokens** would be a significant improvement.

### My Project

![367ei5.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1656859887473/e9UqDBvPD.jpg?height=300 )

I have to use access tokens for the token-based authorization, the access token I opted for was [**JSON Web Token (JWT)**](https://jwt.io/) because:

* I have already used it
    
* I read their official documentation and got a fan of JWT and its advantages over Simple WebToken (SWT) and Security Assertion Markup Language Tokens (SAML). [source](https://jwt.io/introduction#:~:text=Why%20should%20we%20use%20JSON%20Web%20Tokens%3F)
    

> JWT authentication flow

**Expected Outcomes**:

* User will be able to create a JWT token when the request is sent to a particular endpoint of the api
    
* Once the token is generated the user can only see it once and the token needs to be stored somewhere in order to access sessions later on.
    
* Authorization will be based on `HTTPTokenAuth`
    

### Project Progress

<iframe src="https://giphy.com/embed/Tfk8R5KDPn3PxEUVaI" width="400" height="400" class="giphy-embed"></iframe>

#### Headstart 🚀

At first, when I went through Specter's codebase it was difficult for me to understand and I was not able to figure out where to make the changes, but thanks to my mentor ([k9ert](https://github.com/k9ert)) who helped me sort things out when I was stuck. He suggested making a small FLASK API in a similar structured way as Specter's API was made. This really helped me understand `Flask-RESTful` and `Flask_HTTPAuth`.

#### PoC Implementation 📖

**Step 1**

The very first step was to install `PyJWT`:

```plaintext
pip install PyJWT
```

Next, we need to create a 'jwt\_token' variable in the `UserMixin`of `src/cryptoadvance/specter/user.py`, then we pass it in the `user_dict` with the help of a property.

```plaintext
class User(UserMixin):

     def __init__(
        ...
        jwt_token, 
        ...
    ):
        ...
        self.jwt_token = jwt_token
        ...
    # TODO: User obj instantiation belongs in UserManager
    @classmethod
    def from_json(cls, user_dict, specter):
        try:
            user_args = {
                ...
                // Setting default value of jwt_token to None so that it can be stored in the user's data and then later on updated
                "jwt_token": user_dict.get("jwt_token", None),
                ...
            }
            if not user_dict["is_admin"]:
                user_args["config"] = user_dict["config"]
                return cls(**user_args)
            else:
                user_args["is_admin"] = True
                return cls(**user_args)

        except Exception as e:
            handle_exception(e)
            raise SpecterError(f"Unable to parse user JSON.:{e}")
    @property
    def json(self):
        user_dict = {
            ...
            "jwt_token": self.jwt_token,
            ...
        }
        if not self.is_admin:
            user_dict["config"] = self.config
        return user_dict
```

We also need to add **helper** functions in order to fetch and delete the token:

```plaintext
    def save_jwt_token(self, jwt_token):
        self.jwt_token = jwt_token
        self.save_info()

    def delete_jwt_token(
        self,
    ):
        self.jwt_token = None
        self.save_info()
```

**Step 2**

This step includes the creation of token based endpoints in the API. For this I created a new file namely `jwt.py` in the `rest` directory.

> Directory tree

```plaintext
import jwt
from flask import current_app as app
from cryptoadvance.specter.api.rest.base import (
    BaseResource,
    rest_resource,
    AdminResource,
)
import uuid
import datetime
import logging
from ...user import *
from .base import *

from .. import auth

logger = logging.getLogger(__name__)


def generate_jwt(user):
    payload = {
        "user": user.username,
        "exp": datetime.datetime.utcnow() + datetime.timedelta(days=1),
    }
    return jwt.encode(payload, app.config["SECRET_KEY"], algorithm="HS256")


@rest_resource
class TokenResource(AdminResource):
    endpoints = ["/v1alpha/token/"]

    def get(self):
        user = auth.current_user()
        user_details = app.specter.user_manager.get_user(user)
        jwt_token = user_details.jwt_token
        return_dict = {
            "username": user_details.username,
            "id": user_details.id,
            "jwt_token": jwt_token,
        }
        if jwt_token is None:
            return_dict["jwt_token"] = generate_jwt(user_details)
            jwt_token = return_dict["jwt_token"]
            user_details.save_jwt_token(jwt_token)
            return {
                "message": "Token generated",
                "username": user_details.username,
                "jwt_token": jwt_token,
            }
        return {"message": "Token already exists", "jwt_token": jwt_token}

    def delete(self):
        user = auth.current_user()
        user_details = app.specter.user_manager.get_user(user)
        jwt_token = user_details.jwt_token
        if jwt_token is None:
            return {"message": "Token does not exist"}
        user_details.delete_jwt_token()
        return {"message": "Token deleted"}
```

This basically includes the function `generate_jwt` which takes `user` as an argument and generates the token with the payload as `user.username` and the expiry date of the token (`exp`).

Then we have two endpoints - `GET` and `DELETE` which receive data from the `User` model using `user_manager` by passing the authenticated `user`.

> GET request functionality

> DELETE request functionality

**Last Step**

The last step is to register the endpoints in the API, this can be done by adding:

```plaintext
from .jwt import TokenResource
```

in `src/cryptoadvance/specter/api/rest/api.py`

> PR related to this project https://github.com/cryptoadvance/specter-desktop/pull/1785

---

> Demo of the implemented PoC:

**Future milestones**

<iframe src="https://giphy.com/embed/kit7OGuNpZQzDZRmrm" width="430" height="300" class="giphy-embed"></iframe>

The plans for the rest of my journey are:

* Adding a `verfiy_token` function that verifies if the given token is correct or not.
    
* Replacing `HTTPBasicAuth` with `HTTPTokenAuth`
    
* Add one-time view functionality for the users.
    

### Conclusion

<iframe src="https://giphy.com/embed/fMKLjY1O45PxN90eB1" width="400" height="400" class="giphy-embed"></iframe>

Thank you for reading, hope you enjoyed it! I'll continue to update my progress via the series of blogs ;)

Follow me on [Twitter](https://twitter.com/ankurrap) | [LinkedIn](https://www.linkedin.com/in/ankur-patil-a112a3202/) for more web development-related tips and posts.

That's all for today! You have read the article till the end.