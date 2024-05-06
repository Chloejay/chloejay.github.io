---
layout: post
title:  "Tesla interview for senior software engineer role"
date:   2024-05-06 21:00:00 +0800
categories: tech interview
tags: FastAPI, system design
image: 
applause: true
short_description: writing
--- 


<div markdown="1" id="text">
It's been two more month journey for my current interview with Tesla Berlin, for senior software engineer role. I received the initial interview call from this end of Febuary, and conducted interviews with HR, HM for different rounds till this April 20th. Each round takes 2 more weeks and each round I need to sign a new NDA with different HR interview coordinators. The interview was cancelled just before the last technical interview I will conduct with another HM, as this interview was confirmed one week ago. I have daily practice DSA, but it made sense from the email I received from HR that the current hiring is freeze, due to Tesla layoff situation. 

However, to move on and continue my dream chasing in Berlin, I will come to the round two of job search. It is a neverous, but an exciting journey for sure, for when I have the courage to put myself in a new space, what I got is full freedom to chase what my heart tells me, what my unsettlement guides me and what a new beautiful story is in front of me.

So its time to backpack the previous round, now I wrap up what I learnt from previous interview with Tesla.

Tesla has its own internal database of interview questions, which is share across different countries. For the question I have been tested is an RESTful API design question. For the NDA I have signed, the question will not exactingly same as interview one, below is my answer and thinking process.

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

To design this feature, clearly understand the requirements into two steps,
1. When user has a long URL, have to create a short URL;
2. When user given a short URL, have to redirect to the original long URL. 

To design this HTTP call in a service layer which use FastAPI, below are few questions need to ask myself before design:
1. how many shorten size is allowed, what is data type? string or alphanumeric?
2. which kind of hash or algorithm can be used?
3. what the API call exception handling?
4. where to store this created shorten URL? in memory or in database?
5. how about cache layer? (I remember I answered Redis on this design)


```Python
from fastapi import FastAPI, status, HTTPException, Response 
from fastapi.responses import RedirectResponse 
from fastapi.responses import JSONResponse
from pydantic import BaseModel, HttpUrl 
import random 
import hashlib 
import string 

app= FastAPI() 

class URL(BaseModel):
    short_url: str 

class Long_URL(BaseModel):
    long_url: HttpUrl 

short_url_dict={} #map the short_url and long_url
BASE_URL= "https:example"

@app.post("/", response_model= URL, status_code= status.HTTP_201_CREATED)
def post(query: dict):
    """
    Request:
    POST / {"long_url":"https:example/long-url-path"}
    Response:
    {"short_url": "https://example/abcdef"}
    """

    pass 


def redirect():
    pass 

```


And how to expand this question in an system design perspective? It's always interesting to discuss more with HM when time is allowed. Even we have passed the above shorter URL question test cases. So for this feature, more to expand is how to design its data layer and cache layer, loader balancer and web server. 

</div>