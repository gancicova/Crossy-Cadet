

import pygame as p
import time


class Cadet(p.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.x = 50
        self.y = HEIGHT / 2
        self.vel = 4
        self.width = 64
        self.height = 64

        # IMAGES

        self.cadet1 = p.image.load('cadet1.png')
        self.cadet2 = p.image.load('cadet2.png')
        self.cadet1 = p.transform.scale(self.cadet1, (self.width, self.height))
        self.cadet2 = p.transform.scale(self.cadet2, (self.width, self.height))

        self.image = self.cadet1
        self.rect = self.image.get_rect()
        self.mask = p.mask.from_surface(self.image)

    def update(self):
        self.movement()
        self.correction()
        self.checkCollision()
        self.rect.center = (self.x, self.y)

    def movement(self):
        keys = p.key.get_pressed()
        if keys[p.K_LEFT]:
            self.x -= self.vel
            self.image = self.cadet2

        elif keys[p.K_RIGHT]:
            self.x += self.vel
            self.image = self.cadet1

        if keys[p.K_UP]:
            self.y -= self.vel

        elif keys[p.K_DOWN]:
            self.y += self.vel

    def correction(self):
        if self.x - self.width / 2 < 0:
            self.x = self.width / 2

        elif self.x + self.width / 2 > WIDTH:
            self.x = WIDTH - self.width / 2

        if self.y - self.height / 2 < 0:
            self.y = self.height / 2

        elif self.y + self.height / 2 > HEIGHT:
            self.y = HEIGHT - self.height / 2

    def checkCollision(self):
        car_check = p.sprite.spritecollide(self, car_group, False, p.sprite.collide_mask)
        if car_check:
            explosion.explode(self.x, self.y)


class Car(p.sprite.Sprite):
    def __init__(self, number):
        super().__init__()
        if number == 1:
            self.x = 190
            self.image = p.image.load('Slow Car.png')
            self.vel = -4

        else:
            self.x = 460
            self.image = p.image.load('Fast Car.png')
            self.vel = 6

        self.y = HEIGHT / 2
        self.width = 80
        self.height = 80
        self.image = p.transform.scale(self.image, (self.width, self.height))
        self.rect = self.image.get_rect()
        self.mask = p.mask.from_surface(self.image)

    def update(self):
        self.movement()
        self.rect.center = (self.x, self.y)

    def movement(self):
        self.y += self.vel

        if self.y - self.height / 2 < 0:
            self.y = self.height / 2
            self.vel *= -1

        elif self.y + self.height / 2 > HEIGHT:
            self.y = HEIGHT - self.height / 2
            self.vel *= -1


class Screen(p.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.img1 = p.image.load('Scene.png')
        self.img2 = p.image.load('You Win.png')
        self.img3 = p.image.load('You lose.png')

        self.img1 = p.transform.scale(self.img1, (WIDTH, HEIGHT))
        self.img2 = p.transform.scale(self.img2, (WIDTH, HEIGHT))
        self.img3 = p.transform.scale(self.img3, (WIDTH, HEIGHT))

        self.image = self.img1
        self.x = 0
        self.y = 0

        self.rect = self.image.get_rect()

    def update(self):
        self.rect.topleft = (self.x, self.y)


class Flag(p.sprite.Sprite):
    def __init__(self, number):
        super().__init__()
        self.number = number

        if self.number == 1:
            self.image = p.image.load('green flag.png')
            self.visible = False
            self.x = 50

        else:
            self.image = p.image.load('white flag.png')
            self.visible = True
            self.x = 580

        self.y = HEIGHT / 2
        self.image = p.transform.scale2x(self.image)
        self.rect = self.image.get_rect()
        self.mask = p.mask.from_surface(self.image)

    def update(self):
        if self.visible:
            self.collision()
            self.rect.center = (self.x, self.y)

    def collision(self):
        global SCORE, cadet

        flag_hit = p.sprite.spritecollide(self, cadet_group, False, p.sprite.collide_mask)
        if flag_hit:
            self.visible = False

            if self.number == 1:
                white_flag.visible = True
                if SCORE < 5:
                    SwitchLevel()

                else:
                    cadet_group.empty()
                    DeleteOtherItems()

                    EndScreen(1)

            else:
                green_flag.visible = True


class Explosion(object):
    def __init__(self):
        self.costume = 1
        self.width = 140
        self.height = 140
        self.image = p.image.load('explosion' + str(self.costume) + '.png')
        self.image = p.transform.scale(self.image, (self.width, self.height))

    def explode(self, x, y):
        x = x - self.width / 2
        y = y - self.height / 2
        DeleteCadet()

        while self.costume < 9:
            self.image = p.image.load('explosion' + str(self.costume) + '.png')
            self.image = p.transform.scale(self.image, (self.width, self.height))
            win.blit(self.image, (x, y))
            p.display.update()

            self.costume += 1
            time.sleep(0.1)

        DeleteOtherItems()
        EndScreen(0)


def ScoreDisplay():
    global gameOn

    if gameOn:
        score_text = score_font.render(str(SCORE) + ' / 6', True, (0, 0, 0))
        win.blit(score_text, (210, 390))


def checkFlags():
    for flag in flags:
        if not flag.visible:
            flag.kill()

        else:
            if not flag.alive():
                flag_group.add(flag)


def SwitchLevel():
    global SCORE

    if slow_car.vel < 0:
        slow_car.vel -= 1

    else:
        slow_car.vel += 1

    if fast_car.vel < 0:
        fast_car.vel -= 1

    else:
        fast_car.vel += 1

    SCORE += 1


def DeleteCadet():
    global cadet

    cadet.kill()
    screen_group.draw(win)
    car_group.draw(win)
    flag_group.draw(win)

    screen_group.update()
    car_group.update()
    flag_group.update()

    p.display.update()


def DeleteOtherItems():
    car_group.empty()
    flag_group.empty()
    flags.clear()


def EndScreen(n):
    global gameOn

    gameOn = False

    if n == 0:
        bg.image = bg.img3

    elif n == 1:
        bg.image = bg.img2


WIDTH = 640
HEIGHT = 480

p.mixer.pre_init(44100,16,2,4096)
p.init()

win = p.display.set_mode((WIDTH, HEIGHT))
p.display.set_caption('Crossy Cadet')
clock = p.time.Clock()

p.mixer.music.load("Old Post.mp3")
p.mixer.music.set_volume(.75)
p.mixer.music.play(-1)

SCORE = 0
score_font = p.font.SysFont('TimesNewRoman', 80, True)

bg = Screen()
screen_group = p.sprite.Group()
screen_group.add(bg)

cadet = Cadet()
cadet_group = p.sprite.Group()
cadet_group.add(cadet)

slow_car = Car(1)
fast_car = Car(2)
car_group = p.sprite.Group()
car_group.add(slow_car, fast_car)

green_flag = Flag(1)
white_flag = Flag(2)
flag_group = p.sprite.Group()
flag_group.add(green_flag, white_flag)
flags = [green_flag, white_flag]

explosion = Explosion()

gameOn = True
run = True
while run:
    clock.tick(60)
    for event in p.event.get():
        if event.type == p.QUIT:
            run = False

    screen_group.draw(win)

    ScoreDisplay()
    checkFlags()

    car_group.draw(win)
    cadet_group.draw(win)
    flag_group.draw(win)

    car_group.update()
    cadet_group.update()
    flag_group.update()

    screen_group.update()

    p.display.update()

p.quit()

