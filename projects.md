---
layout: project
title: Projects


subtitle: I write code in
speed: 50
strings: ["Python.","JavaScript.","Java.","C.", "MATLAB.", "HTML.", "CSS.",]
backdelay: 1000
---

<div>
  
  {% for project in site.projects reversed%}
    
    <a href="{{ project.url | prepend: site.baseurl }}">
      <div class="col-xs-12 col-sm-12 col-md-12">
        <div class="card">
          <img class="img-responsive" src="{{project.card-img}}">
          <div class="card-block">
                {{ project.title }}
          </div>
        </div>
      </div>
    </a>

  {% endfor %}

</div>