---
title: pygame
toc: true
date: 2018-06-11 08:14:30
---
# 可以补充进来的

- 其实我一直觉得，这个模拟一个游戏，然后在这个模拟里面进行加速训练，这样的做法有用是有用，但是现实世界能完全模拟吗？
- 模拟场景的开发感觉还是有些重要的，它能够加速训练，尽管也许不能 100% 的模拟，但是能够加速，这个。。
- 不知道模拟场景会不会影响真实世界？比如一个 智能体在 模拟场景中训练了，但是，这个模拟场景已经是 99.9999% 的真实了，但是还不是最真实的，那么他在真实世界中会出问题吗？还是说这种容错是可以允许的？毕竟人类连 99% 都做不到，大多数人根本不管这个世界是不是真实的。

# pygame


在做强化学习的时候，会要做一些小游戏，比如 flappybird，就有人用 pygame 做了一个这个游戏，然后用 DQN 来学习，还有那个经典的立杆子的游戏也是 pygame 做的。。因此 pygame 还是要会用的，有时候自己想做一些小游戏进行训练的时候就可以自己做了。


## 要点：


现在暂时没有花时间在这上面，只是 mark 一下，看到一个 pygame 的简单的例子，因此记在这里，后续的详细的进行补充。


```Python
import sys
import math
import pygame
from pygame.locals import *
pygame.init()
screen = pygame.display.set_mode((600,500))
pygame.display.set_caption("The Pie Game - Press 1,2,3,4")
myfont = pygame.font.Font(None, 60)

color = 200, 80, 60
width = 4
x = 300
y = 250
radius = 200
position = x-radius, y-radius, radius*2, radius*2

piece1 = False
piece2 = False
piece3 = False
piece4 = False

while True:
    for event in pygame.event.get():
        if event.type == QUIT:
            sys.exit()
        elif event.type == KEYUP:
            if event.key == pygame.K_ESCAPE:
                sys.exit()
            elif event.key == pygame.K_1:
                piece1 = True
            elif event.key == pygame.K_2:
                piece2 = True
            elif event.key == pygame.K_3:
                piece3 = True
            elif event.key == pygame.K_4:
                piece4 = True

    #clear the screen
    screen.fill((0,0,200))

    #draw the four numbers
    textImg1 = myfont.render("1", True, color)
    screen.blit(textImg1, (x+radius/2-20, y-radius/2))
    textImg2 = myfont.render("2", True, color)
    screen.blit(textImg2, (x-radius/2, y-radius/2))
    textImg3 = myfont.render("3", True, color)
    screen.blit(textImg3, (x-radius/2, y+radius/2-20))
    textImg4 = myfont.render("4", True, color)
    screen.blit(textImg4, (x+radius/2-20, y+radius/2-20))


    #should the pieces be drawn?
    if piece1:
        start_angle = math.radians(0)
        end_angle = math.radians(90)
        pygame.draw.arc(screen, color, position, start_angle, end_angle, width)
        pygame.draw.line(screen, color, (x,y), (x,y-radius), width)
        pygame.draw.line(screen, color, (x,y), (x+radius,y), width)
    if piece2:
        start_angle = math.radians(90)
        end_angle = math.radians(180)
        pygame.draw.arc(screen, color, position, start_angle, end_angle, width)
        pygame.draw.line(screen, color, (x,y), (x,y-radius), width)
        pygame.draw.line(screen, color, (x,y), (x-radius,y), width)
    if piece3:
        start_angle = math.radians(180)
        end_angle = math.radians(270)
        pygame.draw.arc(screen, color, position, start_angle, end_angle, width)
        pygame.draw.line(screen, color, (x,y), (x-radius,y), width)
        pygame.draw.line(screen, color, (x,y), (x,y+radius), width)
    if piece4:
        start_angle = math.radians(270)
        end_angle = math.radians(360)
        pygame.draw.arc(screen, color, position, start_angle, end_angle, width)
        pygame.draw.line(screen, color, (x,y), (x,y+radius), width)
        pygame.draw.line(screen, color, (x,y), (x+radius,y), width)

    #is the pie finished?
    if piece1 and piece2 and piece3 and piece4:
        color = 0,255,0

    pygame.display.update()
```
