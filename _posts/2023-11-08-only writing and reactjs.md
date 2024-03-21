---
layout: post
title:  "Reactjs in Django project"
date:   2023-11-08 18:20:00 +0800
categories: reactjs
tags: reactjs
image: joy_moment.jpg
applause: true
short_description: writing, Reactjs, MUI
--- 


<div markdown="1" id="text">
I haven't really written any words for one more year, but I learnt a life lesson and Singapore really teaches me a lot in the subject of life, which is a missed course to me. Move on, to this writing, which is the Reactjs project I'm working on, part of my Django project, which is an interesting experience that combines these two tech stacks in one full stack project, I did feel joy here and much learning, along the way. Who said life is not then?

First two questions why use Reactjs not `jinja templating` that offers by Django or use `htmx`, the new javascript language wrapped for Django. I think it comes from the project requirement and scale, also which tool you as the developer, are comfortable with in work context. Though I prefer to choose something I'm not comfortable with as a new learning and not get bored.

Have used Reactjs in 2016 to try build the portfolio, but it ended nowhere, just a website. After 5 more years in Python, which is more OOP, I feel lucky to have the chance(time) to practice Reactjs in my current work.

For the setup, I use Webpack and Babel to bundle Reactjs JXS files. React is component based design, which is quite intuitive and reusable component is more FP style. I recently used often is `useState` and `useContext` React hooks, which required the components designing in the first place, aka. <b>abstract components</b>.

For web application I'm building is for the risk management, which has functionalities that allow use to create customized report structures contains main components: Scenario, Questions, Templates, Search Bar (API based), Report.

These required to design the `eventful components` that based on user behaviors, such as click, then the state of clicked Templates will be the shared state with other components, such as Questions, Search Bar and Report.

I will give the <b>high level code example</b> that I wrote, but not the same code I wrote in company for which is not allowed.

#### Component Design


```js 
import react, { useState, useContext, createContext, useEffect } from React; 


const SelectedItemContext = createContext();
function SelectTemplateWrapper() {
    const [selectedItem, setSelectedItem] = useState([]);
    // other states variables; 

    // user click 
    const handleClick = (e) => {
        setSelectedItem(e.target.value);
    }

    // retrieve data via API call;
    
    useEffect(() => {
        // api fetch call 
        fetch(API_URI, {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(data)
        })
            .then(res => res.json())
            .then(data => {
                setSelectedItem(data.item)
            })
            .catch(error => {
                console.error(error)
            });
    }, []);

    return (
        <>
            ... 
            <SelectedItemContext.Provider value={selectedItem}>
                <APIComponentWrapper/>
                <ReportComponentWrapper/>
            </SelectedItemContext.Provider>
        </>
    )
};

// for the searchBar component and API one, which extract one level higher
// use APIComponentWrapper;
function APIComponentWrapper(){
    const selectedItem= useContext(SelectedItemContext);
    ...
    return null;
}


function ReportComponentWrapper(){
    const selectedItem= useContext(SelectedItemContext);
    ...
    return null;
}
```

#### Thoughts
Abstract is interesting, and I think I will explore more with reactjs and this new project, I will share on next article.
<hr>
News to update: 
I always feel I can't connect with Singapore, for the culture part, but then I asked myself why not I try to build the culture I want to be? So finally, I will organize Python events here under the <b>Pyladies</b>, that bring the community here to share and code. 
Community is core culture of coding and important to me, be it a support for any new learner, be it a corner for people to share, be it a space for people to learn and grow, and share good time when coding together. I'm quite excited for the new community that I bring here. 
<br>
<small><b>Spolier: pic I took in Python Europe </b></small><br>
<img src="/assets/pycon_prague_2023.jpg" width="450" >
</div>