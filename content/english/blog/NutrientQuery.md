---
title: "Building a Cutting-Edge Nutrition Facts Web Application: Unleashing the Power of API Integration"
meta_title: ""
description: ""
date: 2023-12-13T05:00:00Z
image: "images/gallery/api.png"
categories: ["Web Aplications"]
author: "Asiel Elaouare"
tags: ["API", "JQuery", "HTML/CSS", "Bootstrap"]
draft: false  
---

{{< toc >}}

## NutrientQuery 

Welcome to the beginning of my nutrition facts [Web Aplication](https://stupendous-bombolone-7c3480.netlify.app "NutrientQuery Page") journey! It all started with a chat with a talented UX designer, Dennis, deeply immersed in a personal project for 5 months. While crafting his grand vision, he discovered a crucial feature essential for his project's success.

My web application's story began with a prototype test for his project, revealing the need for a nutrition-focused function. As my friend Dennis continues to build his masterpiece, this functionality has become the heart of his journey to healthier living. Join us in exploring the tale of collaboration, innovation, and creating a web application rooted in practicality.

My friend, the UX designer, is a coding newcomer, teaching himself the ropes. Despite being a beginner, his dedication has been crucial. Venturing into public APIs for his functionality prototype, his fresh perspective and eagerness to learn became a driving force. Innovation knows no boundaries, especially for those taking on the exciting challenges of coding for the first time.

Starting with API instructions, before the integration of the API into his project I created a variant some days before, using Bootstrap as my frontend and JQuery for the Document Object Model. This decision not only showcased the efficiency of public APIs but also marked the beginning of us working together to bring the UX designer’s vision to life through coding.


{{< notice "note" >}}
In this blog post, I'm explaining the development steps of my prototype variant.
{{< /notice >}}

## API Provider: The Essential Gateway to Nutritional Data


With the API documentation in hand, the next phase beckoned—the hands-on development of the functionality that would breathe life into this web application variant. Armed with the structured code provided by the API documentation, we embarked on a journey of implementation, weaving together the intricate threads of code and logic to create a seamless user experience.


```javascript
var query = '3lb carrots and a chicken sandwich'
$.ajax({
    method: 'GET',
    url: 'https://api.calorieninjas.com/v1/nutrition?query=' + query,
    headers: { 'X-Api-Key': 'YOUR_API_KEY'},
    contentType: 'application/json',
    success: function(result) {
        console.log(result);
    },
    error: function ajaxError(jqXHR) {
        console.error('Error: ', jqXHR.responseText);
    }
});
```

The dynamic nature of the API query request added another layer of flexibility to our project. This dynamism meant that the need for extensive error handling for users was minimized, making the user experience smoother. This adaptability not only simplified the development process but also contributed to a more user-friendly application, allowing us to focus on refining the core functionality without getting bogged down in unnecessary error management complexities.

Here is how the response of `10oz onion and a tomato` looked like from the API:

```javascript
{
  "items": [
    {
      "sugar_g": 13.3,
      "fiber_g": 4,
      "serving_size_g": 283.495,
      "sodium_mg": 8,
      "name": "onion",
      "potassium_mg": 99,
      "fat_saturated_g": 0.1,
      "fat_total_g": 0.5,
      "calories": 126.7,
      "cholesterol_mg": 0,
      "protein_g": 3.9,
      "carbohydrates_total_g": 28.6
    },
    {
      "sugar_g": 2.6,
      "fiber_g": 1.2,
      "serving_size_g": 100,
      "sodium_mg": 4,
      "name": "tomato",
      "potassium_mg": 23,
      "fat_saturated_g": 0,
      "fat_total_g": 0.2,
      "calories": 18.2,
      "cholesterol_mg": 0,
      "protein_g": 0.9,
      "carbohydrates_total_g": 3.9
    }
  ]
}
```

The provided data is formatted in JSON (JavaScript Object Notation), which is a lightweight data interchange format commonly used in web development and APIs. In technical terms:

The entire structure is enclosed in curly braces `{}` denoting a JSON object.
Within this object, there is an array labeled `items` that begins with square brackets `[]`. Each element inside the array is a JSON object containing nutritional information for a specific food item.
Each food item is represented as a JSON object with key-value pairs, where the keys are attribute names (e.g., "sugar_g," "fiber_g") and the values are the corresponding data for each attribute.

Working with the API was a smooth experience, facilitating seamless integration into this project. This efficiency allowed us to progress swiftly in developing the desired functionality for Dennis' web application.

The use of jQuery further simplified the process with clean code, offering a better overview of the Document Object Model (DOM). This contributed to an efficient and straightforward development experience.

The cornerstone of our functionality lies in the following pivotal code block. This JavaScript function, 'updateTableOfContent,' plays a central role in updating the content displayed on the web application.


```javascript
function updateTableOfContent(result){
    $('#result').empty()

    let totalGrams = 0;
    let totalCalories = 0;
    let totalFat = 0;
    let totalCholesterol = 0;
    let totalSodium = 0;
    let totalCarbs = 0;
    let totalFiber = 0;
    let totalSugar = 0;
    let totalProtein = 0;
    
    result.items.forEach(element => {
        const servingSize = parseFloat(element.serving_size_g);
        const calories = parseFloat(element.calories);
        const fat = parseFloat(element.fat_total_g);
        const cholesterol = parseFloat(element.cholesterol_mg);
        const sodium = parseFloat(element.sodium_mg);
        const carbs = parseFloat(element.carbohydrates_total_g);
        const fiber = parseFloat(element.fiber_g);
        const sugar = parseFloat(element.sugar_g);
        const protein = parseFloat(element.protein_g);
    
        totalGrams += servingSize;
        totalCalories += calories;
        totalFat += fat;
        totalCholesterol += cholesterol;
        totalSodium += sodium;
        totalCarbs += carbs;
        totalFiber += fiber;
        totalSugar += sugar;
        totalProtein += protein;
    });

    result.items.forEach(element => {
        $('#result').append(
            '<tr>' +
            '<td>' + element.name + '</td>' +
            '<td>' + element.serving_size_g + '</td>' +
            '<td>' + element.calories + '</td>' +
            '<td>' + element.fat_total_g + '</td>' +
            '<td>' + element.fat_saturated_g + '</td>' +
            '<td>' + element.cholesterol_mg + '</td>' +
            '<td>' + element.sodium_mg + '</td>' +
            '<td>' + element.carbohydrates_total_g + '</td>' +
            '<td>' + element.fiber_g + '</td>' +
            '<td>' + element.sugar_g + '</td>' +
            '<td>' + element.protein_g + '</td>' +
            '</tr>'
        );

        google.charts.load('current', {'packages':['corechart']});

        google.charts.setOnLoadCallback(drawChart);

        function drawChart() {
            var data = new google.visualization.DataTable();
            data.addColumn('string', 'name');
            data.addColumn('number', 'values');
            data.addRows([
                ['Calories', totalCalories],
                ['Fat', totalFat],
                ['Cholesterol', totalCholesterol],
                ['Sodium', totalSodium],
                ['Carbohydrates', totalCarbs],
                ['Fiber', totalFiber],
                ['Sugar', totalSugar],
                ['Protein', totalProtein],
            ]);
        
            var options = {};
        
            var chart = new google.visualization.PieChart(document.getElementById('chart_div'));
            chart.draw(data, options)
        }
    });
};
```
This function dynamically updates the content on the web page, calculating and displaying key nutritional information. It also generates a pie chart using Google Charts API to offer a visual representation of the overall nutritional composition. This block stands as the linchpin in ensuring a comprehensive and user-friendly presentation of the nutrition data.

## Frontend Choice: Building User-Friendly Experiences with Bootstrap

In the world of web development, choosing the right tools can significantly impact the success of a project. In my journey, I decided to go with Bootstrap as the frontend framework for this web application variant. Why Bootstrap? It's well-known for being fast and easy, and it offered me a way to skip the often time-consuming task of styling with CSS.

This decision wasn't just about making things easy—it was a strategic move to make the development process smoother. Bootstrap comes with a bunch of pre-designed elements and a flexible grid system, which meant I could concentrate more on the logic and functionality of the web application. It was like having a toolkit that made building things visually appealing and consistent a breeze.

In simpler terms, Bootstrap became my go-to because it promised to make things both easy and good-looking. It's like having a reliable sidekick—always there to support me in crafting a web app that not only works well but also looks polished.

## Building Bonds and Innovations through Collaborative Coding

This functionality not only enhanced our web application but also deepened my connection with my friend. As we collaborated on developing the final variant for his project, I gained valuable insights and learned new things from his perspective. The shared experience of building and refining this feature strengthened our teamwork and fostered a dynamic exchange of knowledge, making the development process not only efficient but also enriching.


<hr>

***Final result: https://stupendous-bombolone-7c3480.netlify.app***

[![GitHub](https://img.shields.io/badge/GitHub-Repository-blue.svg?style=for-the-badge&logo=github)](https://github.com/AsielElaouare/NutrientQuery)

<hr>

