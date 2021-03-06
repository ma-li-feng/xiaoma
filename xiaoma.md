##xiaoma
##马立峰
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>今天天气不错</title>
  <style>
    * {
      margin: 0;
      padding: 0;
    }

    #map {
      width: 800px;
      height: 600px;
      background-color: #CCCCCC;
      position: relative;
    }
  </style>
</head>
<body>
<div id="map">

</div>


<script>
  //食物的构造函数：
  (function () {
    function Food(options) {
      //构造函数中使用对象参数的好处：传参的顾虑比较少，可扩展性强
      //食物需要设置的属性：宽高背景色，横坐标纵坐标，地图
      this.width = options.width || 20;          //保存食物盒子宽度
      this.height = options.height || 20;        //保存食物盒子高度
      this.bgColor = options.bgColor || "green"; //保存食物盒子背景色
      this.map = options.map;                    //保存地图
      //x和y属性保存的是坐标（当前处于哪一个格子，不是具体的left和top值）
      this.x = 0;//在初始化食物的阶段，进行随机值的设置
      this.y = 0;
      this.element;                              //这是用来保存食物的结构的属性
    }

    Food.prototype.init = function () {
      //6 通过功能分析，我们发现每当一个食物被吃掉后，都需要重新创建新的食物，创建之前需要先删除被吃掉的旧食物
      //6.1 如何找到旧的那个食物？谁表示食物？this.element用于保存食物的盒子，只需要对内部值进行判断即可
      if (this.element) {
        //说明已经具有旧的食物了，删除
        this.map.removeChild(this.element);
      }
      //将食物创建出来，并且进行样式的设置
      //1 创建结构
      this.element = document.createElement("div");
      var div = this.element;//保存，方便后面书写使用
      //2 设置基本样式
      div.style.width = this.width + "px";
      div.style.height = this.height + "px";
      div.style.backgroundColor = this.bgColor;
      div.style.position = "absolute";//定位
      //3 计算出随机的坐标值 : Math.floor(Math.random()*地图宽/食物盒子宽);
      this.x = Math.floor(Math.random() * this.map.offsetWidth / this.width);
      this.y = Math.floor(Math.random() * this.map.offsetHeight / this.height);
      //4 根据取出的随机坐标位置，设置left和top即可
      div.style.left = this.x * this.width + "px";
      div.style.top = this.y * this.height + "px";
      //5 放入到地图中
      this.map.appendChild(div);
    };
    window.Food = Food;
  })();

  //小蛇的构造函数
  (function () {

    function Snake(options) {
      //小蛇这个对象需要哪些属性呢？
      //由于小蛇身体中的每个盒子都必须为通常的宽高，所以进行统一设置
      this.width = options.width || 20;  //宽度
      this.height = options.height || 20;//高度
      //body属性保存的是小蛇位置的所有信息，每个部分单独保存，每个部分中又有x和y和颜色属性
      this.body = [
        {x: 3, y: 7, bgColor: "red"},
        {x: 2, y: 7, bgColor: "orange"},
        {x: 1, y: 7, bgColor: "orange"}
      ];
      //小蛇还需要哪些属性？地图，食物对象,运动方向
      this.map = options.map;   //保存地图
      this.food = options.food; //保存食物对象
      this.direction = options.direction || "right";  //保存运动的方向

      // ---- 用于进行当前小蛇身体结构的保存 ----
      this.elements = [];
    }

    //设置初始化操作方法
    Snake.prototype.init = function () {

      // ---- 必须进行数据删除的原因 ----
      //this.elements = [];
      //第一轮创建执行后，内部应当具有3个div    init()
      //this.elements = ["div","div","div"];
      //第二轮创建执行前，先将内部原有的三个div对应的页面结构进行删除 move()
      //一定注意，删除的是页面中的div结构，但是elements中的对象是不受任何影响的
      //this.elements =   ["div","div","div"];
      //第三轮创建执行前，也要进行删除，此时this.elements应当有6个div
      //this.elements = ["旧的div", "旧的div", "旧的div", "新的div", "新的div", "新的div"];
      //当执行删除操作时，遇到了旧的div，执行removeChild，就会出现报错

      // ---- 4 删除操作 ----
      //新的小蛇绘制出来后，旧的小蛇依然存在，进行提前删除即可
      for (var i = this.elements.length - 1; i >= 0; i--) {
        //结构删除: removeChild()
        this.map.removeChild(this.elements[i]);
        //数据删除: 使用数组的splice方法。 ---- 需要进行数据删除的原因见上面的步骤分析
        this.elements.splice(i, 1);
      }

      //console.log(this.elements);

      // ---- 1 如果想要进行旧结构的删除，需要在创建时，先进行保存 ----
      // ---- 2 由于小蛇的身体分为n的部分，单一个属性形式无法保存，需要通过数组结构进行存储 ----
      //body中的所有对象表示小蛇身体中的每个部分，每个部分都是一个独立的盒子，进行遍历创建
      var body = this.body;//用于保存this.body的值
      for (var i = 0; i < body.length; i++) {
        //body[i] - 表示身体中某个盒子的所有信息
        var div = document.createElement("div");
        div.style.width = this.width + "px";
        div.style.height = this.height + "px";
        div.style.backgroundColor = body[i].bgColor;
        div.style.position = "absolute";
        //根据身体中每个部分的坐标信息设置小蛇每个部分的left和top属性值
        div.style.left = body[i].x * this.width + "px";
        div.style.top = body[i].y * this.height + "px";
        this.map.appendChild(div);//放入地图中显示

        // ---- 3 在创建时将创建的结构保存到this.elements数组中，方便删除时的查找 ----
        this.elements.push(div);

      }
    };
    //设置小蛇移动的功能
    Snake.prototype.move = function () {
      //我们发现，小蛇在运动的过程中，每次移动都是修改this.body中每个对象的x和y的值
      //我们又发现，每个身体部分在运动一次时，要更改为的值是前一个蛇身当前的位置
      //蛇头的坐标需要根据运动的方向进行单独设置
      var body = this.body;

      //保存了改变之前的蛇尾盒子的坐标
      var lastX = body[body.length - 1].x;
      var lastY = body[body.length - 1].y;

      for (var i = body.length - 1; i > 0; i--) {
        //body[i]就是除了蛇头以外的其他蛇身
        //x和y需要修改为前一项的对应x和y
        body[i].x = body[i - 1].x;
        body[i].y = body[i - 1].y;
      }

      //坐标重置完毕，再设置头的坐标
      //蛇头的坐标修改需要依据当前的运动位置
      switch (this.direction) {
        case "right":
          body[0].x++;
          break;
        case "left":
          body[0].x--;
          break;
        case "top":
          body[0].y--;
          break;
        case "bottom":
          body[0].y++;
      }

      // ---- 后期添加的功能 ----
      //运动完毕后，检测是否吃到了食物
      //先将游戏控制书写完毕，让小蛇跑起来，再处理这个问题
      //只要判断食物的坐标和蛇头的坐标，如果相等，说明吃到了食物，这时添加一个身体即可
      if (body[0].x == this.food.x && body[0].y == this.food.y) {
        //给小蛇添加一个身体的部分，实际上就是给this.body中添加一个新的对象即可
        //坐标可以使用蛇尾在运动之前的旧坐标(需要在上面的坐标修改代码前先进行一次保存)
        body.push({
          bgColor: "orange",//注意设置背景色时的属性名称，是bgColor！！！
          x: lastX,
          y: lastY,
        });
        //吃完后，设置完新的蛇身坐标后，重新创建一个食物
        this.food.init();
      }


      //所有坐标均设置完毕，调用小蛇的init，根据新的坐标重新绘制一个新的小蛇
      //this.init();
      //正常的思考方式，小蛇坐标改变后，应当立刻进行绘制操作，为了避免游戏结束后会出现小蛇走出到地图外的情况
      //我们可以将init操作设置到Game的snakeRun中

    };
    //将Snake暴露给window对象
    window.Snake = Snake;
  })();

  //创建游戏控制构造函数
  (function () {

    //可以在游戏功能的自调用函数中访问
    var that;

    function Game(options) {
      //需要使用哪些属性？地图，小蛇，食物
      this.map = options.map;
      this.food = new Food({map: this.map});//直接进行食物对象的实例化操作
      this.snake = new Snake({map: this.map, food: this.food});//使用snake属性直接进行小蛇的实例化操作（创建一个实例对象）


      that = this;
    }

    //Game的初始化操作
    Game.prototype.init = function () {
      //设置按键操作
      this.changeDirection();
      //初始化食物
      this.food.init();
      //初始化小蛇
      this.snake.init();
      //设置小蛇的运动
      this.snakeRun();
    };
    //Game对象设置让小蛇运动的代码
    Game.prototype.snakeRun = function () {
      //设置定时器，让小蛇间隔一段时间进行一次move操作即可
      var timer = setInterval(function () {
        //注意，此位置是定时器内部，是一个普通的函数结构，使用this指向了window对象
        //console.log(this);//window
        //console.log(this.snake);//undefined
        //this.snake.move();//报错
        //我们需要this指向通过Game创建的实例对象
        //通过在外部进行对this的保存操作，我们在这个位置可以使用that变量取到正确的值
        var she = that.snake;
        she.move();//此处修改后，move中仅仅修改了坐标，而没有进行小蛇的绘制操作

        //设置游戏结束的场景：
        //检测，当前小蛇运动的位置(蛇头的位置，如果超出了地图的边界，设置游戏结束);
        //超出的最大值，最大值为39，地图宽/单个小格宽度
        var maxX = that.map.offsetWidth / she.width;
        var maxY = that.map.offsetHeight / she.height;

        var sheHead = she.body[0];

        if (sheHead.x < 0 || sheHead.x > maxX - 1 || sheHead.y < 0 || sheHead.y > maxY - 1) {
          alert("游戏结束");
          clearInterval(timer);   //清除定时器
          return;                 //跳出，防止后面的init操作将小蛇绘制到地图以外。
        }

        //书写在这个位置的目的就是为了在视觉效果上看起没有问题
        //如果小蛇的坐标在合理范围内，进行绘制，否则直接在上面的if中结束所有代码，不进行绘制操作
        she.init();

      }, 150);

    };

    //给Game对象设置一个方法，用于键盘操作  keyup keydown
    Game.prototype.changeDirection = function () {
      //给document设置keydown事件
      document.onkeydown = function (e) {
        //如何判断，当前按下的按键是哪个按键呢？
        var e = e || window.event;
        //事件对象的keyCode属性，可以返回某个按键对应的唯一键盘码，数值类型
        //console.log(e.keyCode);
        //37 - 左    38 - 上   39 - 右   40 - 下
        var keyCode = e.keyCode;

        console.log(keyCode);

        //如果当前direction属性的值为right，而用户点击了37左按钮，阻止下面的运动方向修改
        if (that.snake.direction == "right" && keyCode == 37) {
          return;
        }


        switch (keyCode) {
          case 37:
            //更改小蛇的direction属性
            //事件内部，this不是我们需要的值，可以使用之前设置的that变量进行访问
            that.snake.direction = "left";
            break;
          case 38:
            that.snake.direction = "top";
            break;
          case 39:
            that.snake.direction = "right";
            break;
          case 40:
            that.snake.direction = "bottom";
            break;
        }
      };
    };

    //将Game暴露给window对象
    window.Game = Game;
  })();


  //具体的操作代码：
  var mapDiv = document.getElementById("map");
  var game1 = new Game({map: mapDiv});
  game1.init();


</script>
</body>
</html>