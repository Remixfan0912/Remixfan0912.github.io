---
title: Cloudy Game教學細項

---

[TOC]
## 閱讀文檔須知
- 程式須知
    - 程式區塊的標題會註記該程式應該寫在哪個檔案。
    - 角色的函式撰寫都放置於class內部，因此我不會特別呈現class架構。
## 著作聲明
- 聲明
    - 著作權屬Rix所有，內容由Rix撰寫，模板由Gemini 3.5提供。
    - 此為中山大學資工營課務組 - 專案教學文檔，僅提供閱讀及參考。
## 角色介紹、架構展示
### 基本角色  Player、Platform、Pickups
#### Player架構
- 初始化__init__
    - 繼承Sprite
    - 載入圖檔
    - 渲染玩家、定位
    - 圖檔格式化
    - 玩家position(處理float格式)、速度
    - 處理玩家能力
    - 玩家是否站在平台布林
    - 能力增益List
- update()
    - 處理流程  玩家位置變換(x) - 能力增益ONOFF - 鍵盤偵測 - 重力 - 平台碰撞 - 道具碰撞 - 角色動畫

#### Platform架構
- 類別變數
    - 第一個平台生成
    - 垂直移動速度
- 初始化__init__
    - 繼承Sprite
    - 引入圖檔List、定位
    - 設定速度
    - 平台加速計時
- update()
    - 處理流程  邊界偵測 - 速度變換 - 位置變換 - 超出高度自毀

#### Pickups架構
- 初始化__init__
    - 繼承Sprite
    - 引入道具圖檔
    - 道具追蹤平台
    - 定位道具(同步平台)
- update()
    - 處理流程  位置變換 - 超出高度自毀

---
### 主檔案  main
#### main架構  
- pygame初始化
- 遊戲資料初始化
- 引入字體、圖片檔、音檔
- 建立Sprite群組
- Sprite角色物件生成加群組
- 管理平台生成、管理道具生成函式
- 遊戲迴圈  
    - 設定clock
    - 取得鍵盤輸入
    - 切分遊戲Menu&遊戲Start
    - 遊戲Menu
        - 渲染Menu(背景+data顯示)
    - 遊戲Start
        - 玩家死亡  更新data、清除群組、初始化、重新生成sprite
        - 分數、時間開始計算
        - 遊戲時間計算
        - sprite.update()
        - 渲染Start
    - 總畫面更新
---
### 常數設定  Settings
#### Settins架構
- 首先，常數檔全都是變數
    - 我分成幾個區塊 
        - 遊戲本體常數
        - 玩家常數
        - 平台常數
        - 道具常數
---
## pygame架構建置
### 建立基本pygame架構
- 🌟畫面初見！🌟
    - 首先先建立Settings常數檔案，專門處理像是長寬、顏色等
    ```python=
    # Settings.py
    HEIGHT = 760
    WIDTH = 1280
    FPS = 60
    
    BLACK = (0,0,0)
    WHITE = (255,255,255)
    RED = (255,0,0)
    GREEN = (0,255,0)
    BLUE = (0,0,255)
    ```
    - 再來建立畫面呈現
    ```python=
    # main.py
    import pygame
    from Settings import *

    pygame.init()
    pygame.display.set_caption("My first game")
    screen = pygame.display.set_mode((WIDTH, HEIGHT))
    clock = pygame.time.Clock()
    running = True

    while running:
        clock.tick(FPS)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
                
        # 畫面渲染
        screen.fill(BLACK)
        pygame.display.flip()
        
    pygame.quit()
    ```
- 🌟Player初見！🌟
    - 首先先設定「玩家」
    ```python=
    # Player.py
    import pygame

    class Player(pygame.sprite.Sprite):
        def __init__(self):
            super().__init__()
        def update(self):
            pass
    ```
    - 接著在main中引入Player group
    ```python=
    # main.py
    players = pygame.sprite.Group()
    # 這邊再建立一個Group儲存後續所有sprites
    all_sprites = pygame.sprite.Group()
    
    player1 = Player()
    players.add(player1)
    # 將群組加入另外一個群組
    all_sprites.add(players)
    
    while running ...
    ```
    
### Player角色的類別撰寫
- 🌟Player顯示！🌟
    - 為使Player顯示在畫面當中，我們以方塊作為玩家，之後再加上圖檔！
    ```python=
    # Player.py
    class Player(pygame.sprite.Sprite):
        def __init__(self):
            super().__init__()
            self.image = pygame.Surface((20,20))
            self.image.fill(BLUE)
            self.rect = self.image.get_rect()
            self.rect.centerx = WIDTH/2
            self.rect.bottom = 15 # 設這個位置有它的意義，之後你會了解的！
        def update(self):
            pass
    ```
    - 接著在main.py中以群組繪製出角色，同時先寫好更新的呼叫
    ```python=
    # main.py
    # ...省略
    while running:
        clock.tick(FPS)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
        
        # 角色更新
        players.update() # <--這行
                
        # 畫面渲染
        screen.fill(BLACK)
        all_sprites.draw(screen) # <--這行
        pygame.display.flip()

    pygame.quit()    
    ```
- 🌟Player移動！🌟
    - 接著我們讓玩家可以按鍵盤來左右的移動
        - 我們要先設定玩家的 `x_speed`
        ```python=
        # Settings.py
        # 玩家資訊
        PLAYER_X_SPEED = 5
        ```
        - 接著我們要設定玩家的位置資訊、速度資訊
        ```python=
        # Player.py
        def __init__(self):
            # ...省略
            # 為何要使用pos而不直接用rect存的center就好？
            # 因為 -- rect所存的座標都是「整數」，但有時玩家移動速度可能被設定成小數點
            # 所以 -- 我設定了pos來存取玩家的「精確座標」。
            # 儘管最後還是指派給rect，但可以更好抓取玩家的移動幀數
            self.pos = [float(self.rect.centerx), float(self.rect.centery)]
            self.velocity = [0,0]
            # 玩家資訊
            self.x_speed = PLAYER_X_SPEED
        ```
        - 接著要來寫偵測鍵盤的事件
        ```python=
        # Player.py
        # ...省略
        def key_detection(self):
            kp = pygame.key.get_pressed()
            if kp[pygame.K_LEFT] : self.rect.centerx += self.x_speed*-1
            if kp[pygame.K_RIGHT] : self.rect.centerx += self.x_speed
        ```
- 🌟Player重力！🌟
    - 設定地面  我們先設定邊界(border)暫時偵測地面
        - 定義一個border函式  
        ```python=
        # Player.py
        def border(self):
            if self.rect.bottom > HEIGHT:
                self.rect.bottom = HEIGHT
                self.pos[1] = self.rect.centery
                self.velocity[1] = 0
        ```
    - 再設定重力常數
        ```python=
        # Settings.py
        G = 0.3
        ```
    - 接著我們就可以來寫重力函式啦
        ```python=
        # Player.py
        def gravity(self):
            self.velocity[1] += G
            self.pos[1] += self.velocity[1]
            self.rect.centery = self.pos[1]
        ```
- 🌟Player的跳躍！🌟
    - 設定'jump'函式  由於我們有self.pos + self.gravity，接著就可以加上pos來達到跳躍的物理效果。
    - 先來設定檔設定玩家的跳躍初速度
        ```python=
        # Settings.py
        # ...省略
        PLAYER_JUMP_V = -14
        ```
    - 再來要設定玩家的jump函式
        ```python=
        # Player.py
        def __init__(self):
            # ...省略
            self.jump_v = PLAYER_JUMP_V
        def jump(self):
            # 若玩家速度小於0  亦即在往上跳的過程
            # 禁止連跳，因此直接return中止函式執行。
            if self.velocity[1] < 0:
                return 
            
            # 增加玩家Y速度為我們在Settings設定的數值
            self.velocity[1] += self.jump_v
            
            # 再來更新pos的座標，每幀加一次速度
            self.pos[1] += self.velocity[1]
            
            # 更新實際玩家的位置（若沒有這行，不會正確更新到物件）
            self.rect.centery = self.pos[1]
        ```
    - 接著到key_detection的部分偵測，若玩家按下「上鍵」，則觸發jump函式
        ```python=
        # Player.py
        def key_detection(self):
            kp = pygame.key.get_pressed()
            ...
            if kp[pygame.K_UP] : self.jump()
        ```
    - 最後結合border、gravity、jump、key_detection，放進update去！
        - 注意順序 -- 每次執行會先使角色移動函式執行完畢再做border！
        ```python=
        # Player.py
        def update(self):
            self.gravity()
            self.key_detection()
            self.border()
        ```
    
### Platform角色的類別撰寫
- 🌟Platform的生成🌟
    - 與Player相同，我們利用self.image及self.rect來繪製、定位sprite
    ```python=
    # Platform.py
    import pygame
    from Settings import *

    class Platform(pygame.sprite.Sprite):
        def __init__(self):
            super().__init__()
            self.image = pygame.Surface((150,20))
            self.image.fill(WHITE)
            self.rect = self.image.get_rect()
            self.rect.centerx = WIDTH/2
            self.rect.top = 15 # 有沒有發現平台的top跟玩家的bottom接在一起了！

        def update(self):
            pass
    ```
    - 由於平台都從同一個位置生成有點單調，因此我在self.rect.centerx加了點料 : 
    ```python=
    # Platform.py
    import random
    def __init__(self):
        #...省略
        self.rect.centerx = random.randint(self.rect.width//2+300, WIDTH-self.rect.width//2-300)
        self.rect.top = 15
        # 上述的random.randint是甚麼呢？
        # 它接收start、end，並從兩數字之間的範圍「隨機」選一個數字
        # 這樣是不是每次生成平台時，x座標就是隨機的啦～
    ```
    - 在主程式中，我們建立Platform的group，同時先在迴圈中呼叫更新
    ```python=
    # main.py
    platforms = pygame.sprite.Group()
    platform1 = Platform()
    platforms.add(platform1)
    all_sprites.add(platforms)
    
    while running:
        # ...省略
        # 角色更新
        players.update()
        platforms.update() # <--這行
                
        # 畫面渲染
        screen.fill(BLACK)
        all_sprites.draw(screen)
        pygame.display.flip()
    ```
- 🌟Platform的移動🌟
    - 由於平台會按照特定速度下落，因此我們需要給定值
    ```python=
    # Settings.py
    # ...省略
    # 平台資訊
    PLATFORM_Y_SPEED = 0.5
    ```
    - 接著我們要來撰寫下落的功能
    ```python=
    # Platform.py
    class Platform(pygame.sprite.Sprite):
        # 類別成員
        y_speed = PLATFORM_Y_SPEED

        def __init__(self):
            # ...省略

            self.velocity = [0, Platform.y_speed]
            self.pos = [float(self.rect.centerx), float(self.rect.centery)]

        def update(self):
            self.change_position()

        def change_position(self):
            self.pos[1] += self.velocity[1]
            self.rect.centery = self.pos[1]
    ```
    - 由於平台除了下落，還會左右移動！因此我們要來寫左右移動的功能。
    - 首先先到Settings設定平台的「**x移動速度**」，我們用range給出一個範圍，到時候使用random隨機抽一個速度，這樣每次生成的平台速度不會都一樣！
    ```python=
    # Settings.py
    # ...省略
    # 平台資訊
    PLATFORM_X_SPEED = range(-3,5,2)
    ```
    - 我們新增隨機抽取的功能
    ```python=
    # Platform.py
    # ...省略
    self.velocity = [random.choice(PLATFORM_X_SPEED), Platform.y_speed]
    
    def change_position(self):
        # ...省略
        self.pos[0] += self.velocity[0]
        self.rect.centerx = self.pos[0]
    ```
    - 接著我們要讓平台能夠碰到邊邊反彈回來！
    ```python=
    # Platform.py
    def border(self):
        if self.rect.left < 0 or self.rect.right > WIDTH:
            self.velocity[0]*=-1
    ```
    
- 🌟Platform的生成🌟
    - 每個平台的生成需要間隔一些距離，生成的動作會在主檔案中寫，那我們就馬上來寫一個函式負責處理平台的生成吧！
    ```python=
    # main.py
    def manage_platforms():
        temp = []
        for p in platforms:
            temp.append(p.rect.top)
        highest_platform = min(temp)

        randomGap = random.randint(130, 160)
        if highest_platform > randomGap:
            p = Platform()
            platforms.add(p)
            all_sprites.add(p)
    ```
    - 這樣就好了！但我想要改動一個東西！請看下方
    ```python=
    # main.py
    def manage_platforms():
        highest_platform = min([p.rect.top for p in platforms]) # <--這行 

        randomGap = random.randint(130, 160)
        if highest_platform > randomGap:
            p = Platform()
            platforms.add(p)
            all_sprites.add(p)
    ```
    - 我修改了找出最高平台的寫法，可以更加簡潔程式碼！
    - 最後一樣，在迴圈中呼叫此函式 : 
    ```python=
    # main.py
    while running:
        # ...省略
        # 角色更新
        manage_platforms() # <--這行
        players.update()
        platforms.update()

        # 畫面渲染
        screen.fill(BLACK)
        all_sprites.draw(screen)
        pygame.display.flip()
    ```

### Platform與Player的碰撞作用
- 🌟碰撞初見！🌟
    - 首先，我們要知道碰撞的概念 --> 精靈對精靈影格重複時稱為碰撞。
    - 針對碰撞，我們需要利用pygame.sprite內建的collision來做
    - 我們想要 --> **玩家站在平台上**，此時即為一種碰撞。
    - 首先要在玩家初始化中加入兩個屬性 : 平台速度、是否站在平台上
    ```python=
    # Player.py
    def __init__(self):
        ...
        self.is_stand_on_platform = False
    ```
    - 接著我們要寫一個偵測碰撞平台的函式在
    ```python=
    # Player.py
    def check_platforms_collision(self, platforms_group):
        self.is_stand_on_platform = False
        hits = pygame.sprite.spritecollide(self, platforms_group, False)
        
        for platform in hits:
            if self.velocity[1] >= 0:
                # 定位玩家於平台上
                self.rect.bottom = platform.rect.top
                # 將玩家x、y軸速度設為平台x、y速度
                self.velocity[1] = platform.velocity[1]
                self.velocity[0] = platform.velocity[0]
                # 追蹤目前的平台
                self.p = platform
                # 刷新玩家的y軸位置(嗯？為什麼沒有刷新到x軸呢？)
                self.pos[1] = float(self.rect.centery)
                # 將玩家站在平台狀態設為True
                self.is_stand_on_platform = True
                break
    ```
    - 由於玩家x軸的速度除了平台速度以外，還會加上玩家左右鍵的輸入，因此我們要額外新增一個函式
    ```python=
    # Player.py
    def change_position(self):
        self.pos[0] += self.velocity[0]
        self.rect.centerx = self.pos[0]
    ```
    - 接著我們要修改當左右鍵被按下時的動作
    ```python=
    # Player.py
    def key_detection(self):
        kp = pygame.key.get_pressed()
        if self.is_stand_on_platform:
            if kp[pygame.K_LEFT]: self.velocity[0] = self.x_speed*-1 + self.p.velocity[0]
            if kp[pygame.K_RIGHT]: self.velocity[0] = self.x_speed + self.p.velocity[0]
        else:
            # 當玩家脫離平台後，速度只跟左右鍵有關
            if kp[pygame.K_LEFT]: self.velocity[0] = self.x_speed*-1
            if kp[pygame.K_RIGHT]: self.velocity[0] = self.x_speed
        ...
    ```
    - 最後修改重力函式，由於站在平台上時，玩家速度跟著平台走，不加重力，因此
    ```python=
    # Player.py
    def gravity(self):
        if self.is_stand_on_platform:
            return
        # ...省略
    ```
    - 組合以上的函式，我們可以寫出下面這個流程 : 
    ```python=
    # Player.py
    def update(self, platforms_group):
        self.gravity()
        self.check_platforms_collision(platforms_group)
        self.key_detection()
        self.change_position()
        self.border()
    ```
    - 你會發現前面先將x速度資訊設定完畢後，才會執行change_position！

### Pickups角色的類別撰寫
- 🌟Pickups初見🌟
    - 寫完了Platform跟Player，我們可以來完成最後一個角色啦！
    - 由於只有玩家跟平台似乎很無聊，因此必須加點道具增加趣味！
    - 我們先定義了三種道具Buff加成 --> X速度加成、跳躍增益、慢速降落，效果持續 : 3秒。
    - 而三者又可對應到Settings中的 : 
    ```python=
    # Settings.py
    # ...省略
    PLAYER_X_SPEED = 5
    PLAYER_JUMP_V = -15
    G = 0.3
    ```
    - 由於吃到道具會更改這些數值，但我們又不能動到Settings的值，因此要先在 Player.py 的初始化中**增加**三個變數 : 
    ```python=
    # Player.py
    def __init__(self):
        # ...省略
        self.x_speed = PLAYER_X_SPEED # 這個前面應該有寫過了！
        self.jump_v = PLAYER_JUMP_V
        self.g = G
    ```
    - 接下來我們要在Settings.py中加入三個buff的字典 : 
    ```python=
    # Settings.py
    # 道具資訊
    PICKUPS_NAME_LIST = {
        0 : "Speed", 
        1 : "Jump_Height", 
        2 : "Feather_Flight"
    }
    ```
    - 好了！目前為止，我們已經定義好了Buff的基本架構，接下來可以進入Pickups類別了！
- 🌟Pickups生成🌟
    - 跟Platform及Player一樣，我們會先定義類別基本長相與定位 : 
    ```python=
    # Pickups.py
    import pygame
    from Settings import *

    class Pickups(pygame.sprite.Sprite):
        def __init__(self):
            super().__init__()
            self.image = pygame.Surface((20,20))
            self.image.fill(BLUE)
            self.rect = self.image.get_rect()
            # 明眼的你發現了這裡沒有center定位！

        def update(self):
            pass
    ```
    - 再來我們要定義當道具生成時，這個道具的效果是甚麼，我們取得key : 
    ```python=
    # Pickups.py
    import random

    def __init__(self):
        # ...省略
        self.buff = random.choice(list(PICKUPS_NAME_LIST.keys()))
    ```
    - 接著在main.py建立道具Group！務必先引入Pickups，最後一樣，先呼叫道具更新
    ```python=
    # main.py
    from Pickups import *
    # ...省略
    pickups = pygame.sprite.Group()
    
    while running:
        # ...省略
        # 角色更新
        manage_platforms()
        players.update(platforms)
        platforms.update()
        pickups.update() # <--這行        

        # 畫面渲染
        screen.fill(BLACK)
        all_sprites.draw(screen)
        pygame.display.flip()
    ```
    - 接下來我們要來做一件很神奇的事 --> 在初始化函式加一個參數 : **platform**
    ```python=
    # Pickups.py
    def __init__(self, platform):
        self.platform = platform
    ```
    - 想想看，為甚麼要這麼做？
    - **原因** : 道具always生成在平台上，又每個平台位置與移動速度不盡相同，因此要明確標註道具所在的平台是哪一個？
    - 接下來我們才能定位道具
    ```python=
    # Pickups.py
    def __init__(self, platform):
        ...
        self.rect.centerx = self.platform.rect.centerx
        self.rect.bottom = self.platform.rect.top
        
    def change_position(self):
        self.rect.centerx = self.platform.rect.centerx
        self.rect.bottom = self.platform.rect.top
        
    def update(self):
        change_position()
    ```
    - 接著呢，我們要在 main.py 寫一個神奇的生成道具函式！
    ```python=
    # main.py
    def manage_pickups(platform):
        p = Pickups(platform)
        pickups.add(p)
        all_sprites.add(p)
    
    while running:...
    ```
    - 我定義 : 每當平台生成時，道具有「**機率**」跟著生成，因此我需要設定機率！
    ```python=
    # Settings.py
    # 道具資訊
    PICKUPS_SPAWN_RATIO = 10 # 機率 = 1/10，如果我改成5，就是1/5機率。
    ```
    - 寫好生成道具函式及機率後，我們還需有呼叫的地方 --> **manage_platforms**
    ```python=
    # main.py
    def manage_platforms():
        # ...省略
        if highest_platform > randomGap:
            # ...省略
            r = random.randint(1, PICKUPS_SPAWN_RATIO)
            if r == 1:
                manage_pickups(p)
    ```
    - 上述用到了randint，讓程式從1到我們設定的值之間抽一個數字 --> 等於1的機率**理論上**即為「1 / PICKUPS_SPAWN_RATIO」。

### Pickups與Player的碰撞作用
- 🌟碰撞邏輯🌟
    - 由於我們有寫過玩家平台碰撞函式了！所以直接搬架構過來再修改成道具形式 : 
    ```python=
    # Player.py
    def check_pickups_collision(self, pickups_group):
        # 這邊寫True意即要刪掉被玩家碰到的道具
        hits = pygame.sprite.spritecollide(self, pickups_group, True)
        
        for pickup in hits:
            pass
    ```
    - 至於for迴圈該寫什麼呢？在寫之前，有些前置作業 : 
    - 首先要先去 Settings.py 定義幾個東西 : **buff倍率、buff時長、buff狀態**。
    ```python=
    # Settings.py
    # 道具資訊
    # RATIO即為倍率、TIME單位是秒
    PICKUP_SPEED_RATIO = 2
    PICKUP_JUMPVELOCITY_RATIO = 1.2
    PICKUP_GRAVITY_RATIO = 0.6
    POWERUP_TIME = 3 # 增益時長
    ```
    - 再來要在 Player.py 中加入幾個屬性 : 
    ```python=
    # 還記得我們有定義過buff的字典嗎？ 忘記了滑上去看！注意鍵值對的位置！
    # 這邊list中的index跟字典中的鍵是相同的！
    # 如index 1是False代表jump_height是False！
    def __init__(self):
        # ...省略
        self.buffList = [False, False, False]
        self.buffTimer = [0,0,0] # 單位是tick
    ```
    - 接著我們要來撰寫當吃到道具後的「倒計時」、「buff失效後的重置」:
    ```python=
    # Player.py
    def timer_detect(self):
        # 玩家X速度buff偵測
        if self.buffList[0]: # 狀態開啟偵測
            self.buffTimer[0] -= 1 # 倒數
            if self.buffTimer[0] <= 0: # 當時間為0時重置 : 時間、狀態、效果
                self.buffTimer[0] = 0
                self.buffList[0] = False
                self.x_speed = PLAYER_X_SPEED
        
        # 玩家跳躍buff偵測
        if self.buffList[1]:
            self.buffTimer[1] -= 1
            if self.buffTimer[1] <= 0:
                self.buffTimer[1] = 0
                self.buffList[1] = False
                self.jump_v = PLAYER_JUMP_V
        
        # 玩家掉落慢速buff偵測
        if self.buffList[2]:
            self.buffTimer[2] -= 1
            if self.buffTimer[2] <= 0:
                self.buffTimer[2] = 0
                self.buffList[2] = False
                self.g = G
    ```
    - 最後我們終於可以來完成碰撞後for迴圈的邏輯拉～
    ```python=
    # Player.py
    def check_pickups_collision(self, pickups_group):
        hits = pygame.sprite.spritecollide(self, pickups_group, True)
        
        for pickup in hits:
            if pickup.buff == 0:
                self.x_speed = PLAYER_X_SPEED*PICKUP_SPEED_RATIO
                self.buffList[0] = True
                # 由於POWERUP_TIME是秒，乘以FPS後，單位才是刻，符合我們倒計時的減法邏輯
                self.buffTimer[0] = POWERUP_TIME*FPS

            if pickup.buff == 1:
                self.jump_v = PLAYER_JUMP_V*PICKUP_JUMPVELOCITY_RATIO
                self.buffList[1] = True
                self.buffTimer[1] = POWERUP_TIME*FPS

            if pickup.buff == 2:
                self.g = G*PICKUP_GRAVITY_RATIO
                self.buffList[2] = True
                self.buffTimer[2] = POWERUP_TIME*FPS
    ```
    - 最後記得在update函式中加入倒計時、碰撞函式 : 
    ```python=
    # Player.py
    def update(self, platforms_group, pickups_group):
        self.gravity()
        self.check_platforms_collision(platforms_group)
        self.check_pickups_collision(pickups_group)
        self.timer_detect()
        self.key_detection()
        self.change_position()
        self.border()
    ```
    - 並且在 main.py 中更新部分加入pickups_group參數 : 
    ```python=
    # main.py
    # 角色更新
    manage_platforms()
    players.update(platforms, pickups) # <--放入pickups Group
    platforms.update()
    pickups.update()       
    ```
### 補全三大角色的屬性及功能
- 🌟Platform生成位置🌟
    - 當遊戲開始後，我希望玩家初始位置就站在平台上且**第一個平台X位置必在螢幕正中央**，因此我們要設定一些東西 : 
    ```python=
    # Platform.py
    class Platform(pygame.sprite.Sprite):
        # 生成平台時，是否為最一開始的平台，為類別變數
        is_First = True
        def __init__(self):
            # ...省略
            
            self.rect.top = 15
            if Platform.is_First:
                self.rect.centerx = WIDTH/2
                # 第一個平台生成完後，將is_First設為False
                Platform.is_First = False
            else:
                self.rect.centerx = random.randint(self.rect.width//2+300, WIDTH-self.rect.width//2-300)
    ```
- 🌟玩家只能在平台上跳躍🌟
    - 原本我們設定是當玩家只能在「下落過程中跳躍」，但我現在要改成只能「站在平台上時才能跳躍」。
    ```python=
    # Player.py
    def jump(self):
        if not self.is_stand_on_platform:
            return
        self.velocity[1] = self.jump_v
        self.pos[1] += self.velocity[1]
        self.rect.centery = self.pos[1]
    ```
- 🌟玩家、平台、道具超出HEIGHT時自毀🌟
    - 由於平台、道具一直生成，若沒有執行self.kill()，到後面會有一堆物件在螢幕外繼續下落，十分占用記憶體。
    - 且遊戲架構中，我們定義玩家掉出螢幕外，是死亡，而不是站在地面上，這也要修改。
    ```python=
    # Platform.py
    def update(self):
        # ...省略
        if self.rect.top > HEIGHT:
            self.kill()
            return
    ```
    ```python=
    # Pickups.py
    def update(self):
        # ...省略
        if self.rect.top > HEIGHT:
            self.kill()
            return
    ```
    ```python=
    # Player.py
    def gravity(self):
        if self.is_stand_on_platform:
            return
        
        if self.rect.top > HEIGHT:
            self.kill()
            return
        
        self.velocity[1] += self.g
        self.pos[1] += self.velocity[1]
        self.rect.centery = self.pos[1]
    ```
    - 最後記得將Player的「border函式」以及「update中呼叫border的部分」**註解掉**，使玩家可以掉出螢幕外並死亡 : 
    ```python=
    # Player.py
    # def border(self):
    #     if self.rect.bottom > HEIGHT:
    #         self.rect.bottom = HEIGHT
    #         self.pos[1] = float(self.rect.centery)
    #         self.velocity[1] = 0
    
    def update(self):
        # ...省略
        # self.border()
    ```
- 🌟平台隨著時間加速🌟
    - 為增加遊戲難度，我們定義「**平台每隔多久加速一次**」、「**每次加速的倍率是多少**」。
    ```python=
    # Settings.py
    PLATFORM_Y_SPEED_RATIO = 1.05 # 倍率為1.05
    PLATFORM_RATIO_CHANGE_SPAN = 2 # 單位是秒
    ```
    - 接著要在 Platform.py 中**新增**計時器屬性、加速函式 : 
    ```python=
    # Platform.py
    def __init(self):
        # ...省略
        self.timer = 0
        
    def change_y_speed(self):
        self.timer += 1
        # 這邊乘以FPS --> 轉換成刻
        if self.timer >= PLATFORM_RATIO_CHANGE_SPAN*FPS:
            Platform.y_speed *= PLATFORM_Y_SPEED_RATIO
            # 設定速度上限，若速度大於2，就強制等於2，不然到後面平台會下落快到無法遊玩
            if Platform.y_speed >= 2: 
                Platform.y_speed=2
            # timer歸零
            self.timer = 0
        self.velocity[1] = Platform.y_speed
    
    def update(self):
        self.change_position()
        self.change_y_speed() # <--加入這行
        self.border()
    ```
- 🌟平台生成的間隔隨著y_speed變大而變大🌟
    - 由於平台速度加快，生成的距離間隔也應拉長 : 
    ```python=
    def manage_platforms():
        # ...省略
        speed_factor = Platform.y_speed / PLATFORM_Y_SPEED
        randomGap = random.randint(130, 160) * speed_factor
        if randomGap > 200 : 
            randomGap = 200

        if highest_platform > randomGap:
            # ...省略
    ```

### 引入圖檔、音檔、字體
- 恭喜你快完成這個遊戲了！由於引入的步驟十分單一，我不打算寫太多引導，請直接照抄我的程式吧！
- 請務必注意，引入的位置一定要在pygame初始化下面，主迴圈的上面！
    - 初始化及while的部分不需要複製，只是告訴你，引入的位置應該放在哪。
    ```python=
    # main.py
    import os
    
    # 初始化在這
    pygame.init()
    pygame.mixer.init() # <--這行是新加的，mixer初始化
    pygame.display.set_caption("My first game")
    screen = pygame.display.set_mode((WIDTH, HEIGHT))
    clock = pygame.time.Clock()
    running = True
    
    # 引入圖檔
    start_bg = pygame.image.load(os.path.join("img", "start_bg.png")).convert_alpha()
    start_bg = pygame.transform.scale(start_bg, (WIDTH, HEIGHT))
    background = pygame.image.load(os.path.join("img", "background.png")).convert_alpha()
    background = pygame.transform.scale(background, (WIDTH, HEIGHT))

    platform1_img = pygame.image.load(os.path.join("img", "platform1.png")).convert_alpha()
    platform2_img = pygame.image.load(os.path.join("img", "platform2.png")).convert_alpha()
    platform3_img = pygame.image.load(os.path.join("img", "platform3.png")).convert_alpha()
    platform_list = [platform1_img, platform2_img, platform3_img]

    feather = pygame.image.load(os.path.join("img", "feather.png")).convert_alpha()
    jump = pygame.image.load(os.path.join("img", "jump.png")).convert_alpha()
    speed = pygame.image.load(os.path.join("img", "speed.png")).convert_alpha()
    pickup_list = [speed, jump, feather]
    
    # 引入音檔
    background_music = pygame.mixer.Sound(os.path.join("sound", "background_music.mp3"))
    background_music.set_volume(background_music.get_volume()/5)
    background_music.play(loops=-1)
    
    #* 引入字體
    font_type = "font/pixel_upgrade.ttf"
    
    # 顯示文字的函式
    def draw_text(surface, text, size, x, y, color, align="center"):
        font = pygame.font.Font(font_type, size)
        text_surface = font.render(text, True, color)
        text_rect = text_surface.get_rect()
        if align == "left":
            text_rect.left = x
        else:
            text_rect.centerx = x
        text_rect.top = y
        surface.blit(text_surface, text_rect)
    
    while running:...
    ```
- 🌟Player角色的渲染、音效🌟
    - 我們要來定義玩家的圖檔大小，因此先到Settings去新增一個常數
    ```python=
    # Settings.py
    PLAYER_PIC_SIZE = (53, 45)
    ```
    - 再來引入玩家的走路、跳躍、站立等圖檔，並設計動畫(當玩家按下左右跳躍鍵時的圖檔切換)，你可以直接複製我標記的區間 : 
    ```python=
    # Player.py
    import os
    
    def __init__(self):
        super().__init__()
        
        # 玩家圖檔引入
        self.front_p = pygame.image.load(os.path.join("img", "front_player.png")).convert_alpha()
        self.jump_p = pygame.image.load(os.path.join("img", "jump_player.png")).convert_alpha()
        self.walk_p = pygame.image.load(os.path.join("img", "walk_player.png")).convert_alpha()
        self.image = pygame.transform.scale(self.front_p, PLAYER_PIC_SIZE)
        
        # 玩家定位
        self.rect = self.image.get_rect()
        self.rect.centerx = WIDTH/2
        self.rect.bottom = 15
        
        # 玩家圖檔格式化
        self.walk_player = pygame.transform.scale(self.walk_p, PLAYER_PIC_SIZE)
        self.walk_L_player = pygame.transform.flip(self.walk_player, True, False) 
        self.jump_player = pygame.transform.scale(self.jump_p, PLAYER_PIC_SIZE)
        self.jump_L_player = pygame.transform.flip(self.jump_player, True, False) 
        
        # 玩家音檔引入
        self.pop_sound = pygame.mixer.Sound(os.path.join("sound", "pop.mp3"))
        self.jump_sound = pygame.mixer.Sound(os.path.join("sound", "jumping.mp3"))
        self.oof_sound = pygame.mixer.Sound(os.path.join("sound", "oof.mp3"))
        
        # 位置資訊...省略
    
    # 玩家動畫
    def animate(self):
        kp = pygame.key.get_pressed()
                
        if kp[pygame.K_UP]:
            self.image = self.jump_player
            if kp[pygame.K_LEFT]: self.image = self.jump_L_player
        else:      
            if kp[pygame.K_LEFT]: self.image = self.walk_L_player
            if kp[pygame.K_RIGHT]: self.image = self.walk_player   
    ```
    - 接著我們要來讓音效現身！
    ```python=
    # Player.py
    def jump(self):
        self.jump_sound.play() # <--跳躍音效
        # ...省略
    
    def gravity(self):
        # ...省略
        if self.rect.top > HEIGHT:
            self.oof_sound.play() # <--死亡音效
            self.kill()
            return
        # ...省略
    
    def check_pickup_collide(self, pickups_group):
        hits = pygame.sprite.spritecollide(self, pickups_group, True)
        for pickup in hits:
            self.pop_sound.play() # <--吃道具音效
            # ...省略
    ```
- 🌟Platform角色的渲染🌟
    - 我們會在Platform的初始化中加一個參數 : 圖片List，由main檔傳入，這樣子每次生成平台時，我們就可以隨機抽一個平台出來，讓每個平台造型不會都一樣！
    ```python=
    # Platform.py
    def __init__(self, platform_list):
        super().__init__()
        self.choice = random.randint(0,2)
        if self.choice == 0:
            self.image = pygame.transform.scale(platform_list[self.choice], (platform_list[self.choice].get_size()[0]/2, platform_list[self.choice].get_size()[1]/5))
        elif self.choice == 1:
            self.image = pygame.transform.scale(platform_list[self.choice], (platform_list[self.choice].get_size()[0], platform_list[self.choice].get_size()[1]/2))
        elif self.choice == 2:
            self.image = pygame.transform.scale(platform_list[self.choice], (platform_list[self.choice].get_size()[0], platform_list[self.choice].get_size()[1]/2))
        # ...省略
    ```
    - 由於接收了參數，因此我們在main檔要修改這些部分 : 
    ```python=
    # main.py
    # ...省略
    platform1 = Platform(platform_list) # <--這行
    platforms.add(platform1)
    
    def manage_platforms():
        # ...省略
        if highest_platform > randomGap:
            p = Platform(platform_list) # <--這行
            platforms.add(p)
            all_sprites.add(p)
            # ...省略
    ```
- 🌟Pickups角色的渲染🌟
    - 由於我們有定義過self.buff，因此可以直接以它為index去取得相對應的圖片。
    - 因此我們也需要在初始化中加入一個pickups_list的參數 : 
    ```python=
    # Pickups.py
    
    def __init__(self, platform, pickup_list):
        super().__init__()

        self.buff = random.choice(list(PICKUPS_NAME_LIST.keys()))
        self.image = pygame.transform.scale(pickup_list[self.buff], (pickup_list[self.buff].get_size()[0]/3, pickup_list[self.buff].get_size()[1]/3))
         # ...省略
    ```
    - 由於接收了參數，因此在main檔案也要修改。
    ```python=
    # main.py
    def manage_pickups(platform):
        p = Pickups(platform, pickup_list) # <--加入pickup_list
        pickups.add(p)
        all_sprites.add(p)
    ```
- 🌟main遊戲畫面的渲染🌟
    - 為了將畫面給渲染出來，我們需要修改main.py中的一個部分。
    - 請找到 `screen.fill(BLACK)`，將其改成底下這行 :  
    ```python=
    # main.py
    while running:
        # ...省略
        screen.blit(background, (0,0)) # background是我們引入的背景圖
    ```
### 最終關卡 : 設計遊戲資料與重置
- 🌟設計遊戲資料🌟
    - 這是最後一部份了，加油！
    - 接著我們要來設計遊戲的資料 : 遊玩時間、死亡次數、最高分數
    ```python=
    # main.py
    import datetime # <--這是拿來計算時間的
    # ...省略初始化部分
    play_time_per_tick = 0
    play_time_s = 0
    play_time = str(datetime.timedelta(seconds=play_time_s))
    score = 0
    data = {"highest_score": 0,
           "total_play_time": play_time,
           "death_count": 0}
    game_state = "START_MENU"
    # ...省略引入圖檔後續部分
    ```
    - 再進行下一步之前，讓我們先去Settings定義一個顏色，這個顏色是拿來渲染文字的。
    ```python=
    # Settings.py
    MENU_DATA_COLOR = (194, 224, 252)
    ```
    - 再來我們要在迴圈當中不斷增加分數、時間並且偵測死亡重置
    ```python=
    #* 取得輸入
    for event in pygame.event.get():
        # ...省略
        if event.type == pygame.KEYDOWN:
            if game_state == "START_MENU" and event.key == pygame.K_SPACE:
                game_state = "PLAYING"

    #* 根據狀態進行邏輯更新與畫面渲染
    if game_state == "START_MENU":
        screen.blit(start_bg, (0, 0))
        draw_text(screen, "最高分數 : " + str(data["highest_score"]), 22, 20, 10, MENU_DATA_COLOR, align="left")
        draw_text(screen, "死亡次數 : " + str(data["death_count"]), 22, 20, 180, MENU_DATA_COLOR, align="left")
        draw_text(screen, "遊戲時間 : " + str(data["total_play_time"]), 22, 20, 40, MENU_DATA_COLOR, align="left")
    
    else:
        # 還記得group是一個容器嗎？若容器為空代表False。
        # 若玩家self.kill()，players就變False，not players就變成True！
        # 因此當玩家死亡就會進入重置 : 
        if not players: 
            #* 更新遊戲資料
            data["death_count"] += 1
            if score > data["highest_score"]:
                data["highest_score"] = score
            data["total_play_time"] = str(datetime.timedelta(seconds=play_time_s))
            
            #* 清除四個群組
            all_sprites.empty()
            platforms.empty()
            players.empty()
            pickups.empty()
            
            #* 初始化設定
            Platform.is_First = True
            Platform.y_speed = PLATFORM_Y_SPEED 
            game_state = "START_MENU"
            score = 0
            
            #* 重新建立
            player1 = Player()
            players.add(player1)
            platform1 = Platform(platform_list)
            platforms.add(platform1)
            all_sprites.add(platforms, pickups, players)
        
        score+=1
        play_time_per_tick+=1
        
        #* 遊戲時間計算
        if play_time_per_tick >= 60:
            play_time_per_tick = 0
            play_time_s += 1
            play_time = str(datetime.timedelta(seconds=play_time_s))
            
        # 角色更新 ...省略   

        # 畫面渲染
        screen.blit(background, (0,0))
        all_sprites.draw(screen)
        draw_text(screen, str(score), 22, WIDTH/2, 10, WHITE)
        draw_text(screen, play_time, 22, WIDTH-60, HEIGHT-50, WHITE)
    ```

### 結尾
- 🌟玩家穿透功能加入🌟
    - 如果你看到這裡，代表你已經走完整體遊戲架構了！接下來是額外功能，可以增加遊戲趣味性！
    ```python=
    # Player.py
    # Credit by 皮蛋（電神）⚡⚡
    # 此功能可以使玩家穿過左螢幕會從右螢幕出現，反之亦然。
    def penetrate(self):
        if self.rect.centerx < 0:
            self.pos[0] = WIDTH
            self.rect.centerx = self.pos[0]
        if self.rect.centerx > WIDTH:
            self.pos[0] = 0
            self.rect.centerx = 0
    
    def update(self):
        # ...省略
        self.penetrate()
    ```
- 🌟感性時間🌟
    - 做這個文檔花了我近10個小時，如果你真的看完的話，我會很開心的QAQ。