---
title: Canvas의 기초 - 1
toc: true
widgets:
  - type: toc
    position: right
  - type: categories
    position: right
  - type: tags
    position: right
  - type: adsense
    position: right
    client_id: ca-pub-5445993070474035
    slot_id: ''
date: 2020-05-31 13:36:40
categories:
  - canvas
tags:
    - canvas
    - 선 그리기
    - 사각형 그리기
    - 원 그리기
    - gradient
    - pattern
    - drawImage
---

HTML 에서는 태그를 중심으로 화면에 글자, 그림 등을 배치하고 CSS 를 이용하여 레이아웃을 그리지만, 캔버스는
하나의 화면에 자바스크립트에서 지원하는 캔버스 함수를 이용하여 그린다.

캔버스의 기본적인 사용법은 소스와 같이 html 태그 안에 넣으면 된다. 캔버스 내에서는 css 가 제어되지 않기 때문에
자바스크립트를 이용하여 코드를 구성해야 한다.

<!-- more -->

```html
<!DOCTYPE html>
<html lang="ko">
    <head>
        <title>Canvas 01</title>
        <meta charset="UTF-8"/>
    </head>
    <body>
        <canvas id="myCanvas" width="400" height="300">CANVAS를 지원하지 않습니다.</canvas>
        <script></script>
    </body>
</html>

```

만약 캔버스가 지원되지 않는 브라우저이면, 캔버스 태그 안의 문구가 표시된다. script 태그에 캔버스 코드를 넣는다.

```html
<script>
    var canvas = document.getElementById('myCanvas');
    var ctx = canvas.getContext('2d');
</script>
```

`getContext` 함수에 2d를 그린다고 선언하여, ctx 변수에 적용한다. 3d는 캔버스가 아닌 WebGL 과 같은 기능을 사용해야
한다.

## 선 그리기

### 선 그리기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.beginPath();
ctx.moveTo(100, 50);
ctx.lineTo(300, 50);
ctx.stroke();
```

3: 선 그리기를 시작한다.
4: 시작점으로 이동한다.
5: 선의 끝점으로 이동한다.
6: 선을 그린다.

![선그리기](/images/canvas-선그리기.png)

### 사각형 그리기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.beginPath();
ctx.moveTo(100, 50);
ctx.lineTo(300, 50);
ctx.lineTo(300, 200);
ctx.lineTo(100, 200);
ctx.lineTo(100, 50);
ctx.stroke();
```

![사각형그리기](/images/canvas-사각형그리기.png)

### 내부에 색 채우기

```javascript
// ...
ctx.stroke();
ctx.fillStyle = "red";
ctx.fill();
```

fillStyle 을 사용해 내부의 색을 지정해 줄 수 있다. fillStyle 없이 fill 함수만 실행할 경우 검정색으로 채워진다.

![색 채우기](/images/canvas-색채우기.png)

### 선의 색을 다른 색으로 채우고 두께 변경하기

```javascript
// 선의 색 변경
ctx.lineWidth = 20;
ctx.strokeStyle = "#0000ff";
ctx.stroke();

// 색 채우기
ctx.fillStyle = "red";
ctx.fill();
```

![선의 색 변경](/images/canvas-선색변경.png)

### 선의 끝 부분 처리하기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.lineWidth = 20;
ctx.strokeStyle = "#0000ff";

ctx.beginPath();
ctx.moveTo(100, 50);
ctx.lineTo(300, 50);
ctx.lineCap = "butt";
ctx.stroke();

ctx.beginPath();
ctx.moveTo(100, 100);
ctx.lineTo(300, 100);
ctx.lineCap = "round";
ctx.stroke();

ctx.beginPath();
ctx.moveTo(100, 150);
ctx.lineTo(300, 150);
ctx.lineCap = "square";
ctx.stroke();
```

![선의 끝부분 처리](/images/canvas-선의끝부분처리.png)

`lineCap` 을 사용하면 선의 끝 부분을 처리할 수 있다.

- butt: 선의 끝 부분을 좌표에 맞추어 마무리한다. 기본값
- round: 선의 끝을 둥글린다. 선 두께를 반지름으로 한다.
- square: 선의 끝을 사각형으로 처리한다. 선 두께만큼 길어진다.

### 선의 꺾인 부분 처리하기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.lineWidth = 20;
ctx.strokeStyle = "#0000ff";

ctx.beginPath();
ctx.moveTo(100, 50);
ctx.lineTo(300, 50);
ctx.lineTo(300, 100);
ctx.lineJoin = 'miter';
ctx.stroke();

ctx.beginPath();
ctx.moveTo(100, 150);
ctx.lineTo(300, 150);
ctx.lineTo(300, 200);
ctx.lineJoin = "round";
ctx.stroke();

ctx.beginPath();
ctx.moveTo(100, 250);
ctx.lineTo(300, 250);
ctx.lineTo(300, 290);
ctx.lineJoin = "bevel";
ctx.stroke();
```

![선의 꺾인 부분 처리하기](/images/canvas-선의꺾인부분처리.png)

`lineJoin` 를 사용하면 꺾인 부분을 처리할 수 있다.

- miter: 각진 모서리 형태로 기본값이다.
- round: 둥근 모서리 형태
- bevel: 잘려나간 모서리 형태 

### 선의 간격을 조정하여 점선 만들기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.lineWidth = 20;
ctx.strokeStyle = "#0000ff";

ctx.beginPath();
ctx.moveTo(100, 50);
ctx.lineTo(300, 50);
ctx.lineTo(300, 100);
ctx.setLineDash([20]);
ctx.stroke();

ctx.beginPath();
ctx.moveTo(100, 150);
ctx.lineTo(300, 150);
ctx.lineTo(300, 200);
ctx.setLineDash([20, 10]);
ctx.stroke();

ctx.beginPath();
ctx.moveTo(100, 250);
ctx.lineTo(300, 250);
ctx.lineTo(300, 290);
ctx.setLineDash([20, 10, 50, 10]);
ctx.stroke();
```

![점선 만들기](/images/canvas-점선처리.png)

`setLineDash` 를 사용하여 점선을 만든다.

1. 선의 간격이 20씩 벌어져 있다.
2. 선의 길이: 20, 간격: 10 만큼 벌어져 있다.
3. 선의 길이: 20, 간격: 10 과 선의 길이: 50, 간격: 10 이 번갈아 가면서 그려진다.

## 사각형 그리기

이전에는 선(line)을 이용하여 사각형을 그렸었는데 Rect 함수를 이용하여 사각형을 그릴 수 있다.

### 사각형 그리기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.strokeRect(20, 20, 100, 100);
ctx.strokeRect(150, 150, 50, 50);
```

![사각형](/images/canvas-사각형.png)

`strokeRect` 함수를 사용하면 사각형을 쉽게 그릴 수 있다.

> strokeRect(x, y, width, height);

### 사각형 색 채우기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.fillStyle = "magenta";
ctx.fillRect(20, 20, 100, 100);
ctx.strokeRect(20, 20, 100, 100);

ctx.fillStyle = "green";
ctx.fillRect(150, 150, 50, 50);
ctx.strokeRect(150, 150, 50, 50);
```

![사각형 색 채우기](/images/canvas-사각형색채우기.png)

### 내부를 사각형으로 지우기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.lineWidth = 10;
ctx.strokeStyle = 'red';
ctx.fillStyle = 'blue';
ctx.fillRect(50, 50, 200, 200);
ctx.strokeRect(50, 50, 200, 200);
ctx.clearRect(70, 70, 100, 50);
```

![내부를 사각형으로 지우기](/images/canvas-사각형내부지우기.png)

## 원 그리기

### 기본 원 그리기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.arc(150, 150, 100,0, Math.PI*2);
ctx.stroke();
```

![원 그리기](/images/canvas-원그리기.png)

`arc` 함수를 실행하여 원을 그릴 수 있다.

>context.arc(x, y, r, sAngle, eAngle, counterclockwise);
>x: x 좌표
>y: y 좌표
>r: 반지름
>sAngle: 시작하는 각도
>eAngle: 끝나는 각도
>counterclockwise: 시계 방향으로 회전

### 선과 호를 연결하여 라운드 코너 그리기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.beginPath();
ctx.moveTo(50, 50);
ctx.lineTo(300, 50);
ctx.arcTo(350, 50, 350, 100, 50);
ctx.lineTo(350, 200);
ctx.stroke();
```

![호 그리기](/images/canvas-호그리기.png)

`arcTo` 함수를 사용하여 호를 그릴 수 있다.

>context.arcTo(x1, y1, x2, y2, r)
>x1: 시작하는 점의 x 좌표
>y1: 시작하는 점의 y 좌표
>x2: 끝나는 점의 x 좌표
>y2: 끝나는 점의 y 좌표
>r: 호의 반지름

### quadraticCurve 그리기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.beginPath();
ctx.moveTo(50, 50);
ctx.lineTo(300, 50);
ctx.quadraticCurveTo(200, 100, 350, 100);
ctx.lineTo(350, 200);
ctx.stroke();
```

![quadraticCurveTo](/images/canvas-quadraticCurve.png)

`quadraticCurveTo` 함수를 이용하여 하나의 조절점의 커브를 그린다.

>context.quadraticCurve(cpx, cpy, x, y)
>cpx: 조절하는 점의 x 좌표
>cpy: 조절하는 점의 y 좌표
>x: 끝나는 점의 x 좌표
>y: 끝나는 점의 y 좌표


### bezierCurve 그리기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
ctx.beginPath();
ctx.moveTo(50, 50);
ctx.lineTo(300, 50);
ctx.bezierCurveTo(200,70, 100,150, 350,100);
ctx.lineTo(350, 200);
ctx.stroke();
```

![bezierCurveTo](/images/canvas-bezierCurve.png)

`bezierCurveTo` 함수를 이용하여 두 조절점의 커브를 그린다.

>context.bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)
>cp1x: 조절하는 점 1번째의 x 좌표
>cp1y: 조절하는 점 1번째의 y 좌표
>cp2x: 조절하는 점 2번째의 x 좌표
>cp2y: 조절하는 점 2번째의 y 좌표
>x: 끝나는 점의 x 좌표
>y: 끝나는 점의 y 좌표

## 내부 채우기

### Gradient 로 내부 채우기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
var grad = ctx.createLinearGradient(50, 50, 250, 50);
grad.addColorStop(0, 'red');
grad.addColorStop(1/6, 'orange');
grad.addColorStop(2/6, 'yellow');
grad.addColorStop(3/6, 'green');
grad.addColorStop(4/6, 'aqua');
grad.addColorStop(5/6, 'blue');
grad.addColorStop(1, 'purple');
ctx.lineWidth = 5;
ctx.fillStyle = grad;
ctx.fillRect(50, 50, 200, 200);
ctx.strokeRect(50, 50, 200, 200);
```

![gradient](/images/canvas-gradient.png)

그라데이션으로 색을 채우기 위해 `createLinearGradient` 함수를 사용했다.

>ctx.createLinearGradient(x0, y0, x1, y1);
>x0: 시작하는 점의 x 좌표
y0: 시작하는 점의 y 좌표
>x1: 끝나는 점의 x 좌표
>y1: 끝나는 점의 y 좌표

### radial gradient

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
var grad = ctx.createRadialGradient(0, 0, 0, 100, 100, 300);
grad.addColorStop(0, 'red');
grad.addColorStop(0.5, 'yellow');
grad.addColorStop(1, 'black');
ctx.lineWidth = 5;
ctx.fillStyle = grad;
ctx.fillRect(0, 0, 300, 300);
ctx.strokeRect(0, 0, 300, 300);
```

![radial gradient](/images/canvas-radialGradient.png)

>ctx.createRadialGradient(x0, y0, r0, x1, y1, r1);
>x0: 시작하는 점의 x 좌표
y0: 시작하는 점의 y 좌표
>r0: 시작하는 곳의 반지름
>x1: 끝나는 점의 x 좌표
>y1: 끝나는 점의 y 좌표
>r1: 끝 나는 곳의 반지름

### 패턴으로 사각형 채우기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
var flower = new Image();
flower.src = "images/flower.png";
flower.onload = function () {
    var pattern = ctx.createPattern(flower, "repeat");
    ctx.fillStyle = pattern;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
}
```

![createPattern](/images/canvas-pattern.png)

`createPattern` 를 이미지를 넘겨주어 실행한다.

context.createPattern(image, "repeat");
- repeat: 패턴을 반복하여 채운다.
- repeat-x: x축으로 반복하여 채운다.
- repeat-y: y축으로 반복하여 채운다.
- no-repeat: 반복하지 않는다.

## 이미지 그리기

### 이미지를 원래 크기대로 그리기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
var myPic = new Image();
myPic.src = 'images/duck.jpg';
myPic.onload = function () {
    ctx.drawImage(myPic, 10, 10);
}
```

![이미지 그리기](/images/canvas-image.png)

> context.drawImage(img, x, y)


### 이미지의 크기를 변형하여 그리기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
var myPic = new Image();
myPic.src = 'images/duck.jpg';
myPic.onload = function () {
    ctx.drawImage(myPic, 10, 10, 150, 150);
}
```

![이미지 크기 변형](/images/canvas-image2.png)

> context.drawImage(img, x, y, width, height)

### 이미지를 잘라 일부만 그리기

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
var myPic = new Image();
myPic.src = 'images/duck.jpg';
myPic.onload = function () {
    ctx.drawImage(myPic, 20, 20, 200, 200, 10, 10, 300, 200);
}
```

![이미지 일부만 그리기](/images/canvas-image3.png)

>context.drawImage(img, sx, sy, swidth, sheight, x, y, width, height)
>sx: 소스 이미지에서 잘라 가져올 시작점의 x 좌표
>sy: 소스 이미지에서 잘라 가져올 시작점의 y 좌표
>swidth: 소스 이미지에서 잘라 가져올 시작점 기준의 이미지 폭
>sheight: 소스 이미지에서 잘라 가져올 시작점 기준의 이미지 높이

참조: [HTML5 캔버스](https://book.naver.com/bookdb/book_detail.nhn?bid=11228559)
[github](https://github.com/kkangil/canvas-practice)
