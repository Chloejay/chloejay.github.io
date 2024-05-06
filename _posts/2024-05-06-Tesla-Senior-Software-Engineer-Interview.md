---
layout: post
title:  "Tesla interview for senior software engineer role"
date:   2024-05-06 21:00:00 +0800
categories: tech interview
tags: FastAPI, system design
image: api_design.png
applause: true
short_description: writing
--- 


<div markdown="1" id="text">
It's been two more month journey for my current interview with Tesla Berlin, for senior software engineer role. I received the initial interview call from this end of February, and conducted interviews with HR, HM for different rounds till this April 20th. Each round takes 2 more weeks and each round I need to sign a new NDA with different HR interview coordinators. The interview was cancelled just before the last technical interview I will conduct with another HM, as this interview was confirmed one week ago. I have daily practice DSA, but it made sense from the email I received from HR that the current hiring is freeze, due to Tesla layoff situation.

However, to move on and continue my dream chasing in Berlin, I will come to the round two of job search. It is a nervous, but an exciting journey for sure, for when I have the courage to put myself in a new space, what I got is full freedom to chase what my heart tells me, what my unsettlement guides me and what a new beautiful story is in front of me. For I officially start learn Deutsch from language school now.

So its time to backpack the previous round, now I wrap up what I learnt from previous interview with Tesla, for its an learning space there.

Tesla has its own internal database of interview questions, which is share across different countries. For the question I have been tested is an RESTful API design. For the NDA I have signed, the question I illustrate is not exactingly same as interview one, below is my answer and thinking process.

Question: how to design a shortened URL using FastAPI and Python?

example:
```bash
Request:
POST / {"long_url":"https:example/long-url-path"}
Response:
{"short_url": "https://example/abcdef"}

# then once user GET, aka type the shorten URL in browser, it will redirect to the long-url
Request: 
GET /abcdef 
Response:
Redirect to https:example/long-url-path
```

To design this feature, clearly understand the functional requirements and divide into two steps,
1. Scenario one: When user has a long URL, have to create a short URL;
2. Scenario two: When user given a short URL, have to redirect to the original long URL.

To design this HTTP call in a service layer which use FastAPI, below are few questions need to ask myself before design:
1. How many shorten size is allowed, what is data type? string or alphanumeric?
2. Which kind of hash or algorithm can be used?
3. What the API call exception handling?
4. Where to store this created shorten URL? in memory or in database?
5. How about cache layer? (I remember I answered Redis on this design)


```Python
from fastapi import FastAPI, status, HTTPException, Response 
from fastapi.responses import RedirectResponse, JSONResponse
from pydantic import BaseModel, HttpUrl 

import random 
import string 

app= FastAPI() 

class URL(BaseModel):
    short_url: str 

class Long_URL(BaseModel):
    long_url: HttpUrl 

short_url_dict={} #map the short_url and long_url
BASE_URL= "https:example"

class URLShortener:
    @staticmethod
    def _get_random_char(long_url: str)-> str:
        #hash algorithm?
        random_char= "".join(random.choices(string.ascii_letters+ string.digits, k=5))

        return random_char

    @staticmethod
    def get_key(dict_: dict, short_url_val: str)-> str:
        for long_url_key, short_url_value in dict_.items():
            if short_url_value== short_url_val:
                return long_url_key

    @app.post("/", response_model= URL, status_code= status.HTTP_201_CREATED)
    def post(self, long_url: str)-> str:
        """
        Request:
        POST / {"long_url":"https:example/long-url-path"}
        Response:
        {"short_url": "https://example/abcdef"}
        """
        # how to handle the long_url is already existed?
        # if long_url is already existed, design decision: then will return existing short_url or can return status code 409, for request conflict, or return status code 400 if request is invalid. 
        if long_url in short_url_dict.keys():
            raise HTTPException(status_code= status.HTTP_409_CONFLICT, detail= "Long URL is already existed.")
            #return long_url_dict[long_url]
        if not long_url:
            raise HTTPException(status_code= status.HTTP_400_BAD_REQUEST, detail="Invalid request")
        else:
            random_char= URLShortener._get_random_char(long_url)    
            short_url= f"{BASE_URL}/{random_char}"
            short_url_dict[long_url]= short_url 
            # will use redirect here?
            return short_url


    @app.get("/{short_url}")
    def redirect(self, short_url: str):
        long_url= URLShortener.get_key(short_url)
        if short_url not in short_url_dict.values():
            raise HTTPException(status_code= status.HTTP_404_NOT_FOUND, detail="Short url is not found")
        else:
            return RedirectResponse(url= long_url)
```

Besides, for the random character, can we use the hashlib or will be this method we choose will cause the collision?

```python
import hashlib 
hash_object= hashlib.sha256(long_url.encode())
random_char= hash_object.hexdigest()[:5]
```

Last but not least, how to expand this question in an system design perspective? For actually this is a classic real case design scenario and it's always interesting to discuss more with HM when time is allowed. Even I have passed the above shorter URL question with test cases. So for this feature, more to expand is how to design its data layer and cache layer, load balancer and web server, SQL and NoSQL database selection.
</div>