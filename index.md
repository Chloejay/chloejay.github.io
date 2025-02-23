---
layout: page
permalink: /
description: "I'm a self-taught programmer and lifelong learner. I like writing code, listening to folk and dreamy music, drinking coffee, and commit to biking."
---

<div markdown="1" class="about">


 <div id="intro">

  <div class="bg">
   <img src="/assets/chloe_profile.jpeg" alt="{{ site.author }} profile pic" class="profile-pic" />
   <h2 id="name">Chloe Ji</h2>
   <div id="adj_">a {passionate, curious} Software Engineer</div>
   <p id="self_intro">I strive to write clean, functional code.
   Code in <span id="lang">Python (mostly), Golang, Scala, Javascript</span>,
   contributing to open source projects.
   I
   <a class="intro_a" href="https://github.com/Chloejay/anti-tldr">read</a>,
   <a class="intro_a" href="https://chloejay.github.io/blog/">write</a>, 
   <a class="intro_a" href="https://www.lewagon.com/blog/shanghai-data-science-teaching-crew?from=timeline&isappinstalled=0">teach</a> 
   and biking.
   
   <span id="closing"> As a self-taught, I keep updated myself with new tech stack.</span>
   </p>
  </div>

 </div>

  <hr>

  <div id="project_section">
    <h5 class="section_header">Projects</h5>
    <div class="proejcts">
      <ul class = "project_ul">
        <li class="project_li">  
          <img class="projectImage" src="/assets/image_detection_api.png">
          <div class="content">
          <div class="hover-content">
            Coca Cola Image Detection (Machine Learning).<br/>
          </div>
            <p class="hover-content-detailed"> 
              Built object detection model (TensorFlow)<br/>
              deliver model result: creating image/content mismatch dashboard (PowerBI)<br/>
              Skills: Python, Computer Vision, AWS S3, MySQL.<br/>
              <a href="https://github.com/Chloejay/vision"><button class="demo_button">Demo</button></a>
            </p>
          </div>
        </li>
        <li class="project_li"> 
          <img class="projectImage" src="/assets/blockchain.png">
          <div class="content">
            <div class="hover-content">Blockchain Data Analytics Visualization product.<br/></div>
            <p class="hover-content-detailed">Created big data pipeline, set up DWH (NoSQL and SQL), <br/>
            designed analytical layer to build real time dashboard using BI tool Apache Superset,<br/>
            deploy to AWS EC2 (Nginx).<br/>
            <a href="https://github.com/Chloejay/superset_nginx"><button class="demo_button">Demo</button></a></p>
          </div>
        </li>
        <a id="more" href="https://github.com/Chloejay">More</a>.
      </ul>
      </div>
  </div>

  <hr>
  <h5 class="section_header">Where to find me?</h5>

  <span class="contact_me">
  You can find me on [Twitter][twitter], [GitHub][github], [Medium][medium] or [send me an email](mailto:{{ site.email }}).
  </span>
  
  [twitter]: https://twitter.com/{{ site.twitter_username }}

  [github]: https://github.com/{{ site.github_username }}

  [medium]: https://medium.com/@{{ site.medium_username}}

  [linkedin]: https://www.linkedin.com/in/{{site.linkedin_username}}

</div>