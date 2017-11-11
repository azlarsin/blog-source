title: canvas - 学习笔记
author: azlar
date: '2016-11-07 11:30:48'
tags: [canvas, learn]

---

之前用的比较多的是 *svg*，一直想学习下 *canvas*。

<!-- desc -->

# 网站
> http://www.html5canvastutorials.com/tutorials/html5-canvas-element/

## lines
### line
> To draw a line using HTML5 Canvas, we can use the `beginPath()`, `moveTo()`, `lineTo()`, and `stroke()` methods.
> 
> - First, we can use the `beginPath()` method to declare that we are about to draw a new path.  
> 
> - Next, we can use the `moveTo()` method to position the context point (i.e. drawing cursor), and then use the `lineTo()` method to draw a straight line from the starting position to a new position.  
> 
> - Finally, to make the line visible, we can apply a stroke to the line using `stroke()`.  Unless otherwise specified, the stroke color is defaulted to black.

```javascript
var canvas = document.getElementById("canvas");

var context = canvas.getContext('2d');

// context.font = '40pt Calibri';
// context.fillStyle = 'blue';
// context.fillText('hello world!', 150, 100);

context.beginPath();
context.moveTo(100, 150);
context.lineTo(450, 50);
context.stroke();
```

### line width
> The `lineWidth` property must be set before calling `stroke()`.

```javascript
//...
context.lineWidth = 15;
context.stroke();
```

### line color
> To set the color of an HTML5 Canvas line, we can use the `strokeStyle` property of the canvas context, which can be set to a color string such as red, green, or blue, a hex value such as *#FF0000* or *#555*, or an RGB value such as *rgb(255, 0, 0)*.

```javascript
//...
context.strokeStyle = '#ff0000';
```

### line cap
给线条两边加上小帽。

> To add a cap to an HTML5 Canvas line, we can use the `lineCap` property. Lines can have one of three cap styles: `butt`, `round`, or `square`. Unless otherwise specified, HTML5 Canvas lines are **defaulted** with the `butt` cap style.  The `lineCap` property must be set before calling `stroke()`.

```javascript
#run: canvas
var canvas = document.getElementById('canvas');
var context = canvas.getContext('2d');

// butt line cap (top line)
context.beginPath();
context.moveTo(200, canvas.height / 2 - 50);
context.lineTo(canvas.width - 200, canvas.height / 2 - 50);
context.lineWidth = 20;
context.strokeStyle = '#0000ff';
context.lineCap = 'butt';
context.stroke();

// round line cap (middle line)
context.beginPath();
context.moveTo(200, canvas.height / 2);
context.lineTo(canvas.width - 200, canvas.height / 2);
context.lineWidth = 20;
context.strokeStyle = '#0000ff';
context.lineCap = 'round';
context.stroke();

// square line cap (bottom line)
context.beginPath();
context.moveTo(200, canvas.height / 2 + 50);
context.lineTo(canvas.width - 200, canvas.height / 2 + 50);
context.lineWidth = 20;
context.strokeStyle = '#0000ff';
context.lineCap = 'square';
context.stroke();
```

this code will draw:

![](//blog.azlar.cc/images/canvas/canvas-line-cap.png)

difference:

![](//blog.azlar.cc/images/canvas/canvas-line-cap-1.jpg)

## curves
### arc
> context.arc(x, y, radius, startAngle, endAngle, counterClockwise);
> 
> To create an arc with HTML5 Canvas, we can use the arc() method. Arcs are defined by a center point, a radius, a starting angle, an ending angle, and the drawing direction (either clockwise or anticlockwise).  Arcs can be styled with the `lineWidth`, `strokeStyle`, and `lineCap` properties.
> 
> ![](//blog.azlar.cc/images/canvas/html5-canvas-arcs-diagram.png)
> 
> An arc is nothing more than a section of the circumference of an imaginary circle. This imaginary circle can be defined by `x`, `y`, and `radius`.
>
> Next, we can define the arc itself with two points along the imaginary circle's circumference defined by `startAngle` and `endAngle`. These two angles are defined in radians and form imaginary lines that originate from the center of the circle and intersect the ends of the arc that we wish to create.
>
> The final argument of the arc method is `antiClockwise` which defines the direction of the arc path between its two ending points. Unless otherwise specified, this argument is defaulted to false, which causes the arc to be drawn clockwise.
>
> Note: Alternatively, we can also create an arc using the `arcTo()` method which is used for creating rounded corners in a path.

**`context.arc(x, y, radius, startAngle, endAngle, counterClockwise);`**

```javascript
#run: canvas
var canvas = document.getElementById('canvas');
var context = canvas.getContext('2d');
var x = canvas.width / 2;
var y = canvas.height / 2;
var radius = 75;
var startAngle = 1.1 * Math.PI;
var endAngle = 1.9 * Math.PI;
var counterClockwise = false;

context.beginPath();
context.arc(x, y, radius, startAngle, endAngle, counterClockwise);
context.lineWidth = 15;

// line color
context.strokeStyle = 'black';
context.stroke();
```


根据评论区是雨伞，画了一个眼镜。。。

```javascript
#run: canvas
var canvas = document.getElementById('canvas');
var ctx = canvas.getContext('2d');

//glasses
ctx.beginPath();
ctx.arc(125, 150, 25, 2 * Math.PI, 0 * Math.PI);
ctx.arc(195, 150, 25, 1 * Math.PI, -1 * Math.PI);
ctx.lineWidth = 2;
ctx.stroke();

//left frame
ctx.beginPath();
ctx.moveTo(100, 150);
ctx.lineTo(100, 100);
ctx.lineWidth = 2;
ctx.lineCap = 'round';
ctx.strokeStyle = "black";
ctx.stroke();

ctx.beginPath();
ctx.moveTo(100, 100);
ctx.lineTo(102, 100);
ctx.lineWidth = 2;
ctx.lineCap = 'round';
ctx.strokeStyle = "black";
ctx.stroke();

//right frame
ctx.beginPath();
ctx.moveTo(220, 150);
ctx.lineTo(220, 100);
ctx.lineWidth = 2;
ctx.strokeStyle = "black";
ctx.lineCap = 'round';
ctx.stroke();

ctx.beginPath();
ctx.moveTo(220, 100);
ctx.lineTo(218, 100);
ctx.lineWidth = 2;
ctx.lineCap = 'round';
ctx.strokeStyle = "black";
ctx.stroke();
```

### Quadratic Curve
> 
