<script lang="ts">
    import { onMount } from 'svelte';
    import * as PIXI from 'pixi.js';
  
  let app: PIXI.Application;
  let board: Array<Array<{ hasMine: boolean, status: string }>> = [];
  const rows = 10;
  const columns = 10;
  const mineCount = 10;
  const cellSize = 50;
  const cellColor = 0x808080; // 灰色
  const openCellColor = 0xC0C0C0; // 打开的单元格的颜色
  let gameOver = false;
  let currentScore = 0;
  // image of flag
  const FLAG_IMAGE_BASE64 = 'data:image/png;base64,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'
  // image of bomb
  const BOMB_IMAGE_BASE64 = 'data:image/png;base64,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'
  onMount(() => {
    app = new PIXI.Application<HTMLCanvasElement>({
      width: columns * cellSize,
      height: rows * cellSize,
      backgroundColor: 0xEEEEEE,
    });


    (app.view as HTMLCanvasElement).oncontextmenu = (e) => {
  e.preventDefault();
};

  (document.getElementById('game-board') as HTMLElement).appendChild(app.view as HTMLCanvasElement);


    initializeBoard();
    drawBoard();
    app.stage.interactive = true;
    app.stage.on('pointerdown', onCellClick);
  });
  function onCellClick(event: any ) {
    const isRightClick = event.data.button === 2; // 检查是否为右键点击
    const position = event.data.getLocalPosition(app.stage);
    const x = Math.floor(position.x / cellSize);
    const y = Math.floor(position.y / cellSize);

    if (x >= 0 && x < columns && y >= 0 && y < rows) {
        if (isRightClick) {
            toggleFlag(x, y); // 右键点击时，切换标记状态
        } else {
            revealCell(x, y); // 左键点击时，揭示单元格
        }
        drawBoard(); // 更新画板
    }
}
function toggleFlag(x: number, y: number) {
  if (board[y][x].status === 'closed') {
        board[y][x].status = 'flagged'; // 如果单元格关闭，将其标记
    } else if (board[y][x].status === 'flagged') {
        board[y][x].status = 'closed'; // 如果单元格已标记，取消标记
    }
    // 如果单元格已经打开，不做任何操作
}
    function initializeBoard() {
      for (let y = 0; y < rows; y++) {
        board[y] = [];
        for (let x = 0; x < columns; x++) {
          board[y][x] = {
            hasMine: false,
            status: 'closed',
          };
        }
      }
  
      // 随机放置地雷
      let minesPlaced = 0;
      while (minesPlaced < mineCount) {
        const x = Math.floor(Math.random() * columns);
        const y = Math.floor(Math.random() * rows);
  
        if (!board[y][x].hasMine) {
          board[y][x].hasMine = true;
          minesPlaced++;
        }
      }
    }
    function drawBoard() {
      const graphics = new PIXI.Graphics();

      // 清除之前的绘制内容
      app.stage.removeChildren();

      // 绘制网格
      for (let y = 0; y < rows; y++) {
        for (let x = 0; x < columns; x++) {
          const cell = board[y][x];
          let color;

          switch (cell.status) {
            case 'closed':
              color = cellColor;
              break;
            case 'mine':
                const bombTexture = PIXI.Texture.from(BOMB_IMAGE_BASE64);
                const bombSprite = PIXI.Sprite.from(bombTexture);
                bombSprite.x = x * cellSize;
                bombSprite.y = y * cellSize;
                bombSprite.width = cellSize; // 调整精灵的大小以适应单元格
                bombSprite.height = cellSize;
                app.stage.addChild(bombSprite);
                continue;
            case 'flagged':
              // 如果单元格被标记，我们将添加一个小旗子的精灵而不是填充颜色
              const flagTexture = PIXI.Texture.from(FLAG_IMAGE_BASE64);
              const flagSprite = PIXI.Sprite.from(flagTexture);
              flagSprite.x = x * cellSize;
              flagSprite.y = y * cellSize;
              flagSprite.width = cellSize; // 调整精灵的大小以适应单元格
              flagSprite.height = cellSize;
              app.stage.addChild(flagSprite);
              continue; // 跳过剩下的绘制代码
            case 'open':
                const adjacentMines = getAdjacentMines(x, y);
                if (adjacentMines > 0) {
                console.log(adjacentMines);
                const textStyle = new PIXI.TextStyle({
                fill: 0x00000,
                fontSize: cellSize / 2, // 调整字体大小以适应单元格
                align: 'center'
              });

              // 创建一个文本对象
              const text = new PIXI.Text(adjacentMines.toString(), textStyle);
              text.x = x * cellSize + cellSize / 2; // 更新位置以居中文本
              text.y = y * cellSize + cellSize / 2;
              text.anchor.set(0.5); // 居中文本
              
              // 将文本添加到舞台
              app.stage.addChild(text);}
              
              continue;
            default:
              color = openCellColor;
              PIXI.LineStyle;
          }

          graphics.beginFill(color);
          graphics.drawRect(x * cellSize, y * cellSize, cellSize, cellSize);
          graphics.lineStyle(1, 0x000000, 1);
          graphics.endFill();
          
          // 绘制单元格边框（分割线）


          
        }
      }

      // 添加新的绘制内容
      app.stage.addChild(graphics);
}
function revealAllMines() {
  drawBoard(); 
  for (let y = 0; y < rows; y++) {
    for (let x = 0; x < columns; x++) {
      if (board[y][x].hasMine) {
        board[y][x].status = 'mine';
                // 创建一个新的精灵
        const bombTexture = PIXI.Texture.from(BOMB_IMAGE_BASE64);
        const bombSprite = PIXI.Sprite.from(bombTexture);
        bombSprite.x = x * cellSize;
        bombSprite.y = y * cellSize;
        bombSprite.width = cellSize; // 调整精灵的大小以适应单元格
        bombSprite.height = cellSize;
        app.stage.addChild(bombSprite);
      }
    }
  }
}
function restartGame() {
  gameOver = false;
  currentScore = 0;
  initializeBoard();
  drawBoard();
}
function showResult() {
  let gameOverPopup = document.getElementById('gameOverPopup');
    let restartButton = document.getElementById('restartButton');
    if (gameOverPopup) gameOverPopup.style.display = 'flex';
    if (restartButton) {
        restartButton.addEventListener('click', function() {
            // 这里可以放置重新开始游戏的逻辑
            if (gameOverPopup) gameOverPopup.style.display = 'none'; // 隐藏弹窗
             restartGame(); // 假设你有一个重新开始游戏的函数
            
          });
    } 
}
function revealCell(x: number , y: number ) {
        if(gameOver){showResult();
          return ;
      }
        if (board[y][x].status !== 'closed') {
            return;
        }
        // 如果是地雷，设置游戏状态为失败
        if (board[y][x].hasMine) {
            gameOver = true;
            revealAllMines(); 
            showResult();
            return;
        }
        board[y][x].status = 'open';
        let openCells = 0;
        for (let y = 0; y < rows; y++) {
            for (let x = 0; x < columns; x++) {
                if (board[y][x].status === 'open') {
                    openCells++;
                }
            }
        }
        currentScore = openCells;
        const adjacentMines = getAdjacentMines(x, y);
        if (adjacentMines > 0) {
          console.log(adjacentMines);
          const textStyle = new PIXI.TextStyle({
          fill: 0xffffff,
          fontSize: cellSize / 2, // 调整字体大小以适应单元格
          align: 'center'
        });

        // 创建一个文本对象
        const text = new PIXI.Text(adjacentMines.toString(), textStyle);
        text.x = x * cellSize + cellSize / 2; // 更新位置以居中文本
        text.y = y * cellSize + cellSize / 2;
        text.anchor.set(0.5); // 居中文本

        // 将文本添加到舞台
        app.stage.addChild(text);
    
  } else {
    // 如果没有相邻地雷，递归揭示周围的单元格
    for (let yOffset = -1; yOffset <= 1; yOffset++) {
      for (let xOffset = -1; xOffset <= 1; xOffset++) {
        if (xOffset === 0 && yOffset === 0) continue; // 跳过中心单元格

        const newX = x + xOffset;
        const newY = y + yOffset;

        if (newX >= 0 && newX < columns && newY >= 0 && newY < rows) {
          revealCell(newX, newY);
        }
      }
    }
  }

  checkWin();
}

function getAdjacentMines(x: number, y: number) {
  let count = 0;
  for (let yOffset = -1; yOffset <= 1; yOffset++) {
    for (let xOffset = -1; xOffset <= 1; xOffset++) {
      if (xOffset === 0 && yOffset === 0) continue;

      const newX = x + xOffset;
      const newY = y + yOffset;

      if (newX >= 0 && newX < columns && newY >= 0 && newY < rows) {
        if (board[newY][newX].hasMine) {
          count++;
        }
      }
    }
  }
  return count;
}

function checkWin() {
  // 遍历所有单元格
  for (let y = 0; y < rows; y++) {
    for (let x = 0; x < columns; x++) {
      if (board[y][x].status === 'closed' && !board[y][x].hasMine) {

        return;
      }
    }
  }

  let winPopup = document.getElementById('winPopup');
    let restartButton = document.getElementById('WinrestartButton');
    if (winPopup) winPopup.style.display = 'flex';
    if (restartButton) {
        restartButton.addEventListener('click', function() {
            // 这里可以放置重新开始游戏的逻辑
            console.log("restart");
            if (winPopup) winPopup.style.display = 'none'; // 隐藏弹窗
             restartGame(); // 假设你有一个重新开始游戏的函数
            
          });
    } 
}
      
    
</script>

<main>
  <div id="game-board" />
  <div id="bottom-bar">
    
    <p id="score">Score: <b>{currentScore}</b></p>
  
        
</main>

<!-- HTML结构 -->
<div id="gameOverPopup" class="popup">
  <div class="popup-content">
      <h2>Game Over!</h2>
      <button id="restartButton" class="restart-button">Restart Game</button> <!-- 添加类以便样式化 -->
  </div>
</div>
<div id="winPopup" class="popup">
  <div class="popup-content">
      <h2>You Win!</h2>
      <button id="WinrestartButton" class="restart-button">Restart Game</button> <!-- 添加类以便样式化 -->
  </div>
</div>

<style>
  /* 基本布局调整 */
  main {
    display: flex;
    flex-direction: column; /* 垂直排列元素 */
    justify-content: center;
    align-items: center;
    height: 100vh; /* 使主容器占满全屏高度 */
    background: #ecf0f1; /* 轻亮的背景色 */
    font-family: 'Arial', sans-serif; /* 使用无衬线字体 */
  }

  #game-board {
    flex-grow: 1; /* 允许游戏板块在可用空间中增长 */
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #bottom-bar {
    width: 100%; /* 底栏全宽 */
    padding: 10px;
    background: #34495e; /* 深蓝色背景 */
    color: white; /* 白色文字 */
    text-align: center;
  }

  /* 弹窗样式优化 */
  .popup {
    display: none;
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.8); /* 暗一点的背景色，更好地凸显弹窗内容 */
    justify-content: center;
    align-items: center;
    z-index: 1000; /* 确保弹窗在最顶层 */
    backdrop-filter: blur(5px); /* 轻微模糊背景 */
  }

  .popup-content {
    background: rgba(255, 255, 255, 0.95); /* 略微透明的白色背景 */
    padding: 40px;
    border-radius: 15px; /* 更圆润的边角 */
    text-align: center;
    box-shadow: 0px 0px 15px rgba(0, 0, 0, 0.2); /* 轻微的阴影效果 */
    max-width: 400px; /* 弹窗最大宽度 */
    width: 100%; /* 宽度自适应 */
  }

  .popup-content h2 {
    margin-bottom: 20px;
    font-size: 24px;
    color: #333;
    margin-top: 0; /* 移除顶部边距 */
  }

  .restart-button {
    background-color: #e74c3c; /* 强烈红色，引起注意 */
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 25px; /* 圆形按钮 */
    cursor: pointer;
    font-size: 18px;
    transition: all 0.3s; /* 过渡效果 */
    display: inline-block; /* 避免按钮宽度过长 */
    text-decoration: none; /* 移除下划线 */
  }

  .restart-button:hover {
    background-color: #c0392b; /* 深红色 */
    transform: scale(1.1); /* 鼠标悬停时放大 */
  }
</style>