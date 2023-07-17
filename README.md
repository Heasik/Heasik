<!DOCTYPE HTML PUBLIC “-//W3C//DTD HTML 4.0 Transitional//EN”>

<HTML>

 <HEAD>

  <TITLE> Heart </TITLE>

  <META NAME=”Generator” CONTENT=”EditPlus”>

  <META NAME=”Author” CONTENT=””>

  <META NAME=”Keywords” CONTENT=””>

  <META NAME=”Description” CONTENT=””>

  <style>

  html, body {

  height: 100%;

  padding: 0;

  margin: 0;

  background: #000;

  display: flex;

  justify-content: center;

  align-items: center;

}

.box {

  width: 100%;

  position: absolute;

  top: 50%;

  left: 50%;

  transform: translate(-50%, -50%);

  display: flex;

  flex-direction: column;

}

canvas {

  position: absolute;

  width: 100%;

  height: 100%;

}

#pinkboard {

  position: relative;

  margin: auto;

  height: 500px;

  width: 500px;

  animation: animate 1.3s infinite;

}

#pinkboard:before, #pinkboard:after {

  content: ”;

  position: absolute;

  background: #FF5CA4;

  width: 100px;

  height: 160px;

  border-top-left-radius: 50px;

  border-top-right-radius: 50px;

}

#pinkboard:before {

  left: 100px;

  transform: rotate(-45deg);

  transform-origin: 0 100%;

  box-shadow: 0 14px 28px rgba(0,0,0,0.25),

              0 10px 10px rgba(0,0,0,0.22);

}

#pinkboard:after {

  left: 0;

  transform: rotate(45deg);

  transform-origin: 100% 100%;

}

@keyframes animate {

  0% {

    transform: scale(1);

  }

  30% {

    transform: scale(.8);

  }

  60% {

    transform: scale(1.2);

  }

  100% {

    transform: scale(1);

  }

}

  </style>

 </HEAD>

 <BODY>

   <div class=”box”>

      <canvas id=”pinkboard”></canvas>

   </div>

  <script>

  /*

 * Settings

 */

var settings = {

  particles: {

    length:   2000, // maximum amount of particles

    duration:   2, // particle duration in sec

    velocity: 100, // particle velocity in pixels/sec

    effect: -1.3, // play with this for a nice effect

    size:      13, // particle size in pixels

  },

};

/*

 * RequestAnimationFrame polyfill by Erik Möller

 */

(function(){var b=0;var c=[“ms”,”moz”,”webkit”,”o”];for(var a=0;a<c.length&&!window.requestAnimationFrame;++a){window.requestAnimationFrame=window[c[a]+”RequestAnimationFrame”];window.cancelAnimationFrame=window[c[a]+”CancelAnimationFrame”]||window[c[a]+”CancelRequestAnimationFrame”]}if(!window.requestAnimationFrame){window.requestAnimationFrame=function(h,e){var d=new Date().getTime();var f=Math.max(0,16-(d-b));var g=window.setTimeout(function(){h(d+f)},f);b=d+f;return g}}if(!window.cancelAnimationFrame){window.cancelAnimationFrame=function(d){clearTimeout(d)}}}());

/*

 * Point class

 */

var Point = (function() {

  function Point(x, y) {

    this.x = (typeof x !== ‘undefined’) ? x : 0;

    this.y = (typeof y !== ‘undefined’) ? y : 0;

  }

  Point.prototype.clone = function() {

    return new Point(this.x, this.y);

  };

  Point.prototype.length = function(length) {

    if (typeof length == ‘undefined’)

      return Math.sqrt(this.x * this.x + this.y * this.y);

    this.normalize();

    this.x *= length;

    this.y *= length;

    return this;

  };

  Point.prototype.normalize = function() {

    var length = this.length();

    this.x /= length;

    this.y /= length;

    return this;

  };

  return Point;

})();

/*

 * Particle class

 */

var Particle = (function() {

  function Particle() {

    this.position = new Point();

    this.velocity = new Point();

    this.acceleration = new Point();

    this.age = 0;

  }

  Particle.prototype.initialize = function(x, y, dx, dy) {

    this.position.x = x;

    this.position.y = y;

    this.velocity.x = dx;

    this.velocity.y = dy;

    this.acceleration.x = dx * settings.particles.effect;

    this.acceleration.y = dy * settings.particles.effect;

    this.age = 0;

  };

  Particle.prototype.update = function(deltaTime) {

    this.position.x += this.velocity.x * deltaTime;

    this.position.y += this.velocity.y * deltaTime;

    this.velocity.x += this.acceleration.x * deltaTime;

    this.velocity.y += this.acceleration.y * deltaTime;

    this.age += deltaTime;

  };

  Particle.prototype.draw = function(context, image) {

    function ease(t) {

      return (–t) * t * t + 1;

    }

    var size = image.width * ease(this.age / settings.particles.duration);

    context.globalAlpha = 1 – this.age / settings.particles.duration;

    context.drawImage(image, this.position.x – size / 2, this.position.y – size / 2, size, size);

  };

  return Particle;

})();

/*

 * ParticlePool class

 */

var ParticlePool = (function() {

  var particles,

      firstActive = 0,

      firstFree   = 0,

      duration    = settings.particles.duration;

  function ParticlePool(length) {

    // create and populate particle pool

    particles = new Array(length);

    for (var i = 0; i < particles.length; i++)

      particles[i] = new Particle();

  }

  ParticlePool.prototype.add = function(x, y, dx, dy) {

    particles[firstFree].initialize(x, y, dx, dy);

    // handle circular queue

    firstFree++;

    if (firstFree   == particles.length) firstFree   = 0;

    if (firstActive == firstFree       ) firstActive++;

    if (firstActive == particles.length) firstActive = 0;

  };

  ParticlePool.prototype.update = function(deltaTime) {

    var i;

    // update active particles

    if (firstActive < firstFree) {

      for (i = firstActive; i < firstFree; i++)

        particles[i].update(deltaTime);

    }

    if (firstFree < firstActive) {

      for (i = firstActive; i < particles.length; i++)

        particles[i].update(deltaTime);

      for (i = 0; i < firstFree; i++)

        particles[i].update(deltaTime);

    }

    // remove inactive particles

    while (particles[firstActive].age >= duration && firstActive != firstFree) {

      firstActive++;

      if (firstActive == particles.length) firstActive = 0;

    }

  };

  ParticlePool.prototype.draw = function(context, image) {

    // draw active particles

    if (firstActive < firstFree) {

      for (i = firstActive; i < firstFree; i++)

        particles[i].draw(context, image);

    }

    if (firstFree < firstActive) {

      for (i = firstActive; i < particles.length; i++)

        particles[i].draw(context, image);

      for (i = 0; i < firstFree; i++)

        particles[i].draw(context, image);

    }

  };

  return ParticlePool;

})();

/*

 * Putting it all together

 */

(function(canvas) {

  var context = canvas.getContext(‘2d’),

      particles = new ParticlePool(settings.particles.length),

      particleRate = settings.particles.length / settings.particles.duration, // particles/sec

      time;

  // get point on heart with -PI <= t <= PI

  function pointOnHeart(t) {

    return new Point(

      160 * Math.pow(Math.sin(t), 3),

      130 * Math.cos(t) – 50 * Math.cos(2 * t) – 20 * Math.cos(3 * t) – 10 * Math.cos(4 * t) + 25

    );

  }

  // creating the particle image using a dummy canvas

  var image = (function() {

    var canvas  = document.createElement(‘canvas’),

        context = canvas.getContext(‘2d’);

    canvas.width  = settings.particles.size;

    canvas.height = settings.particles.size;

    // helper function to create the path

    function to(t) {

      var point = pointOnHeart(t);

      point.x = settings.particles.size / 2 + point.x * settings.particles.size / 350;

      point.y = settings.particles.size / 2 – point.y * settings.particles.size / 350;

      return point;

    }

    // create the path

    context.beginPath();

    var t = -Math.PI;

    var point = to(t);

    context.moveTo(point.x, point.y);

    while (t < Math.PI) {

      t += 0.01; // baby steps!

      point = to(t);

      context.lineTo(point.x, point.y);

    }

    context.closePath();

    // create the fill

    context.fillStyle = ‘#FF5CA4’;

    context.fill();

    // create the image

    var image = new Image();

    image.src = canvas.toDataURL();

    return image;

  })();

  // render that thing!

  function render() {

    // next animation frame

    requestAnimationFrame(render);

    // update time

    var newTime   = new Date().getTime() / 1000,

        deltaTime = newTime – (time || newTime);

    time = newTime;

    // clear canvas

    context.clearRect(0, 0, canvas.width, canvas.height);

    // create new particles

    var amount = particleRate * deltaTime;

    for (var i = 0; i < amount; i++) {

      var pos = pointOnHeart(Math.PI – 2 * Math.PI * Math.random());

      var dir = pos.clone().length(settings.particles.velocity);

      particles.add(canvas.width / 2 + pos.x, canvas.height / 2 – pos.y, dir.x, -dir.y);

    }

    // update and draw particles

    particles.update(deltaTime);

    particles.draw(context, image);

  }

  // handle (re-)sizing of the canvas

  function onResize() {

    canvas.width  = canvas.clientWidth;

    canvas.height = canvas.clientHeight;

  }

  window.onresize = onResize;

  // delay rendering bootstrap

  setTimeout(function() {

    onResize();

    render();

  }, 10);

})(document.getElementById(‘pinkboard’));

  </script>

  <div class=”center-text”,

  style=”background-color:rgb(0, 0, 0);

        width: 100%;

        color: rgb(225, 12, 168);

        height:100%;

        font-size: 31px;

        font-style: italic;

        display: flex;

        align-items: center;

        justify-content: center;

        margin-bottom: 5px;

        text-align: center;”>Anh Yêu Em</div>

 </BODY>

</HTML>

3. Code mẫu trái tim cơ bản
Đây chính là những dòng code trước tiên nổi rần rần trên cộng đồng mạng khi phim đã ra mắt tập đầu tiên. Cụ thể, hình trái tim được thiết lập bởi code HTML, vẫn có hiệu ứng động nhưng lại không thể đập được.

Soure code: Internet

4. Tải code trái tim đập của Thủ Khoa Lý có tên
Nói chung, đây là một mẫu code trái tim giống với loại cơ bản về hình dạng và kết quả. Tuy nhiên, sự khác biệt đó chính là mọi người có thể chèn chữ hoặc chỉnh sửa chữ ở trong theo ý muốn.

Source code: TikTok @_hann_kyy

5. Tải code trái tim đập của Thủ Khoa Lý không tên
Nếu như các bạn chỉ cần code trái tim cơ bản mà không phải là code có tên và chữ bên trong thì hãy sử dụng đoạn code ở bên dưới đây.

Điều chỉnh code hình trái tim đập không tên
<!DOCTYPE HTML PUBLIC “-//W3C//DTD HTML 4.0 Transitional//EN”>

<HTML>

 <HEAD>

  <TITLE> New Document </TITLE>

  <META NAME=”Generator” CONTENT=”EditPlus”>

  <META NAME=”Author” CONTENT=””>

  <META NAME=”Keywords” CONTENT=””>

  <META NAME=”Description” CONTENT=””>

  <style>

  html, body {

  height: 100%;

  padding: 0;

  margin: 0;

  background: #000;

}

canvas {

  position: absolute;

  width: 100%;

  height: 100%;

}

  </style>

 </HEAD>

 <BODY>

  <canvas id=”pinkboard”></canvas>

  <script>

  /*

 * Settings

 */

var settings = {

  particles: {

    length:   500, // maximum amount of particles

    duration:   2, // particle duration in sec

    velocity: 100, // particle velocity in pixels/sec

    effect: -0.75, // play with this for a nice effect

    size:      30, // particle size in pixels

  },

};

/*

 * RequestAnimationFrame polyfill by Erik Möller

 */

(function(){var b=0;var c=[“ms”,”moz”,”webkit”,”o”];for(var a=0;a<c.length&&!window.requestAnimationFrame;++a){window.requestAnimationFrame=window[c[a]+”RequestAnimationFrame”];window.cancelAnimationFrame=window[c[a]+”CancelAnimationFrame”]||window[c[a]+”CancelRequestAnimationFrame”]}if(!window.requestAnimationFrame){window.requestAnimationFrame=function(h,e){var d=new Date().getTime();var f=Math.max(0,16-(d-b));var g=window.setTimeout(function(){h(d+f)},f);b=d+f;return g}}if(!window.cancelAnimationFrame){window.cancelAnimationFrame=function(d){clearTimeout(d)}}}());

/*

 * Point class

 */

var Point = (function() {

  function Point(x, y) {

    this.x = (typeof x !== ‘undefined’) ? x : 0;

    this.y = (typeof y !== ‘undefined’) ? y : 0;

  }

  Point.prototype.clone = function() {

    return new Point(this.x, this.y);

  };

  Point.prototype.length = function(length) {

    if (typeof length == ‘undefined’)

      return Math.sqrt(this.x * this.x + this.y * this.y);

    this.normalize();

    this.x *= length;

    this.y *= length;

    return this;

  };

  Point.prototype.normalize = function() {

    var length = this.length();

    this.x /= length;

    this.y /= length;

    return this;

  };

  return Point;

})();

/*

 * Particle class

 */

var Particle = (function() {

  function Particle() {

    this.position = new Point();

    this.velocity = new Point();

    this.acceleration = new Point();

    this.age = 0;

  }

  Particle.prototype.initialize = function(x, y, dx, dy) {

    this.position.x = x;

    this.position.y = y;

    this.velocity.x = dx;

    this.velocity.y = dy;

    this.acceleration.x = dx * settings.particles.effect;

    this.acceleration.y = dy * settings.particles.effect;

    this.age = 0;

  };

  Particle.prototype.update = function(deltaTime) {

    this.position.x += this.velocity.x * deltaTime;

    this.position.y += this.velocity.y * deltaTime;

    this.velocity.x += this.acceleration.x * deltaTime;

    this.velocity.y += this.acceleration.y * deltaTime;

    this.age += deltaTime;

  };

  Particle.prototype.draw = function(context, image) {

    function ease(t) {

      return (–t) * t * t + 1;

    }

    var size = image.width * ease(this.age / settings.particles.duration);

    context.globalAlpha = 1 – this.age / settings.particles.duration;

    context.drawImage(image, this.position.x – size / 2, this.position.y – size / 2, size, size);

  };

  return Particle;

})();

/*

 * ParticlePool class

 */

var ParticlePool = (function() {

  var particles,

      firstActive = 0,

      firstFree   = 0,

      duration    = settings.particles.duration;

  function ParticlePool(length) {

    // create and populate particle pool

    particles = new Array(length);

    for (var i = 0; i < particles.length; i++)

      particles[i] = new Particle();

  }

  ParticlePool.prototype.add = function(x, y, dx, dy) {

    particles[firstFree].initialize(x, y, dx, dy);

    // handle circular queue

    firstFree++;

    if (firstFree   == particles.length) firstFree   = 0;

    if (firstActive == firstFree       ) firstActive++;

    if (firstActive == particles.length) firstActive = 0;

  };

  ParticlePool.prototype.update = function(deltaTime) {

    var i;

    // update active particles

    if (firstActive < firstFree) {

      for (i = firstActive; i < firstFree; i++)

        particles[i].update(deltaTime);

    }

    if (firstFree < firstActive) {

      for (i = firstActive; i < particles.length; i++)

        particles[i].update(deltaTime);

      for (i = 0; i < firstFree; i++)

        particles[i].update(deltaTime);

    }

    // remove inactive particles

    while (particles[firstActive].age >= duration && firstActive != firstFree) {

      firstActive++;

      if (firstActive == particles.length) firstActive = 0;

    }

  };

  ParticlePool.prototype.draw = function(context, image) {

    // draw active particles

    if (firstActive < firstFree) {

      for (i = firstActive; i < firstFree; i++)

        particles[i].draw(context, image);

    }

    if (firstFree < firstActive) {

      for (i = firstActive; i < particles.length; i++)

        particles[i].draw(context, image);

      for (i = 0; i < firstFree; i++)

        particles[i].draw(context, image);

    }

  };

  return ParticlePool;

})();

/*

 * Putting it all together

 */

(function(canvas) {

  var context = canvas.getContext(‘2d’),

      particles = new ParticlePool(settings.particles.length),

      particleRate = settings.particles.length / settings.particles.duration, // particles/sec

      time;

  // get point on heart with -PI <= t <= PI

  function pointOnHeart(t) {

    return new Point(

      160 * Math.pow(Math.sin(t), 3),

      130 * Math.cos(t) – 50 * Math.cos(2 * t) – 20 * Math.cos(3 * t) – 10 * Math.cos(4 * t) + 25

    );

  }

  // creating the particle image using a dummy canvas

  var image = (function() {

    var canvas  = document.createElement(‘canvas’),

        context = canvas.getContext(‘2d’);

    canvas.width  = settings.particles.size;

    canvas.height = settings.particles.size;

    // helper function to create the path

    function to(t) {

      var point = pointOnHeart(t);

      point.x = settings.particles.size / 2 + point.x * settings.particles.size / 350;

      point.y = settings.particles.size / 2 – point.y * settings.particles.size / 350;

      return point;

    }

    // create the path

    context.beginPath();

    var t = -Math.PI;

    var point = to(t);

    context.moveTo(point.x, point.y);

    while (t < Math.PI) {

      t += 0.01; // baby steps!

      point = to(t);

      context.lineTo(point.x, point.y);

    }

    context.closePath();

    // create the fill

    context.fillStyle = ‘#ea80b0’;

    context.fill();

    // create the image

    var image = new Image();

    image.src = canvas.toDataURL();

    return image;

  })();

  // render that thing!

  function render() {

    // next animation frame

    requestAnimationFrame(render);

    // update time

    var newTime   = new Date().getTime() / 1000,

        deltaTime = newTime – (time || newTime);

    time = newTime;

    // clear canvas

    context.clearRect(0, 0, canvas.width, canvas.height);

    // create new particles

    var amount = particleRate * deltaTime;

    for (var i = 0; i < amount; i++) {

      var pos = pointOnHeart(Math.PI – 2 * Math.PI * Math.random());

      var dir = pos.clone().length(settings.particles.velocity);

      particles.add(canvas.width / 2 + pos.x, canvas.height / 2 – pos.y, dir.x, -dir.y);

    }

    // update and draw particles

    particles.update(deltaTime);

    particles.draw(context, image);

  }

  // handle (re-)sizing of the canvas

  function onResize() {

    canvas.width  = canvas.clientWidth;

    canvas.height = canvas.clientHeight;

  }

  window.onresize = onResize;

  // delay rendering bootstrap

  setTimeout(function() {

    onResize();

    render();

  }, 10);

})(document.getElementById(‘pinkboard’));

  </script>

 </BODY>

</HTML>

6. Code trái tim đập tia sáng nghệ thuật
Nếu bạn thích sự sáng tạo và hấp dẫn hơn nữa thì mẫu code trái tim này là dành cho bạn. Nó sẽ có thêm các tia sáng xung quanh chủ thể nên trông cực kỳ đẹp mắt.

Điều chỉnh code hình trái tim đập tia sáng
<!DOCTYPE html>

<html lang=”en”>

<head>

    <meta charset=”UTF-8″>

    <meta http-equiv=”X-UA-Compatible” content=”IE=edge”>

    <meta name=”viewport” content=”width=device-width, initial-scale=1.0″>

    <title>Document</title>

    <style>

        canvas {

            position: absolute;

            left: 0;

            top: 0;

            width: 100%;

            height: 100%;

            background-color: rgba(0, 0, 0, .2);

        }

    </style>

</head>

<body>

    <canvas id=”heart”></canvas>

    <script>

        window.requestAnimationFrame =

            window.__requestAnimationFrame ||

            window.requestAnimationFrame ||

            window.webkitRequestAnimationFrame ||

            window.mozRequestAnimationFrame ||

            window.oRequestAnimationFrame ||

            window.msRequestAnimationFrame ||

            (function () {

                return function (callback, element) {

                    var lastTime = element.__lastTime;

                    if (lastTime === undefined) {

                        lastTime = 0;

                    }

                    var currTime = Date.now();

                    var timeToCall = Math.max(1, 33 – (currTime – lastTime));

                    window.setTimeout(callback, timeToCall);

                    element.__lastTime = currTime + timeToCall;

                };

            })();

        window.isDevice = (/android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(((navigator.userAgent || navigator.vendor || window.opera)).toLowerCase()));

        var loaded = false;

        var init = function () {

            if (loaded) return;

            loaded = true;

            var mobile = window.isDevice;

            var koef = mobile ? 0.5 : 1;

            var canvas = document.getElementById(‘heart’);

            var ctx = canvas.getContext(‘2d’);

            var width = canvas.width = koef * innerWidth;

            var height = canvas.height = koef * innerHeight;

            var rand = Math.random;

            ctx.fillStyle = “rgba(0,0,0,1)”;

            ctx.fillRect(0, 0, width, height);

            var heartPosition = function (rad) {

                //return [Math.sin(rad), Math.cos(rad)];

                return [Math.pow(Math.sin(rad), 3), -(15 * Math.cos(rad) – 5 * Math.cos(2 * rad) – 2 * Math.cos(3 * rad) – Math.cos(4 * rad))];

            };

            var scaleAndTranslate = function (pos, sx, sy, dx, dy) {

                return [dx + pos[0] * sx, dy + pos[1] * sy];

            };

            window.addEventListener(‘resize’, function () {

                width = canvas.width = koef * innerWidth;

                height = canvas.height = koef * innerHeight;

                ctx.fillStyle = “rgba(0,0,0,1)”;

                ctx.fillRect(0, 0, width, height);

            });

            var traceCount = mobile ? 20 : 50;

            var pointsOrigin = [];

            var i;

            var dr = mobile ? 0.3 : 0.1;

            for (i = 0; i < Math.PI * 2; i += dr) pointsOrigin.push(scaleAndTranslate(heartPosition(i), 210, 13, 0, 0));

            for (i = 0; i < Math.PI * 2; i += dr) pointsOrigin.push(scaleAndTranslate(heartPosition(i), 150, 9, 0, 0));

            for (i = 0; i < Math.PI * 2; i += dr) pointsOrigin.push(scaleAndTranslate(heartPosition(i), 90, 5, 0, 0));

            var heartPointsCount = pointsOrigin.length;

            var targetPoints = [];

            var pulse = function (kx, ky) {

                for (i = 0; i < pointsOrigin.length; i++) {

                    targetPoints[i] = [];

                    targetPoints[i][0] = kx * pointsOrigin[i][0] + width / 2;

                    targetPoints[i][1] = ky * pointsOrigin[i][1] + height / 2;

                }

            };

            var e = [];

            for (i = 0; i < heartPointsCount; i++) {

                var x = rand() * width;

                var y = rand() * height;

                e[i] = {

                    vx: 0,

                    vy: 0,

                    R: 2,

                    speed: rand() + 5,

                    q: ~~(rand() * heartPointsCount),

                    D: 2 * (i % 2) – 1,

                    force: 0.2 * rand() + 0.7,

                    f: “hsla(0,” + ~~(40 * rand() + 60) + “%,” + ~~(60 * rand() + 20) + “%,.3)”,

                    trace: []

                };

                for (var k = 0; k < traceCount; k++) e[i].trace[k] = { x: x, y: y };

            }

            var config = {

                traceK: 0.4,

                timeDelta: 0.01

            };

            var time = 0;

            var loop = function () {

                var n = -Math.cos(time);

                pulse((1 + n) * .5, (1 + n) * .5);

                time += ((Math.sin(time)) < 0 ? 9 : (n > 0.8) ? .2 : 1) * config.timeDelta;

                ctx.fillStyle = “rgba(0,0,0,.1)”;

                ctx.fillRect(0, 0, width, height);

                for (i = e.length; i–;) {

                    var u = e[i];

                    var q = targetPoints[u.q];

                    var dx = u.trace[0].x – q[0];

                    var dy = u.trace[0].y – q[1];

                    var length = Math.sqrt(dx * dx + dy * dy);

                    if (10 > length) {

                        if (0.95 < rand()) {

                            u.q = ~~(rand() * heartPointsCount);

                        }

                        else {

                            if (0.99 < rand()) {

                                u.D *= -1;

                            }

                            u.q += u.D;

                            u.q %= heartPointsCount;

                            if (0 > u.q) {

                                u.q += heartPointsCount;

                            }

                        }

                    }

                    u.vx += -dx / length * u.speed;

                    u.vy += -dy / length * u.speed;

                    u.trace[0].x += u.vx;

                    u.trace[0].y += u.vy;

                    u.vx *= u.force;

                    u.vy *= u.force;

                    for (k = 0; k < u.trace.length – 1;) {

                        var T = u.trace[k];

                        var N = u.trace[++k];

                        N.x -= config.traceK * (N.x – T.x);

                        N.y -= config.traceK * (N.y – T.y);

                    }

                    ctx.fillStyle = u.f;

                    for (k = 0; k < u.trace.length; k++) {

                        ctx.fillRect(u.trace[k].x, u.trace[k].y, 1, 1);

                    }

                }

                //ctx.fillStyle = “rgba(255,255,255,1)”;

                //for (i = u.trace.length; i–;) ctx.fillRect(targetPoints[i][0], targetPoints[i][1], 2, 2);

                window.requestAnimationFrame(loop, canvas);

            };

            loop();

        };

        var s = document.readyState;

        if (s === ‘complete’ || s === ‘loaded’ || s === ‘interactive’) init();

        else document.addEventListener(‘DOMContentLoaded’, init, false);

    </script>

</body>

</html>

7. Code trái tim đập màu đỏ
Đây cũng là một code trái tim xịn sò mà mọi người có thể sử dụng và gửi tặng cho đối phương của mình. Trái tim mà bạn dùng với đoạn mã code này sở hữu màu đỏ. Chưa hết, xung quanh nó còn lan tỏa ra nhiều trái tim nhỏ cực kỳ xinh và dễ thương.

Điều chỉnh mã code hình trái tim màu đỏ
<!DOCTYPE HTML PUBLIC “-//W3C//DTD HTML 4.0 Transitional//EN”>

<HTML>

 <HEAD>

  <TITLE> MINH IT </TITLE>

  <META NAME=”Generator” CONTENT=”EditPlus”>

  <META NAME=”Author” CONTENT=””>

  <META NAME=”Keywords” CONTENT=””>

  <META NAME=”Description” CONTENT=””>

  <link rel=”stylesheet” href=”style.css”>

  <style>

  html, body {

  height: 100%;

  padding: 0;

  margin: 0;

  background: rgba(0, 0, 0, 0.851);

}

canvas {

  position: absolute;

  width: 100%;

  height: 100%;

}

  </style>

 </HEAD>

 <BODY>

  <div class=”box”>

    <canvas id=”pinkboard”></canvas>

  </div>

<script>

var settings = {

  particles: {

    length:   10000, // maximum amount of particles

    duration:   4, // particle duration in sec

    velocity: 80, // particle velocity in pixels/sec

    effect: -1.3, // play with this for a nice effect

    size:      8, // particle size in pixels

  },

};

/*

 */

(function(){var b=0;var c=[“ms”,”moz”,”webkit”,”o”];for(var a=0;a<c.length&&!window.requestAnimationFrame;++a){window.requestAnimationFrame=window[c[a]+”RequestAnimationFrame”];window.cancelAnimationFrame=window[c[a]+”CancelAnimationFrame”]||window[c[a]+”CancelRequestAnimationFrame”]}if(!window.requestAnimationFrame){window.requestAnimationFrame=function(h,e){var d=new Date().getTime();var f=Math.max(0,16-(d-b));var g=window.setTimeout(function(){h(d+f)},f);b=d+f;return g}}if(!window.cancelAnimationFrame){window.cancelAnimationFrame=function(d){clearTimeout(d)}}}());

/*

 * Point class

 */

var Point = (function() {

  function Point(x, y) {

    this.x = (typeof x !== ‘undefined’) ? x : 0;

    this.y = (typeof y !== ‘undefined’) ? y : 0;

  }

  Point.prototype.clone = function() {

    return new Point(this.x, this.y);

  };

  Point.prototype.length = function(length) {

    if (typeof length == ‘undefined’)

      return Math.sqrt(this.x * this.x + this.y * this.y);

    this.normalize();

    this.x *= length;

    this.y *= length;

    return this;

  };

  Point.prototype.normalize = function() {

    var length = this.length();

    this.x /= length;

    this.y /= length;

    return this;

  };

  return Point;

})();

/*

 * Particle class

 */

var Particle = (function() {

  function Particle() {

    this.position = new Point();

    this.velocity = new Point();

    this.acceleration = new Point();

    this.age = 0;

  }

  Particle.prototype.initialize = function(x, y, dx, dy) {

    this.position.x = x;

    this.position.y = y;

    this.velocity.x = dx;

    this.velocity.y = dy;

    this.acceleration.x = dx * settings.particles.effect;

    this.acceleration.y = dy * settings.particles.effect;

    this.age = 0;

  };

  Particle.prototype.update = function(deltaTime) {

    this.position.x += this.velocity.x * deltaTime;

    this.position.y += this.velocity.y * deltaTime;

    this.velocity.x += this.acceleration.x * deltaTime;

    this.velocity.y += this.acceleration.y * deltaTime;

    this.age += deltaTime;

  };

  Particle.prototype.draw = function(context, image) {

    function ease(t) {

      return (–t) * t * t + 1;

    }

    var size = image.width * ease(this.age / settings.particles.duration);

    context.globalAlpha = 1 – this.age / settings.particles.duration;

    context.drawImage(image, this.position.x – size / 2, this.position.y – size / 2, size, size);

  };

  return Particle;

})();

/*

 * ParticlePool class

 */

var ParticlePool = (function() {

  var particles,

      firstActive = 0,

      firstFree   = 0,

      duration    = settings.particles.duration;

  function ParticlePool(length) {

    // create and populate particle pool

    particles = new Array(length);

    for (var i = 0; i < particles.length; i++)

      particles[i] = new Particle();

  }

  ParticlePool.prototype.add = function(x, y, dx, dy) {

    particles[firstFree].initialize(x, y, dx, dy);

    // handle circular queue

    firstFree++;

    if (firstFree   == particles.length) firstFree   = 0;

    if (firstActive == firstFree       ) firstActive++;

    if (firstActive == particles.length) firstActive = 0;

  };

  ParticlePool.prototype.update = function(deltaTime) {

    var i;

    // update active particles

    if (firstActive < firstFree) {

      for (i = firstActive; i < firstFree; i++)

        particles[i].update(deltaTime);

    }

    if (firstFree < firstActive) {

      for (i = firstActive; i < particles.length; i++)

        particles[i].update(deltaTime);

      for (i = 0; i < firstFree; i++)

        particles[i].update(deltaTime);

    }

    // remove inactive particles

    while (particles[firstActive].age >= duration && firstActive != firstFree) {

      firstActive++;

      if (firstActive == particles.length) firstActive = 0;

    }

  };

  ParticlePool.prototype.draw = function(context, image) {

    // draw active particles

    if (firstActive < firstFree) {

      for (i = firstActive; i < firstFree; i++)

        particles[i].draw(context, image);

    }

    if (firstFree < firstActive) {

      for (i = firstActive; i < particles.length; i++)

        particles[i].draw(context, image);

      for (i = 0; i < firstFree; i++)

        particles[i].draw(context, image);

    }

  };

  return ParticlePool;

})();

/*

 * Putting it all together

 */

(function(canvas) {

  var context = canvas.getContext(‘2d’),

      particles = new ParticlePool(settings.particles.length),

      particleRate = settings.particles.length / settings.particles.duration, // particles/sec

      time;

  // get point on heart with -PI <= t <= PI

  function pointOnHeart(t) {

    return new Point(

      160 * Math.pow(Math.sin(t), 3),

      130 * Math.cos(t) – 50 * Math.cos(2 * t) – 20 * Math.cos(3 * t) – 10 * Math.cos(4 * t) + 25

    );

  }

  // creating the particle image using a dummy canvas

  var image = (function() {

    var canvas  = document.createElement(‘canvas’),

        context = canvas.getContext(‘2d’);

    canvas.width  = settings.particles.size;

    canvas.height = settings.particles.size;

    // helper function to create the path

    function to(t) {

      var point = pointOnHeart(t);

      point.x = settings.particles.size / 2 + point.x * settings.particles.size / 350;

      point.y = settings.particles.size / 2 – point.y * settings.particles.size / 350;

      return point;

    }

    // create the path

    context.beginPath();

    var t = -Math.PI;

    var point = to(t);

    context.moveTo(point.x, point.y);

    while (t < Math.PI) {

      t += 0.01; // baby steps!

      point = to(t);

      context.lineTo(point.x, point.y);

    }

    context.closePath();

    // create the fill

    context.fillStyle = ‘#f50b02’;

    context.fill();

    // create the image

    var image = new Image();

    image.src = canvas.toDataURL();

    return image;

  })();

  // render that thing!

  function render() {

    // next animation frame

    requestAnimationFrame(render);

    // update time

    var newTime   = new Date().getTime() / 1000,

        deltaTime = newTime – (time || newTime);

    time = newTime;

    // clear canvas

    context.clearRect(0, 0, canvas.width, canvas.height);

    // create new particles

    var amount = particleRate * deltaTime;

    for (var i = 0; i < amount; i++) {

      var pos = pointOnHeart(Math.PI – 2 * Math.PI * Math.random());

      var dir = pos.clone().length(settings.particles.velocity);

      particles.add(canvas.width / 2 + pos.x, canvas.height / 2 – pos.y, dir.x, -dir.y);

    }

    // update and draw particles

    particles.update(deltaTime);

    particles.draw(context, image);

  }

  // handle (re-)sizing of the canvas

  function onResize() {

    canvas.width  = canvas.clientWidth;

    canvas.height = canvas.clientHeight;

  }

  window.onresize = onResize;

  // delay rendering bootstrap

  setTimeout(function() {

    onResize();

    render();

  }, 10);

})(document.getElementById(‘pinkboard’));

  </script>

 </BODY>

</HTML>
<!---
Heasik/Heasik is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
