# Tizen-App-SaoLei
###概述
扫雷基于web project开发，本篇主要介绍扫雷中的核心算法。
核心代码主要位于main.js文件中

####对事件进行注册
<pre>
var backEventListener = null;
var unregister = function() {
    if ( backEventListener !== null ) {
        document.removeEventListener( 'tizenhwkey', backEventListener );
        backEventListener = null;
        window.tizen.application.getCurrentApplication().exit();
    }
}

var init = function () {
    // register once
    if ( backEventListener !== null ) {
        return;
    }
    
    // TODO:: Do your initialization job
    console.log("init() called");
    
    var backEvent = function(e) {
        if ( e.keyName == "back" ) {
            try {
                if ( $.mobile.urlHistory.activeIndex <= 0 ) {
                    // if first page, terminate app
                    unregister();
                } else {
                    // move previous page
                    $.mobile.urlHistory.activeIndex -= 1;
                    $.mobile.urlHistory.clearForward();
                    window.history.back();
                }
            } catch( ex ) {
                unregister();
            }
        }
    }
    
    // add eventListener for tizenhwkey (Back Button)
    document.addEventListener( 'tizenhwkey', backEvent );
    backEventListener = backEvent;
};
</pre>

####图片文件
<pre>
var imageData = 'data:image/gif;base64,R0lGODlhXwATANUAAAAAAP////8AAP7+/v39/fz8/Pr6+vn5+fDw8Ovr6+rq6uXl5eHh4eDg4Nzc3NnZ2djY2NXV1dTU1NPT09LS0s3NzczMzMvLy8rKysfHx8bGxsDAwL29vbCwsK2traGhoaCgoJ+fn56enpycnJmZmZiYmP///wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACH5BAEAACYALAAAAABfABMAAAb/wIBwSCwKDxGjkohcOpGmqHRKrVqvUeGAMOh6v92Co6Plgs9icmB7Ro+x8Ph1jWAs7vj8QvHwjOh2enp8foCCg30jcouLawwaFZGSkxYSGyQhjpCTnJWXmQOPnJ2WmFgAjFOocwMLFhOwsbITDxkhoK6zurW3a7m6srwhp3ICAqpTxlK+r8CwFBC2uM2xxrHQ0syyANwAz9G3VKjd3CarVsrkqMrLrRXOExQUDxq97tXGAs/z9bjvsNwAApBHL5w4deeuGFPHrt2Cf8AI9vP1L5+1ePzsPYwVcAI3iQalcBOiDg4AYwG4LaRCER6sgv7wXXw58Z6ujjAPkiRn8qQA/5/lprSUJS9YzY2yZtI6CvFZx6UhEUqtwnAhzyw2r1EwqrFpPG+xYA7lCJZmyChSg17plq8by6zOxMIFyDXmtrJQh6lK23Oh1beuJGAEJjcwRnJhj1YavKsfVYTFfqo8Btgr0Yx202IeK3BfTmQlE1axypayQ69FMUKoB2Ko5tUhWs/tKA82CGKRRUppaGJNAloPgj+AAEE4hAYYYvsGbrx48OPJZf8W/tz5cOSxccPhnbBhAAMPNmgYT768hgsclIMXb948evXh27tPfzuV/SoHHHgocat/fxC3iBAbCPnt559/AIYgIAgE6sffgbckuGB991VoQgQdjBAhhAkO+BUBhhrGxuF/IHyY4YYHdsjgBxZaGAQAOw==';

</pre>

####面板初始化
<pre>
(function (win, doc) {
  var Config = {
    level: 0,//初始化时的级别
    size: 20,//每个方块大小
    btnWidth: 80,//按钮宽度
    btnHeight: 30,//按钮高度
    imageData: win['imageData']//图片base64数据
  };
  var Color = {
    bg: '#ddd',//空白颜色
    border: ['#888'/*正常颜色*/, '#ff0000'/*hover颜色*/],//方块边框颜色
    btnBg: '#00ffff',//按钮背景
    btnFont: '#ff0000',//按钮字体颜色
    mask: 'rgba(0,0,0,0.8)',//遮罩背景颜色，半透明
    number: ['#0000ff', '#008000', '#ff0000', '#000080', '#800000', '#008080', '#000000', '#808080'],//数字颜色
    bombBg: '#ff0000',//点击使爆炸游戏结束的方块背景
    overFont: '#ffffff',//结束文字颜色
  };
  var levels = [
    {
      name: '初级',//级别名称
      x: 9,//水平方向方块格式
      y: 9,//垂直方向方块个数
      mines: 10//雷的数量
    },
    {
      name: '中级',
      x: 13,
      y: 13,
      mines: 30
    },
    {
      name: '高级',
      x: 16,
      y: 16,
      mines: 40
    }
  ];
  var mineSweeping,//类实例
    level,//当前级别
    lastMines,//剩余雷数
    lastHidden,//剩余未被揭开的方块数
    canvas,//canvas对象
    flagButton,//开或标记
    context,//canvas上下文
    image,//img对象
    matrix,//二维数组
    currentFlag,
    openMatrix=1,//单击
    flagMatrix=2;//双击
    var MineSweeping = function() {
      level = levels[Config.level];
      canvas = doc.getElementsByTagName('canvas')[0];
      flagButton=doc.getElementById("flagButton");
      context = canvas.getContext('2d');
      canvas.style.background = Color.bg;
      this.infos = doc.getElementsByTagName('dd');
      this.status = 'stopped';
      currentFlag=openMatrix;
      this.createImage(function() {
        mineSweeping.createLevels();
        mineSweeping.init();
        mineSweeping.bindEvent();
      });
    };
</pre>

#### 绑定事件
<pre>
    bindEvent: function() {
      canvas.addEventListener('click', function(e) {
    	        if(mineSweeping.status === 'stopped') {
    	            mineSweeping.stoppedClickHandler(e);
    	          } else {
    	            mineSweeping.startedClickHandler(e);
    	          }
      });   
      flagButton.addEventListener('click',function(e){
    	  if(currentFlag==openMatrix){
    		  currentFlag=flagMatrix;
    		  flagButton.innerHTML="标记";
    	  }else {
			currentFlag=openMatrix;
			 flagButton.innerHTML="开";
		}
      });
      doc.querySelector('select').addEventListener('change', function() {
        //改变当前的级别，并重新初始化
        level = levels[parseInt(this.value)];
        mineSweeping.init();
      });
    },
</pre>

####清除画布
<pre>
   clear: function() {
      context.clearRect(0, 0, canvas.width, canvas.height);
    },
</pre>
   
####加载图片
 <pre>
    createImage: function(fn) {
      image = new Image();
      image.onload = function() {
        fn && fn();
      };
      image.src = Config.imageData;
    },
</pre>
    
####创建级别下拉列表
   <pre>
    createLevels: function() {
      var select = doc.querySelector('select');
      var frag = doc.createDocumentFragment();//用文档碎片来存储
      for(var i = 0, len = levels.length; i< len; i++) {
        var option = doc.createElement('option');
        option.value = i;
        option.innerHTML = levels[i].name;
        frag.appendChild(option);
      }
      select.appendChild(frag);
    },
</pre>
   
####创建二维数组，用来存储各种类型对象
    <pre>
    createMatrix: function() {
      matrix = [];
      for(var i = 0; i < level.y; i++) {
        matrix[i] = [];
        for(var j = 0; j < level.x; j++) {
          //0-否 >=1-是
          matrix[i][j] = {
            hidden: 1,//是否隐藏的
            number: 0,//数字
            flag: 0,//旗子
            doubt: 0,//问号
            bomb: 0,//炸弹 0-没有 1-正常炸弹 2-标记错误的炸弹 3-被点击后造成游戏结束的炸弹
            hover: 0//hover样式
          };
        }
      }
    },
</pre>
####游戏结束时显示所有的方块
<pre>
    createMatrixForOver: function() {
      for(var i = 0; i < level.y; i++) {
        for(var j = 0; j < level.x; j++) {
          var m = matrix[i][j];
          m.hidden = 0;
          //如果当前格子没有炸弹，但是却标了红旗，表示错误的标法，同时把旗子去掉
          if(m.bomb === 0 && m.flag === 1) {
            m.flag = 0;
            m.bomb = 2;
          }
        }
      }
    },
</pre>

####随机布雷，1为有雷
 <pre>
    createMines: function(count) {
      while(count > 0) {
        var i = Math.ceil(Math.random() * (level.y - 1));
        var j = Math.ceil(Math.random() * (level.x - 1));
        if(matrix[i][j].bomb !== 1) {
          matrix[i][j].bomb = 1;
          count--;
        }
      }
    },
</pre>

####创建数字，根据方格周围的雷数量统计
<pre>
    createNumbers: function() {
      for(var i = 0; i < level.y; i++) {
        for(var j = 0; j < level.x; j++) {
          //确保当前格子不是雷
          if(matrix[i][j].bomb === 0) {
            i > 0 && j > 0 && matrix[i-1][j-1].bomb === 1 && matrix[i][j].number++;//左上方
            i > 0 && matrix[i-1][j].bomb === 1 && matrix[i][j].number++;//正上方
            i > 0 && j < level.x - 1 && matrix[i-1][j+1].bomb === 1 && matrix[i][j].number++;//右上方
            j < level.x - 1 && matrix[i][j+1].bomb === 1 && matrix[i][j].number++;//正右方
            i < level.y - 1 && j < level.x - 1 && matrix[i+1][j+1].bomb === 1 && matrix[i][j].number++;//右下方
            i < level.y - 1 && matrix[i+1][j].bomb === 1 && matrix[i][j].number++;//正下方
            i < level.y - 1 && j > 0 && matrix[i+1][j-1].bomb === 1 && matrix[i][j].number++;//左下方
            j > 0 && matrix[i][j-1].bomb === 1 && matrix[i][j].number++;//正左方
          }
        }
      }
    },
</pre>

####创建计时
     <pre>
    createStartTimer: function() {
      this.startTime += 1e3;//当前时间加1s
      this.infos[2].innerHTML = this.startTime/1e3 +'秒';
      this.startTimer && clearTimeout(this.startTimer);//先清除一下，避免重复打乱节奏
      this.startTimer = setTimeout(function() {
        mineSweeping.createStartTimer();
      }, 1e3);
    },
</pre>

####绘制各种形状
   <pre>
    draw: function() {
      this.clear();//首先清除画布，重新绘制
      for(var i = 0; i < level.y; i++) {
        for(var j = 0; j < level.x; j++) {
          var m = matrix[i][j];
          var size = Config.size;
          var x = j * size, y = i * size;
          if(m.hidden === 1) {
            //隐藏的就绘制隐藏的icon
            this.drawImage(x, y, 0);
          } else if(m.bomb === 2 || m.bomb === 3) {
            //正常炸弹
            if (m.bomb === 3) {
              //游戏结束时会有一个产生结束的肇事者炸弹，将其背景变红
              context.fillStyle = Color.bombBg;
              context.fillRect(x, y, size, size);
            }
            this.drawImage(x, y, 4);
          } else if(m.bomb === 1) {
            //标记错误的炸弹
            this.drawImage(x, y, 3);
          } else if(m.number > 0) {
            //数字
            this.drawNumber(m.number, x, y);
          }
          if(m.doubt === 1) {
            //问号
            this.drawImage(x, y, 2);
          } else if(m.flag === 1) {
            //旗子
            this.drawImage(x, y, 1);
          }
          //绘制边框
          this.drawBorder(x, y, m.hover);
        }
      }
   
    },
</pre>

#### 绘制边框
<pre>
     * @param x x坐标
     * @param y y坐标
     * @param color 颜色序号，0-正常，1-hover
    drawBorder: function(x, y, color) {
      context.beginPath();
      context.strokeStyle = Color.border[color];
      context.rect(x, y, Config.size, Config.size);
      context.closePath();
      context.stroke();
    },
</pre>

####绘制图形
<pre>
     * @param x x坐标
     * @param y y坐标
     * @param sx 裁剪位置序号
    drawImage: function(x, y, sx) {
      context.drawImage(image, sx*(Config.size-1), 0, Config.size-1, Config.size-1, x, y, Config.size, Config.size);
    },
</pre>

####绘制数字
<pre>
     * @param number
     * @param x
     * @param y
    drawNumber: function(number, x, y) {
      context.textAlign = 'center';
      context.textBaseline = 'middle';
      context.font = '14px Arial bold';
      context.fillStyle = Color.number[number-1];
      context.fillText(number, x+Config.size/2, y+Config.size/2);
    },
</pre>

####绘制结束屏幕
<pre>
     * @param isPassed 是否通关造成的结束
    drawOver: function(isPassed) {
      var tipsText = isPassed ? 'GAME PASS!' : 'GAME OVER!';
      context.fillStyle = Color.mask;
      context.fillRect(0, 0, canvas.width, canvas.height);
      context.fillStyle = Color.btnBg;
      context.fillRect(canvas.width/2-Config.btnWidth/2, canvas.height/2-Config.btnHeight/2, Config.btnWidth, Config.btnHeight);
      context.textAlign = 'center';
      context.textBaseline = 'middle';
      context.font = '20px Arial';
      context.fillStyle = Color.btnFont;
      context.fillText('再来一次', canvas.width/2, canvas.height/2+Config.btnHeight/2-20);
      context.fillStyle = Color.overFont;
      context.fillText(tipsText, canvas.width/2, canvas.height/2-Config.btnHeight);
    },
</pre>

####初始化
<pre>
    init: function() {
      lastMines = level.mines;
      lastHidden = level.x * level.y;
      canvas.width = level.x * Config.size;
      canvas.height = level.y * Config.size;
      this.infos[1].innerHTML = lastMines;
      this.status = 'inited';
      this.startTimer && clearTimeout(this.startTimer);
      this.infos[2].innerHTML = '0秒';
      this.createMatrix();
      this.createMines(lastMines);
      this.createNumbers();
      this.draw();
    },
</pre>

#### 打开方格
<pre>
     * @param x x坐标
     * @param y y坐标
    openBlock: function(x, y) {
      var m = matrix[y][x];
      if(m.bomb === 1) {
        //如果有炸弹
        if(this.status === 'started') {
          //此时游戏已经开始了，因此要直接结束游戏
          m.bomb = 3;
          this.over();
        } else if(this.status === 'inited') {
          //此时游戏刚初始化，避免第一次点击就碰着雷，所以重新初始化，知道没碰到雷位置
          this.init();
          this.openBlock(x, y);
        }
      } else if(m.flag === 1) {
        //如果有旗子的话，会造成游戏结束，此种情况可能发生在自动打开周围方格时
        m.hidden = 0;
        m.flag = 0;
        m.bomb = 2;
        this.over();
      } else {
        //如果有数字,正常情况是直接显示数字
        m.hidden = 0;
        if(m.doubt === 1) {
          //如果是问号，则把问号去掉
          m.doubt = 0;
        }
        lastHidden--;//剩余未被开启的格子减1
        if(m.number === 0) {
          //如果是空白的话则把周围的都显示出来
          x > 0 && y > 0 && matrix[y-1][x-1].hidden === 1 && this.openBlock(x-1, y-1);
          y > 0 && matrix[y-1][x].hidden === 1 && this.openBlock(x, y-1);
          y > 0 && x < level.x - 1 && matrix[y-1][x+1].hidden === 1 && this.openBlock(x+1, y-1);
          x < level.x - 1 && matrix[y][x+1] === 1 && this.openBlock(x+1, y);
          y < level.y - 1 && x < level.x - 1 && matrix[y+1][x+1].hidden === 1 && this.openBlock(x+1, y+1);
          y < level.y - 1 && matrix[y+1][x].hidden === 1 && this.openBlock(x, y+1);
          y < level.y - 1 && x > 0 && matrix[y+1][x-1].hidden === 1 && this.openBlock(x-1, y+1);
          x > 0 && matrix[y][x-1].hidden === 1 && this.openBlock(x-1, y);
        }
      }
    },
</pre>

####打开方格周围的方格
<pre>
     * @param x
     * @param y
    openAroundBlock: function(x, y) {
      var flagCount = 0,//周围旗子统计
        hiddenCount = 0;//周围隐藏的格子统计
      for(var i = y - 1; i <= y + 1; i++) {
        for(var j = x - 1; j <= x + 1; j++) {
          //在边界的话有可能索引报错
          try {
            if(matrix[i][j].flag === 1) {
              flagCount++;
            }
            if(matrix[i][j].hidden === 1) {
              hiddenCount++;
            }
          } catch (ex) {}
        }
      }
</pre>

#### 如果旗子数等于周围的炸弹数，并且有未标记旗子的隐藏格子
    <pre>
      if(flagCount === matrix[y][x].number && hiddenCount > flagCount) {
        for(i = y - 1; i <= y + 1; i++) {
          for(j = x - 1; j <= x + 1; j++) {
            try {
              if(matrix[i][j].bomb === 0 && matrix[i][j].hidden === 1) {
                //没有炸弹，并且是隐藏的才打开格子
                this.openBlock(j, i);
              }
            } catch (ex) {}
          }
        }
      }
    },
    </pre>

#### 游戏结束
<pre>
     * @param isPassed 是否通关造成的结束

    over: function(isPassed) {
      this.status = 'stopped';
      this.startTimer && clearTimeout(this.startTimer);
      this.createMatrixForOver();
      this.draw();
      this.drawOver(isPassed);
    },
</pre>

#### 开始后点击事件回调
<pre>
     * @param e
    startedClickHandler: function(e) {
      var x = Math.floor(e.offsetX/Config.size),
        y = Math.floor(e.offsetY/Config.size);
      var m = matrix[y][x];
      if(this.status === 'started') {//游戏已经开始
        if(currentFlag === openMatrix) {//开
          if(m.hidden === 1 && m.flag === 0) {
            //格子是隐藏的并且未标记旗子则直接打开
            this.openBlock(x, y);
          } else if(m.hidden === 0) {
            //格子已经显示，则开启周围的格子，windows上是双击触发这个效果
            this.openAroundBlock(x, y);
          }
        } else if(currentFlag === flagMatrix && m.hidden === 1) {//标记鸽子
          if(m.flag === 1) {
            //如果有旗子，则把旗子换成问号，同时炸弹和隐藏格子数量增加
            m.flag = 0;
            m.doubt = 1;
            lastMines++;
            lastHidden++;
          } else if(m.doubt === 1) {
            //如果是问号，则直接去掉问号
            m.doubt = 0;
          } else {
            //如果不是问号，也不是旗子，则标记旗子
            m.flag = 1;
            lastMines--;
            lastHidden--;
          }
        }
        if(lastMines === lastHidden) {
          //剩余的雷和剩余未被揭开的格子相等时游戏结束，传一个true过去代表通关造成的结束
          this.over(true);
        }
      } else if(this.status === 'inited') {//游戏刚刚初始化
        if(e.which === 1) {
          //如果是鼠标右键
          this.openBlock(x, y);
          this.status = 'started';
          this.startTime = 0;
          this.createStartTimer();//开始计时
        }
      }
      this.infos[1].innerHTML = lastMines;//修改剩余雷数
      if(this.status !== 'stopped') {
        //游戏开始后才进行绘制
        this.draw();
      }
    },
</pre>

####  结束后的点击事件回调
<pre>
     * @param e
     */
    stoppedClickHandler: function(e) {
      var x = e.offsetX,
        y = e.offsetY;
      //坐标在按钮范围内则点击有效
      if(x > (canvas.width-Config.btnWidth)/2
        && x < (canvas.width+Config.btnWidth)/2
        && y > (canvas.height-Config.btnHeight)/2
        && y < (canvas.height+Config.btnHeight)/2) {
        this.init();
      }
    }
  };
  mineSweeping = new MineSweeping();
})(window, document);
</pre>
