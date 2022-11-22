<template>
  <div>
      <canvas id="canvas" width="320" height="476" style="#000">
        Canvas not supported by your browser.
      </canvas>
  </div>
</template>

<script setup>
let canvas = null,
  ctx = null,
  scaleX=1,scaleY=1,
  lastUpdate = 0,

  cont = 0,
  puntoinicial = 0,
  puntofinal = 0,
  x1,y1,x2,y2,x, y,

  pause = false,
  lastPress = null,
  dir = 0,
  KEY_ENTER = 13,
  KEY_LEFT = 37,
  KEY_UP = 38,
  KEY_RIGHT = 39,
  KEY_DOWN = 40,

  mouse = {x: 0, y: 0},
  pointer = {x: 0, y:0},
  dragging = null,
  draggables = [],
  i = 0,
  l = 0,

  wall = new Array(),

  player = null,
  food = null,
  score = 0,
  record = 0,
  gameover = true;

onMounted(() => {
  window.requestAnimationFrame = (function () {
  return window.requestAnimationFrame ||
      window.mozRequestAnimationFrame ||
      window.webkitRequestAnimationFrame ||
      function (callback) {
          window.setTimeout(callback, 17);
      };
  }());

  document.addEventListener('keydown', function (evt) {lastPress = evt.which; }, false);

  window.addEventListener('load', init, false);
  window.addEventListener('resize',resize,false);
})

function enableInputs() {
    canvas.addEventListener('touchstart', function (evt)
    {
    evt.preventDefault();
    puntoinicial = evt.changedTouches;
    x1 = puntoinicial[0].pageX - canvas.offsetLeft;
    y1 = puntoinicial[0].pageY - canvas.offsetTop;

    if (
    	puntoinicial[0].pageX>(canvas.width/2)-20
    	&&
    	puntoinicial[0].pageX<(canvas.width/2)+20
    	&&
    	puntoinicial[0].pageY>(canvas.height/2)-20
    	&&
    	puntoinicial[0].pageY<(canvas.height/2)+20
    	) 
    	{lastPress = KEY_ENTER;};

    //cont++;
    
    //document.write(t[0].pageX);
    //if ((t[0].pageX-canvas.offsetLeft) < (t[0].pageX-canvas.offsetTop)) {
    //	lastPress = 1;
    //}
    //mouse.x = t[0].pageX - canvas.offsetLeft;
    //mouse.y = t[0].pageY - canvas.offsetTop;
    }, false);
    canvas.addEventListener('touchend', function (evt)
    {
    evt.preventDefault();
    puntofinal = evt.changedTouches;
    x2 = puntofinal[0].pageX - canvas.offsetLeft;
    y2 = puntofinal[0].pageY - canvas.offsetTop;


    x = x1-x2;
    y = y1-y2;
    if (x<0) {x=x*-1};
    if (y<0) {y=y*-1};

    if (y>x) {
    	if (y1>y2) {
    		lastPress = "arr";
    	} else lastPress = "aba";
    }
    if (y<x) {
    	if (x1<x2) {
    		lastPress = "der";
    	} else lastPress = "izq";
    }



    //cont++;
    //alert(t[0].pageY);
    //document.write(t[0].pageX);
    //if ((t[0].pageX-canvas.offsetLeft) < (t[0].pageX-canvas.offsetTop)) {
    //	lastPress = 1;
    //}
    //mouse.x = t[0].pageX - canvas.offsetLeft;
    //mouse.y = t[0].pageY - canvas.offsetTop;
    }, false);

    
}

function random(max) {
    return Math.floor(Math.random() * max);
}

function resize(){
        if(window.innerWidth>window.innerHeight){
            canvas.width=480;
            canvas.height=320;
        }
        else{
            canvas.width=320;
            canvas.height=480;
        }
        scaleX=canvas.width/window.innerWidth;
        scaleY=canvas.height/window.innerHeight;
    }

function reset() {
    score = 0;
    dir = 1;
    player.x = 40;
    player.y = 40;
    food.x = random(canvas.width / 20 - 1) * 20;
    food.y = random(canvas.height / 20 - 1) * 20;
    wall = new Array();
    gameover = false;
}

function Rectangle(x, y, width, height) {
    this.x = (x == undefined) ? 0 : x;
    this.y = (y == undefined) ? 0 : y;
    this.width = (width == undefined) ? 0 : width;
    this.height = (height == undefined) ? this.width : height;

    this.intersects = function (rect) {
        if (rect != undefined) {
            return (
                this.x < rect.x + rect.width
                &&
                this.x + this.width > rect.x
                &&
                this.y < rect.y + rect.height
                &&
                this.y + this.height > rect.y);
        }
    };

    this.fill = function (ctx) {
        if (ctx != undefined) {
            ctx.fillRect(this.x, this.y, this.width, this.height);
            //ctx.strokeRect(this.x, this.y, this.width, this.height);    //El contorno
        }
    };
}

function paint(ctx) {
    var i = 0,
        l = 0;
    
    //Clean canvas
    ctx.fillStyle = '#000';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    //Draw player
    ctx.fillStyle = '#00a000';
    //ctx.strokeStyle = '#000';   //Se difine el color del contorno
    player.fill(ctx);
    //ctx.strokeStyle = '#000';
    //ctx.strokeRect(x, y, 30, 30);
    
    //Draw wall
    ctx.fillStyle = '#999';
    for (i = 0, l = wall.length; i < l; i += 1) {
        wall[i].fill(ctx);
    }

    //Draw food
    ctx.fillStyle = '#f00';
    food.fill(ctx);
    
    //Debug last key pressed
    ctx.fillStyle = '#fff';
    
    //Draw Score and Record
    if (localStorage.record) {record=localStorage.getItem('record')};
    ctx.fillText('Puntaje: '+score, 0, 10);
    if(score > record) {
		record = score;
    	localStorage.setItem('record',record);
    }
    ctx.fillText('Record: '+record, 0, 20);
    
    if (pause) {
        ctx.textAlign = 'center';
        if (gameover) {
            ctx.fillText('Perdiste :(', (canvas.width/2), (canvas.height/2));
        } else {
            ctx.fillText('PAUSE', (canvas.width/2), (canvas.height/2));
        }
        
        ctx.textAlign = 'left';
    };
    
    //ctx.fillText('Last Press: ' + lastPress, 0, 20);
}

function act(deltaTime) {
    pointer.x = mouse.x;
    pointer.y = mouse.y;

    if (!pause) {

        // GameOver Reset
        if (gameover) {
            reset();
        }

        //definir direction
        if (lastPress == KEY_UP) {
            dir = 0;
        }
        if (lastPress == KEY_RIGHT) {
            dir = 1;
        }
        if (lastPress == KEY_DOWN) {
            dir = 2;
        }
        if (lastPress == KEY_LEFT) {
            dir = 3;
        }
	
		//direccionTouch
		if (lastPress == "arr") {
            dir = 0;
        }
        if (lastPress == "der") {
            dir = 1;
        }
        if (lastPress == "aba") {
            dir = 2;
        }
        if (lastPress == "izq") {
            dir = 3;
        }
	    
        //Move Rect
        if (dir == 0) {
        	cont += 120 * deltaTime;
        	if(cont > 20){
            player.y -= 20;
            cont = 0;
        	}
        }
        if (dir == 1) {
        	cont += 120 * deltaTime;
        	if (cont >= 20) {
        		player.x += 20;
        		cont = 0;
        	}
        }
        if (dir == 2) {
        	cont += 120 * deltaTime;
        	if (cont >= 20) {
        		player.y += 20;
        		cont = 0;
        	}
        }
        if (dir == 3) {
        	cont += 120 * deltaTime;
        	if (cont >=20) {
        		player.x -= 20;
        		cont = 0;
        	}
        }
        
        //Food  Intersects 
        if (player.intersects(food)) {
            score += 1;
            food.x = random(canvas.width / 20 - 1) * 20;
            food.y = random(canvas.height / 20 - 1) * 20;

            wall.push(new Rectangle(random(canvas.width / 20 - 1) * 20, random(canvas.height / 20 - 1) * 20, 20, 20));
        }

        // Wall Intersects
        for (i = 0, l = wall.length; i < l; i += 1) {
            if (food.intersects(wall[i])) {
                food.x = random(canvas.width / 20 - 1) * 20;
                food.y = random(canvas.height / 20 - 1) * 20;
            }

            if (player.intersects(wall[i])) {
                gameover = true;
                pause = true;
            }
        }

        //Out Screen
        if (player.y < 0) {
            player.y = canvas.height-20;
        }
        if (player.x > canvas.width-20) {
            player.x = 0;
        }
        if (player.y > canvas.height-20) {
            player.y = 0;
        }
        if (player.x < 0) {
            player.x = canvas.width-20;
        }
    }
    
    //Pause/Unpause
    if (lastPress == KEY_ENTER) {
        pause = !pause;
        lastPress = null;
    }
}

//function repaint() {
//    window.requestAnimationFrame(repaint);
//    paint(ctx);
//}

//function run() {
//    setTimeout(run, 150);
//    act();
//}

function run() {
	window.requestAnimationFrame(run);

	var now = Date.now(),
        deltaTime = (now - lastUpdate) / 1000;
    if (deltaTime > 1) {
        deltaTime = 0;
    }
    lastUpdate = now;

    act(deltaTime);
    paint(ctx)
}

    
function init() {
    // Get canvas and context
    canvas = document.getElementById('canvas');
    ctx = canvas.getContext('2d');

    canvas.style.position='fixed';
    canvas.style.top='0';
    canvas.style.left='0';
    canvas.style.width='100%';
    canvas.style.height='100%';
    
    // Create player and food
    player = new Rectangle(40,40,20,20);
    food = new Rectangle(80,80,20,20);

    resize();

    // Create walls
    var tdcc = 20;
    var ancho = canvas.width;
    var alto = canvas.height;
    var ndcsancho = ancho/tdcc;
    var ndcsalto = alto/tdcc;
    var unatercerapartean = ndcsancho/3;
    var unaterceraparteal = ndcsalto/3;

    var p1an = Math.floor(unatercerapartean*1)*20;
    var p2an = Math.floor(unatercerapartean*2)*20;
    var p3an = Math.floor(unatercerapartean*3)*20;

    var p1al = Math.floor(unaterceraparteal*1)*20;
    var p2al = Math.floor(unaterceraparteal*2)*20;
    var p3al = Math.floor(unaterceraparteal*3)*20;

    wall.push(new Rectangle(p1an, p1al, 20, 20));
    wall.push(new Rectangle(p1an, p2al, 20, 20));
    wall.push(new Rectangle(p2an, p1al, 20, 20));
    wall.push(new Rectangle(p2an, p2al, 20, 20));
    




    //start game
    enableInputs();
    //repaint();
    run();
    
}

</script>

<style>
  body {
      margin: 0px;
      padding: 0px;
  }
</style>