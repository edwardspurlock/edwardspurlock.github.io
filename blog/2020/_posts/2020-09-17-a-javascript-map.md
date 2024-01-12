---
layout: posts
title: A JavaScript Map
date: 2020-09-17 11:05
author: edward
categories: blog Google
slug: a-javascript-map
status: published
---



## My first home:





::: {#map style="width: 100%; height: 400px;"}
:::


<p>
```

<script><br />
function initMap(){<br />
const prest = {lat:42.357681, lng:-83.195785};<br />
let myMap = new google.maps.Map(<br />
   document.getElementById('map'), {zoom: 15, center: prest, mapId: "f15394c6ca8b38f9"});<br />
let marker = new google.maps.Marker({position: prest, map: myMap});<br />
}<br />
</script>
```
  


<script defer src="https://maps.googleapis.com/maps/api/js?key=AIzaSyAZChkw3BDtjjITJFWUfTMnLLl0TUsZmZk&amp;callback=initMap&amp;libraries=places"><br />
</script>
```
  



</p>
```



