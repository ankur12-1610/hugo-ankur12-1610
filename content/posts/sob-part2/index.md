---
title: "Access tokens for Specter's REST API: Part 2 | Summer of Bitcoin '22"
read_time: true
date: 2022-08-24
cover:
    image: "https://cdn.hashnode.com/res/hashnode/image/upload/v1661357884206/clyKLsXzY.png?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "sob-part2"
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

This blog covers the work that I've done post mid-term evaluation. In the last part, I implemented the token generation part which included generating and saving the token in the `users.json` file, which acts as the local database of the user.

### Implementation

Some things took a considerable amount of time and energy, as they needed to be discussed with the mentors.

#### Change in plans

<iframe src="https://giphy.com/embed/l3q2waulCQqmLNY3K" width="300" height="300" class="giphy-embed"></iframe>

Initially, the data was stored in individual entities, which resulted in creating several hashmaps which weren't necessary. After discussing with Kim and Manolis, we concluded that all the information related to the JWT tokens should be stored in a nested hashmap instead of storing it in separate entities. Storing it in separate entities created unnecessary crowding of variables and a nested hashmap was perfect to tackle this problem.

The fields related to the JWT token that needed to be stored:

![jwt-fields.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661089000351/t30ABhBO4.png )

* Earlier, the data was stored in the format:
    

![json-before.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661093571428/CoC17nm2R.png?height=300 )

* Later, the data was stored in the format of nested hashmap:
    

![json-after.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661093416372/fluUiPHfz.png )

#### Token Authentication

Implementing Token Authentication was the most important and challenging part of this project. Specter used [`Flask-HTTPAuth`](https://flask-httpauth.readthedocs.io/en/latest/index.html) for their Basic Authentication, which I was not entirely familiar with ;')

After reading the documentation and experimenting with the Token Authentication part in a small project, I cleared my basic concepts and hoped on the project's codebase.

The first step was to decode the JWT token to get the payload, and from that payload, I grepped the `username` of the user.

The next step was to get the user from username with the help of Specter's `user_manager` and then store it in the global user.

The final check was to add `jwt.ExpiredSignatureError`, to check whether the token is expired or not and `jwt.InvalidTokenError`, to check whether the token is in a valid format or not.

![token-auth.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661254215989/sfft5Y8a4.png )

#### Time Parser

The expiration time in the payload while encoding the token could only be set to either seconds or minutes or days or any time unit. The problem was that, if the standard time is set to seconds and the user wants his token to be valid for 3 days, he might need to convert 3 days to seconds.

In order to avoid this, I used `pytimeparser`, which basically converts the string into seconds. For eg: If the user sets the time to `3 days` then the parser will convert it to seconds (3 days = 259200 seconds) and then it can be easily stored in the database. Similarly, the user can set the token life to `3 minutes`, `3 weeks`, `3 months` and other time units as well.

> Note: For more information regarding `pytimeparser` visit: https://pypi.org/project/pytimeparse/

#### Helper Functions

Building these functions made the implementation a lot easier, once written they can be used anywhere and one helper function helped me build other helper functions.

For example:  
The helper functions for saving a newly created token to the database and deleting the token from the database.

```plaintext
def add_jwt_token(
    self, jwt_token_id, jwt_token, jwt_token_description, jwt_token_life
):
    # Adding a newly created JWT to the hashmap
    self.jwt_tokens[jwt_token_id] = {}
    self.jwt_tokens[jwt_token_id]["jwt_token"] = jwt_token
    self.jwt_tokens[jwt_token_id]["jwt_token_description"] = jwt_token_description
    self.jwt_tokens[jwt_token_id]["jwt_token_life"] = jwt_token_life
    self.save_info()

def delete_jwt_token(self, jwt_token_id):
    # Deleting a JWT from the hashmap
    if jwt_token_id in self.jwt_tokens:
        del self.jwt_tokens[jwt_token_id]
        self.save_info()
```

### Endpoints

* `/v1alpha/token/`: Creating new tokens and fetching all of them.
    

\*\* Method \*\*: `POST`

**Code**: `201 Created`

**Request Body**:

```plaintext
{
    "jwt_token_description": "Token alpha",
    "jwt_token_life": "6 minutes",
}
```

**Success Response**:

```plaintext
{
    "message": "Token generated",
    "jwt_token_id": "2bc0160d-edf4-4ab6-9801-52d185f65b59",
    "jwt_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiand0X3Rva2VuX2lkIjoiMmJjMDE2MGQtZWRmNC00YWI2LTk4MDEtNTJkMTg1ZjY1YjU5Iiwiand0X3Rva2VuX2Rlc2NyaXB0aW9uIjoiVG9rZW4gYWxwaGEiLCJleHAiOjE2NjEzMTg3NjIsImlhdCI6MTY2MTMxODQwMn0.MWbUuZ8BS5L9kryi_zG_PNJ4ud84mPuJhsqhiH_U5Rc",
    "jwt_token_description": "Token alpha",
    "jwt_token_life": 360
}
```

**Method**: `GET`

**Code**: `200 OK`

**Success Response**:

```plaintext
{
    "message": "Tokens exists",
    "jwt_tokens": {
        "edd6a8f7-47d8-405e-97a8-2e4e26d0fbeb": {
            "jwt_token_description": "Token 1",
            "jwt_token_life": 480,
            "jwt_token_remaining_life": 0
        },
        "2bc0160d-edf4-4ab6-9801-52d185f65b59": {
            "jwt_token_description": "Token alpha",
            "jwt_token_life": 360,
            "jwt_token_remaining_life": 232.19542360305786
        }
    }
}
```

* `/v1alpha/token/<token_id>`: Fetching and deleting a particular token by Id.
    

**Method**: `GET`

**Code**: `200 OK`

**Success Response**:

```plaintext
{
    "message": "Tokens exists",
    "jwt_token_description": "Token alpha",
    "jwt_token_life": 360,
    "jwt_token_life_remaining": 232.19542360305786,
    "expiry_status": "Valid"
}
```

**Method**: `DELETE`

**Code**: `200 OK`

**Success Response**:

```plaintext
{
    "message": "Token deleted"
}
```

### Project Details:

> For more info, visit the PR related to this project: https://github.com/cryptoadvance/specter-desktop/pull/1785

### Future Idea:

<iframe src="https://giphy.com/embed/ZZkCo8zKWtt2ZgozfX" width="400" height="300" class="giphy-embed"></iframe>

This implementation of \`JWT Token-Based Authentication\` is fairly straightforward and clearly helps improve the security of Specter's API.

The following things can be added to the project:

* creating a UI for it
    
* adding one more property to tokens, namely `endpoints` which will store the endpoints the user can access
    

### Learning:

<iframe src="https://giphy.com/embed/fhAwk4DnqNgw8" width="350" height="210" class="giphy-embed"></iframe>

\- Through this journey, I learned a lot about how open source projects work, getting involved with the community, and contributing to the projects. - The major learning for me was how familiar I got with \`Flask-RESTful\` framework and \`Flask-HTTPAuth\`.

### Special Thanks

<iframe src="https://giphy.com/embed/3o6Zt6KHxJTbXCnSvu" width="350" height="200" class="giphy-embed"></iframe>

It was an awesome experience for me to work on this project. I learned a lot of new things, Kim and Manolis really helped me whenever I got stuck.

I would also like to give Adi Shankara and Summer of Bitcoin for this amazing learning experience. I was able to develop great skills and valuable experience. I look forward to contributing to the Bitcoin space.

Also, super thanks for the Ledger Nano S and an awesome Swag Kit ;) ment

### Conclusion:

<iframe src="https://giphy.com/embed/3o6Mb5M2dmWNNzrSE0" width="300" height="270" class="giphy-embed"></iframe>

Thank you for reading, hope you enjoyed it! I'll continue to update my progress via the series of blogs ;)

Follow me on [Twitter](https://twitter.com/ankurrap) | [LinkedIn](https://www.linkedin.com/in/ankur-patil-a112a3202/) for more web development-related tips and posts.

That's all for today! You have read the article till the end.