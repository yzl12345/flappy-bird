# flappy-bird
# I made the flappy bird in sophomore year, using python.

#计算得分
import pygame
import sys
import random
import PIL.Image as Image
import time
import os

speed = 4               #初始速度
score = 0               #初始分数
difficulty = 0          #初始难度
choice = 0              #难度选择
button_stop = False     #暂停按钮
button_begin = False    #开始按钮
button_restart = False  #重新开始按钮
button_quit = False     #退出按钮
button_music = True     #音乐按钮
button_choice = False   #难度选择按钮
space = [730, 680, 630] #管道间距(难度控制)

# ----------------创建枚举类----------------#
class Enum():
    esay = 0
    normal = 1
    difficult = 2


# ----------------创建小鸟类----------------#
class Bird():
    def __init__(self):
        self.birdstatus = [pygame.image.load("assets/1.png"),
                           pygame.image.load("assets/2.png"),
                           pygame.image.load("assets/dead.png")]
        self.status = 0
        self.X = 100
        self.Y = 300
        self.jump = False
        self.up = 10
        self.down = 5
        self.dead = False

    # 小鸟移动
    def move(self):
        # 跳跃
        if self.jump == True:
            self.up -= 1
            self.Y -= self.up
        # 下坠(没跳跃)
        else:
            self.down += 0.2
            self.Y += self.down


# ----------------创建管道类----------------#
class Pipeline():
    def __init__(self):
        global space, difficulty
        self.PipeUp = pygame.image.load("assets/top.png")
        self.PipeDown = pygame.image.load("assets/bottom.png")
        self.X = 500
        self.Up_Y = random.randint(-350, -200)
        self.Down_Y = self.Up_Y + space[difficulty]
        print(difficulty)

    # 更新管道
    def updatePipeline(self):
        global speed, space, difficulty
        self.X -= speed
        if self.X <= -85:
            global score        # 全局变量
            score += 1
            if speed >= 14:
                speed = 14
            else:
                speed += 0.2
            self.X = 450
            self.Up_Y = random.randint(-350, -200)
            self.Down_Y = self.Up_Y + space[difficulty]


# ------------------3秒倒计时准备------------------#
def Delay():
    #更新屏幕
    Show()

    str3 = '3'
    str2 = '2'
    str1 = '1'
    str = 'Go!'
    begin3 = font_100.render(str3, 1, (255,255,255))
    begin2 = font_100.render(str2, 1, (255,255,255))
    begin1 = font_100.render(str1, 1, (255,255,255))
    begin = font_100.render(str, 1, (255,255,255))
    #开始填充
    screen.blit(begin3, (220, 270))
    pygame.display.update()
    pygame.time.delay(1000)         #延时1s

    Show()
    screen.blit(begin2, (220, 270))
    pygame.display.update()
    pygame.time.delay(1000)         #延时1s

    Show()
    screen.blit(begin1, (220, 270))
    pygame.display.update()
    pygame.time.delay(1000)         # 延时1s

    Show()
    screen.blit(begin, (200, 270))
    pygame.display.update()
    pygame.time.delay(1000)         # 延时1s


# ------------------图像背景透明化------------------#
def transparent_back(img):
    img = img.convert('RGBA')    #转化成4个通道，RGB加上透明度的通道
    L, H = img.size
    color_0 = img.getpixel((0,0))
    for h in range(H):
        for l in range(L):
            dot = (l,h)
            color_1 = img.getpixel(dot)
            if color_0[0]-20< color_1[0] and color_1[0]<color_0[0]+20:#与左上角第一个像素点第一个通道值相差20以内的，透明度都设置为0。20这个阈值可以根据图片效果修改。
                color_1 = color_1[:-1] + (0,)
                img.putpixel(dot,color_1)
    return img


# ------------------图像处理------------------#
def image():
    img1 = Image.open("游戏按钮/暂停.png")
    img1 = transparent_back(img1)
    img1.save("游戏按钮/暂停.png")

    img2 = Image.open("游戏按钮/重新开始.png")
    img2 = transparent_back(img2)
    img2.save("游戏按钮/重新开始.png")

    img3 = Image.open("游戏按钮/退出2.png")
    img3 = transparent_back(img3)
    img3.save("游戏按钮/退出2.png")

    img4 = Image.open("游戏按钮/暂停2.png")
    img4 = transparent_back(img4)
    img4.save("游戏按钮/暂停2.png")

    img5 = Image.open("游戏按钮/取消暂停.png")
    img5 = transparent_back(img5)
    img5.save("游戏按钮/取消暂停.png")

    img6 = Image.open("游戏按钮/声音.png")
    img6 = transparent_back(img6)
    img6 = img6.resize((70,70))         #图像缩放
    img6.save("游戏按钮/声音.png")

    img7 = Image.open("游戏按钮/静音.png")
    img7 = transparent_back(img7)
    img7 = img7.resize((70, 70))        #图像缩放
    img7.save("游戏按钮/静音.png")

    img8 = Image.open("游戏按钮/简单.png")
    img8 = img8.resize((350, 150))  # 图像缩放
    img8.save("游戏按钮/简单.png")

    img9 = Image.open("游戏按钮/普通.png")
    img9 = img9.resize((350, 150))  # 图像缩放
    img9.save("游戏按钮/普通.png")

    img10 = Image.open("游戏按钮/困难.png")
    img10 = img10.resize((350, 150))  # 图像缩放
    img10.save("游戏按钮/困难.png")


# ----------------获取小鸟状态----------------#
def GetStatus():
    # 死亡
    if bird.dead:
        bird.status = 2
    # 跳跃
    elif bird.jump:
        bird.status = 1


# ----------------检查小鸟是否死亡----------------#
def CheckDead():
    #检测碰撞
    bird_rect = pygame.Rect(bird.X, bird.Y, 40, 30)
    Up_rect = pygame.Rect(pipeline.X, pipeline.Up_Y, 100, 500)
    Down_rect = pygame.Rect(pipeline.X, pipeline.Down_Y, 98, 500)

    if bird_rect.colliderect(Up_rect) or bird_rect.colliderect(Down_rect):
        bird.dead = True

    #检测边缘
    if bird.Y<0 or bird.Y>650:
        bird.dead = True


# ------------------执行------------------#
def execute():
    # 更新屏幕
    bird.move()  # 小鸟移动
    CheckDead()  # 检查小鸟是否死亡
    GetStatus()  # 获取小鸟状态
    pipeline.updatePipeline()  # 管道移动
    Show()  # 更新屏幕


# ------------------背景音乐1------------------#
def music_begin():
    pygame.mixer.music.load("song/大酥-此时此世与你.mp3")
    pygame.mixer.music.set_volume(0.25)                 #声音设置
    global button_music
    if button_music:
        pygame.mixer.music.play(-1, 0)
    # 第一个参数：播放n遍(-1是循环播放)         第二个参数：第x秒开始播放
    else:
        pygame.mixer.music.stop()


# ------------------背景音乐2------------------#
def music_ing():
    pygame.mixer.music.load("song/BoA-MASAYUME CHASING.mp3")
    pygame.mixer.music.set_volume(0.25)  # 声音设置
    global button_music
    if button_music:
        pygame.mixer.music.play(-1, 0)
    # 第一个参数：播放n遍(-1是循环播放)         第二个参数：第x秒开始播放
    else:
        pygame.mixer.music.stop()


# ------------------背景音乐3------------------#
def music_end():
    pygame.mixer.music.load("song/马嘉祺&刘栖子-凉凉.mp3")
    pygame.mixer.music.set_volume(0.25)  # 声音设置
    global button_music
    if button_music:
        pygame.mixer.music.play(-1, 0)
    # 第一个参数：播放n遍(-1是循环播放)         第二个参数：第x秒开始播放
    else:
        pygame.mixer.music.stop()


# ------------------初始菜单------------------#
def Show_Begin1():
    # 背景
    background = pygame.image.load("assets/background.png")
    screen.blit(background, (0, 0))
    screen.blit(background, (390, 0))  # 图片不够大，所以填充两次
    # 小鸟
    screen.blit(bird.birdstatus[bird.status], (20, 35))
    #字体
    str = "flappy bird"
    topic = font_100.render(str, 1, (255, 255, 255))
    screen.blit(topic, (70,20))
    #游戏按钮
    begin = pygame.image.load("游戏按钮/开始.png")
    end = pygame.image.load("游戏按钮/退出.png")
    screen.blit(begin, (30, 200))
    screen.blit(end, (30, 400))
    #声音按钮
    sound = pygame.image.load("游戏按钮/声音.png")
    mute = pygame.image.load("游戏按钮/静音.png")
    global  button_music
    if button_music:
        screen.blit(sound, (430, 620))
    else:
        screen.blit(mute, (430, 620))

    pygame.display.update()  # 更新显示


# ------------------初始操作------------------#
def Begin1():
    music_begin()
    global button_begin
    while(not button_begin):
        Show_Begin1()

        for event in pygame.event.get():
            #开始
            if event.type == pygame.MOUSEBUTTONDOWN and 30<=event.pos[0]<=468 \
                    and 200<=event.pos[1]<=345:
                button_begin = True
            #结束
            if event.type == pygame.MOUSEBUTTONDOWN and 30<=event.pos[0]<=468 \
                    and 400<=event.pos[1]<=545 or event.type == pygame.QUIT:
                sys.exit()
                # 判断按钮(声音)
            if event.type == pygame.MOUSEBUTTONDOWN and 430 <= event.pos[0] <= 500 \
                    and 620 <= event.pos[1] <= 700:
                global button_music
                if button_music == True:
                    button_music = False
                    music_begin()
                else:
                    button_music = True
                    music_begin()


# ------------------难度选择界面------------------#
def Show_Begin2():
    # 背景
    background = pygame.image.load("assets/background.png")
    screen.blit(background, (0, 0))
    screen.blit(background, (390, 0))  # 图片不够大，所以填充两次
    # 小鸟
    screen.blit(bird.birdstatus[bird.status], (20, 35))
    #字体
    str = "Choose Difficulty"
    topic = font_75.render(str, 1, (255, 255, 255))
    screen.blit(topic, (65,30))
    #难度选择按钮
    easy = pygame.image.load("游戏按钮/简单.png")
    normal = pygame.image.load("游戏按钮/普通.png")
    difficult = pygame.image.load("游戏按钮/困难.png")
    screen.blit(easy, (75, 200))
    screen.blit(normal, (75, 360))
    screen.blit(difficult, (75, 520))
    #声音按钮
    sound = pygame.image.load("游戏按钮/声音.png")
    mute = pygame.image.load("游戏按钮/静音.png")
    global  button_music
    if button_music:
        screen.blit(sound, (430, 620))
    else:
        screen.blit(mute, (430, 620))

    pygame.display.update()  # 更新显示


# ------------------难度选择操作------------------#
def Begin2():
    global button_choice, difficulty
    while(not button_choice):
        Show_Begin2()

        for event in pygame.event.get():
            #简单
            if event.type == pygame.MOUSEBUTTONDOWN and 75<=event.pos[0]<=425 \
                    and 200<=event.pos[1]<=350:
                button_choice = True
                difficulty = 0
            #普通
            if event.type == pygame.MOUSEBUTTONDOWN and 75 <= event.pos[0] <= 425 \
                    and 360 <= event.pos[1] <= 510:
                button_choice = True
                difficulty = 1
            #困难
            if event.type == pygame.MOUSEBUTTONDOWN and 75 <= event.pos[0] <= 425 \
                    and 520 <= event.pos[1] <= 670:
                button_choice = True
                difficulty = 2

            # 判断按钮(声音)
            if event.type == pygame.MOUSEBUTTONDOWN and 430 <= event.pos[0] <= 500 \
                    and 620 <= event.pos[1] <= 700:
                global button_music
                if button_music == True:
                    button_music = False
                    music_begin()
                else:
                    button_music = True
                    music_begin()
            #结束
            if event.type == pygame.QUIT:
                sys.exit()


# ------------------更新背景------------------#
def Show():
    # 背景
    background = pygame.image.load("assets/background.png")
    screen.blit(background, (0, 0))
    screen.blit(background, (390, 0))  # 图片不够大，所以填充两次
    # 小鸟
    screen.blit(bird.birdstatus[bird.status], (bird.X, bird.Y))
    # 管道
    screen.blit(pipeline.PipeUp, (pipeline.X, pipeline.Up_Y))  # 显示上管道
    screen.blit(pipeline.PipeDown, (pipeline.X, pipeline.Down_Y))
    # 显示得分、速度、难度
    Score = font_50.render("Score:" + str(score), 1, (255, 255, 255))  # 内容，平滑，颜色
    Speed = font_50.render("Speed:" + str(round(speed, 2)), 1, (255, 255, 255))
    Difficult = font_50.render("Difficulty:" + str(difficulty), 1, (255, 255, 255))
    screen.blit(Score, (0, 0))
    screen.blit(Speed, (0, 50))
    screen.blit(Difficult, (0, 100))

    #暂停按钮
    global stop
    if button_stop:
        stop = pygame.image.load("游戏按钮/取消暂停.png")
        stop2 = pygame.image.load("游戏按钮/暂停2.png")
        screen.blit(stop2, (140, 200))
    else:
        stop = pygame.image.load("游戏按钮/暂停.png")
    screen.blit(stop, (423, 0))

    # 声音按钮
    sound = pygame.image.load("游戏按钮/声音.png")
    mute = pygame.image.load("游戏按钮/静音.png")
    if button_music:
        screen.blit(sound, (430, 620))
    else:
        screen.blit(mute, (430, 620))

    pygame.display.update()  # 更新显示


# ------------------游戏结束------------------#
def Show_Last():
    #文字
    str1 = "Game Over!"
    str2 = "Your score is:" + str(score)
    result1 = font_50.render(str1, 1, (0, 0, 0))
    result2 = font_50.render(str2, 1, (0, 0, 0))
    screen.blit(result1, (155, 200))
    screen.blit(result2, (130, 280))

    #重新开始
    restart = pygame.image.load("游戏按钮/重新开始.png")
    screen.blit(restart, (120, 350))

    #退出
    quit = pygame.image.load("游戏按钮/退出2.png")
    screen.blit(quit, (260, 350))

    #声音
    sound = pygame.image.load("游戏按钮/声音.png")
    mute = pygame.image.load("游戏按钮/静音.png")
    if button_music:
        screen.blit(sound, (430, 620))
    else:
        screen.blit(mute, (430, 620))

    pygame.display.update()


# ------------------最终处理------------------#
def end():
    music_end()
    global button_restart
    while(not button_restart):
        Show_Last()
        for event in pygame.event.get():
            #重新开始
            if event.type == pygame.MOUSEBUTTONDOWN and 120<=event.pos[0]<=256\
                    and 350<=event.pos[1]<=435:
                button_restart = True

            #退出程序
            if event.type == pygame.MOUSEBUTTONDOWN and 260<=event.pos[0]<=396\
                    and 350<=event.pos[1]<=433 or event.type == pygame.QUIT:
                sys.exit()

            #音乐
            if event.type == pygame.MOUSEBUTTONDOWN and 430<=event.pos[0]<=500\
                    and 620<=event.pos[1]<=700:
                global button_music
                if button_music == True:
                    button_music = False
                    music_end()
                else:
                    button_music = True
                    music_end()


# ------------------游戏暂停------------------#
def Stop():
    global button_stop
    while button_stop:
        Show()
        for event in pygame.event.get():
            # 取消暂停
            if event.type == pygame.MOUSEBUTTONDOWN and 423 <= event.pos[0] <= 500 \
                and 0 <= event.pos[1] <= 72:
                button_stop = False
                Delay()

            # 判断按钮(声音)
            if event.type == pygame.MOUSEBUTTONDOWN and 430 <= event.pos[0] <= 500 \
                    and 620 <= event.pos[1] <= 700:
                global button_music
                if button_music == True:
                    button_music = False
                    music_ing()
                else:
                    button_music = True
                    music_ing()

            #退出程序
            if event.type == pygame.QUIT:
                sys.exit()


# ------------------退出程序------------------#
def Judge():
    if not bird.dead:
        #执行
        execute()
        # 游戏暂停
        Stop()

    else:
        # 游戏结束
        end()

        global button_restart
        if button_restart:
            button_restart = False
            # 重启程序
            main()


# ------------------初始化------------------#
def Init():
    # 创建小鸟实例
    global bird
    bird = Bird()

    # 状态初始化
    global speed, score, button_begin, button_restart, button_choice, difficulty
    speed = 5           # 初始速度
    score = 0           # 初始分数
    difficulty = 0      # 简单难度
    button_begin = False
    button_restart = False
    button_choice = False


# ------------------主程序------------------#
def main():
    #初始化
    Init()

    # 图像处理
    image()

    # 初始菜单
    Begin1()
    #难度选择
    Begin2()

    # 创建管道类实例
    global pipeline
    pipeline = Pipeline()

    # 更新屏幕
    Show()
    # 播放音乐
    music_ing()
    # 延时
    Delay()

    # 设置时间控制
    clock = pygame.time.Clock()

    # 确保窗口一直显示
    while True:
        clock.tick(60)

        # 事件检测
        for event in pygame.event.get():
            # 窗口退出事件
            if event.type == pygame.QUIT:
                sys.exit()
            # 鼠标或键盘按下
            if event.type == pygame.KEYDOWN or event.type == pygame.MOUSEBUTTONDOWN:
                bird.jump = True
                bird.down = 5
                bird.up = 10
            # 判断按钮(暂停)
            if event.type == pygame.MOUSEBUTTONDOWN and 423 <= event.pos[0] <= 500 \
                    and 0 <= event.pos[1] <= 72:
                global  button_stop
                button_stop = True
            #判断按钮(声音)
            if event.type == pygame.MOUSEBUTTONDOWN and 430 <= event.pos[0] <= 500 \
                    and 620 <= event.pos[1] <= 700:
                global  button_music
                if button_music == True:
                    button_music = False
                    music_ing()
                else:
                    button_music = True
                    music_ing()

        # 判断处理
        Judge()

    # 退出程序
    pygame.quit()


# ------------------主程序------------------#
if __name__ == '__main__':
    # 初始化pygame
    pygame.init()
    # 设置并显示窗口
    size = width, height = 500, 700
    screen = pygame.display.set_mode(size)

    # 初始化pygame字体类
    pygame.font.init()
    font_50 = pygame.font.SysFont(None, 50)  # 默认字体，大小50
    font_75 = pygame.font.SysFont(None, 75)  # 默认字体，大小75
    font_100 = pygame.font.SysFont(None, 100)  # 默认字体，大小为100

    # 初始化音乐类
    pygame.mixer.init()

    #主程序
    main()
