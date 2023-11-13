---
layout: post
title:  "Reactjs component - multi select"
date:   2023-11-13 18:40:00 +0800
categories: reactjs
tags: reactjs
image: joy_moment.jpg
applause: true
short_description: writing, reactjs
--- 


<div markdown="1" id="text">
Reactjs is truly FP style, to adjust the new business logic I have to redesign the codebase, which made it more module and new changes is to place the logic in the components level. During the weekend, I wanted to implement <b>TF-IDF algorithm</b> in the search component, which backend I will use `Haystack` library, while to achieve the same result I can implement this logic via `react-select` library. 
<br/>
To achieve I will still use the `DRF` to build the communication bridge between the client and server. Still I will illustrate with the high level code without details, abstractly. 

Generic business logic, in the report page, UI component `Search/Select` will be dynamic switched, once user click component `Template`, which talked in previous articles with shared states. I will focus on `multi select` component on this article. 


#### Backend API call 
```python
from rest_framework import generics 
from django.http import JsonResponse

class CountryAPIView(generics.ListAPIView):
    def get(self, request):
        try:
            db_conn=DB().conn() 
            data_query= """ SELECT * FROM ...
            """
            data= pd.read_sql(data_query, db_conn)

            response= {"data": data}

            return Response(response)

        except Exception as e:
            return Response({"message":"error"})

# then config the urls.py to for routing; skip here. 
```

#### Install libs
```shell 
npm install --save react-select
```

#### Frontend API call 
```javascript

import React from 'react'
import Select from 'react-select'
import {useState} from "react";
import {CircularProgress} from "material-ui/core";
import Typography from "@mui/material/Typography";
import FormLabel from "@mui/material/FormLabel";
import FormControlLabel from "@mui/material/FormControlLabel";
import Radio from "@mui/material/Radio";
import RadioGroup from "@mui/material/RadioGroup";


export function CountryDropDown(){
    const [ctryList, setCtryList]= useState([]);
    const [loading, setLoading]= useState(false);
    const URL= "api/...";

    const handleChange= (selectedOptions)=>{
        console.log(JSON.stringify(selectedOptions))
    }

    if (loading){
        return (<CircularProgress/>)
    }

    const newArray= (data)=> Objects.values(data.data.c).map(val, index)=>{
        return {
            "value": val, 
            "label": val
        }
    }

    useEffect(()=>{
        try{
            fetch(URL)
            .then((res)=>{
                if (res.status>= 200 && res.status<400){
                    setLoading(false);
                    return res.json()
                }else(res.status)
            })
            .then((data)=> {
                setCtryList(newArray(data))
            })catch(err){
                console.log(err)
            }
        }
    }, [])

    return (
        <>
        <Typography variant="h6"> 
            <FormControl component="fieldset">
            <FormLabel component="legend"> Search: </FormLabel>
                <RadioGroup
                row 
                >
                    <FormControlLabel 
                    value={searchBy}
                    control={<Radio checked={true} />}
                    label={searchBy}} />
                </RadioGroup>
            </FormControl>
        </Typography>
       
        <Select 
        styles={{...}} {/* skip...*/}
        options={ctryList}
        closeMenuSelect={false}
        defaultOptions
        isMulti
        onchange= {handleChange}
        />
         </>
    )
}
```

Below is the select component, which is quite neat. Writing to me is just the free style exercise, never plan, more spantaneous, so for the reactjs component design, I will share on next articles, for it deservers some standalone space. 

</div>