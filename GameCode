import numpy as np
import numpy.random
import pygame
import sys
from numba import njit

# Setup pygame/window #
mainClock = pygame.time.Clock()
from pygame.locals import *

pygame.init()
pygame.display.set_caption('Game Base')

# Main Menu Window
menu_screen = pygame.display.set_mode((1280, 720), 0, 32)

font = pygame.font.SysFont(None, 36)


def draw_text(text, font, color, surface, x, y):
    textobj = font.render(text, 1, color)
    textrect = textobj.get_rect()
    textrect.center = (x, y)
    surface.blit(textobj, textrect)


click = False


def main_menu():
    global click
    while True:
        menu_screen.fill((0, 0, 0))
        draw_text('Main Menu', font, (255, 255, 255), menu_screen, 640, 50)

        mx, my = pygame.mouse.get_pos()

        button_width, button_height = 300, 100
        button_1 = pygame.Rect((menu_screen.get_width() - button_width) // 2, 200, button_width, button_height)
        button_2 = pygame.Rect((menu_screen.get_width() - button_width) // 2, 350, button_width, button_height)
        if button_1.collidepoint((mx, my)):
            if click:
                open_game_window()
        if button_2.collidepoint((mx, my)):
            if click:
                options()

        pygame.draw.rect(menu_screen, (255, 0, 0), button_1)
        pygame.draw.rect(menu_screen, (255, 0, 0), button_2)

        draw_text('Play', font, (255, 255, 255), menu_screen, menu_screen.get_width() // 2, 250)
        draw_text('Options', font, (255, 255, 255), menu_screen, menu_screen.get_width() // 2, 400)

        click = False
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
            if event.type == KEYDOWN:
                if event.key == K_ESCAPE:
                    pygame.quit()
                    sys.exit()
            if event.type == MOUSEBUTTONDOWN:
                if event.button == 1:
                    click = True

        pygame.display.update()
        mainClock.tick(60)


def open_game_window():
    game_screen = pygame.display.set_mode((1280, 720), 0, 32)

    running = True
    hRes = 120  # horizontal resolution
    halfVres = 100  # vertical resolution divided by 2
    mod = hRes / 60  # scaling factor 60 fov
    size = 25
    posx, posy, rot, maph, mapc, exitx, exity = gen_map(size)

    """
    maph = numpy.random.choice([0, 0, 0, 1], (size, size))

    # making walls different colors
    mapc = np.random.uniform(0, 1, (size, size, 3))
    """
    frame = numpy.random.uniform(0, 1, (hRes, halfVres * 2, 3))
    sky = pygame.image.load('skybox.png')
    sky = pygame.surfarray.array3d(pygame.transform.scale(sky, (360, halfVres * 2))) / 255
    floor = pygame.surfarray.array3d(pygame.image.load('floor.jpg')) / 255
    wall = pygame.surfarray.array3d(pygame.image.load('wall2.jpg')) / 255

    while running:
        if int(posx) == exitx and int(posy) == exity:
            print("your not that good relax buddy")
            running = False
        for event in pygame.event.get():
            if event.type == pygame.QUIT or event.type == pygame.KEYDOWN and event.key == pygame.K_ESCAPE:
                running = False

        frame = new_frame(posx, posy, rot, frame, sky, floor, hRes, halfVres, mod, maph, size, wall, mapc, exity, exitx)

        surf = pygame.surfarray.make_surface(frame * 255)
        surf = pygame.transform.scale(surf, (1280, 720))
        fps = int(mainClock.get_fps())
        pygame.display.set_caption("Jason's winter Project 2024 - FPS: " + str(fps))

        game_screen.blit(surf, (0, 0))
        pygame.display.update()

        posx, posy, rot = movement(posx, posy, rot, maph, mainClock.tick()/500)

    # Return to the main menu after the game loop exits
    main_menu()


def movement(posx, posy, rot, maph, et):
    # Get currently pressed keys
    pressed_keys = pygame.key.get_pressed()
    x, y, diag = posx, posy, rot

    # Update rotation based on mouse movement
    p_mouse = pygame.mouse.get_rel()
    rot = rot + np.clip((p_mouse[0]) / 200, -0.2, 0.2)

    # Handle movement in the forward direction
    if pressed_keys[pygame.K_UP] or pressed_keys[ord('w')]:
        x, y, diag = x + et * np.cos(rot), y + et * np.sin(rot), 1

    # Handle movement in the backward direction
    elif pressed_keys[pygame.K_DOWN] or pressed_keys[ord('s')]:
        x, y, diag = x - et * np.cos(rot), y - et * np.sin(rot), 1

    # Handle strafing left
    if pressed_keys[pygame.K_LEFT] or pressed_keys[ord('a')]:
        et = et / (diag + 1)
        x, y = x + et * np.sin(rot), y - et * np.cos(rot)

    # Handle strafing right
    elif pressed_keys[pygame.K_RIGHT] or pressed_keys[ord('d')]:
        et = et / (diag + 1)
        x, y = x - et * np.sin(rot), y + et * np.cos(rot)

    # Check collisions with walls and update position
    if not (maph[int(x - 0.2)][int(y)] or maph[int(x + 0.2)][int(y)] or
            maph[int(x)][int(y - 0.2)] or maph[int(x)][int(y + 0.2)]):
        posx, posy = x, y
    elif not (maph[int(posx - 0.2)][int(y)] or maph[int(posx + 0.2)][int(y)] or
              maph[int(posx)][int(y - 0.2)] or maph[int(posx)][int(y + 0.2)]):
        posy = y
    elif not (maph[int(x - 0.2)][int(posy)] or maph[int(x + 0.2)][int(posy)] or
              maph[int(x)][int(posy - 0.2)] or maph[int(x)][int(posy + 0.2)]):
        posx = x

    return posx, posy, rot


def gen_map(size):
    mapc = numpy.random.uniform(0, 1, (size, size, 3))
    maph = numpy.random.choice([0, 0, 0, 0, 1, 1], (size, size))
    maph[0, :], maph[size - 1, :], maph[:, 0], maph[:, size - 1] = (1, 1, 1, 1)

    posx, posy, rot = 1.5, numpy.random.randint(1, size - 1) + .5, np.pi / 4
    x, y = int(posx), int(posy)
    maph[x][y] = 0
    count = 0
    while True:
        testx, testy = (x, y)
        if np.random.uniform() > 0.5:
            testx = testx + np.random.choice([-1, 1])
        else:
            testy = testy + np.random.choice([-1, 1])
        if 0 < testx < size - 1 and 0 < testy < size - 1:
            if maph[testx][testy] == 0 or count > 5:
                count = 0
                x, y = (testx, testy)
                maph[x][y] = 0
                if x == size - 2:
                    exitx, exity = (x, y)
                    break
            else:
                count = count + 1
    return posx, posy, rot, maph, mapc, exitx, exity


@njit()
def new_frame(posx, posy, rot, frame, sky, floor, hres, halfvres, mod, maph, size, wall, mapc, exity, exitx):
    for i in range(hres):
        rot_i = rot + numpy.deg2rad(i / mod - 30)
        sin, cos, cos2 = numpy.sin(rot_i), numpy.cos(rot_i), numpy.cos(numpy.deg2rad(i / mod - 30))
        frame[i][:] = sky[int(numpy.rad2deg(rot_i) % 359)][:]

        x, y = posx, posy
        while maph[int(x) % (size - 1)][int(y) % (size - 1)] == 0:
            x, y = x + 0.02 * cos, y + 0.02 * sin

        n = abs((x - posx) / cos)
        h = int(halfvres / (n * cos2 + 0.001))

        xx = int(x * 3 % 1 * 99)
        if x % 1 < 0.02 or x % 1 > 0.98:
            xx = int(y * 3 % 1 * 99)
        yy = numpy.linspace(0, 108, h * 2) % 99

        shade = 0.3 + 0.7 * (h / halfvres)
        # fixes shading for certain walls
        if shade > 1:
            shade = 1
        # wall shadows
        if maph[int(x - 0.01) % (size - 1)][int(y - 0.01) % (size - 1)]:
            shade = shade * 0.5

        c = shade * mapc[int(x) % (size - 1)][int(y) % (size - 1)]

        for k in range(h * 2):
            if 0 <= halfvres - h + k < 2 * halfvres:
                frame[i][halfvres - h + k] = c * wall[xx][int(yy[k])]

        for j in range(halfvres - h):
            n = (halfvres / (halfvres - j)) / cos2
            x, y = posx + cos * n, posy + sin * n
            xx, yy = int(x * 2 % 1 * 99), int(y * 2 % 1 * 99)

            shade = 0.2 + 0.8 * (1 - j / halfvres)
            # casting shadows to the floor
            if maph[int(x - 0.33) % (size - 1)][int(y - 0.33) % (size - 1)]:
                shade = shade * 0.5
            elif (maph[int(x - 0.33) % (size - 1)][int(y) % (size - 1)] and y % 1 > x % 1) or \
                    (maph[int(x) % (size - 1)][int(y - 0.33) % (size - 1)] and x % 1 > y % 1):
                shade = shade * 0.5

            frame[i][halfvres * 2 - j - 1] = shade * (floor[xx][yy] + frame[i][halfvres * 2 - j - 1]) / 2
            if int(x) == exitx and int(y) == exity and (x % 1 - 0.5) ** 2 + (y % 1 - 0.5) ** 2 < 0.2:
                ee = j / (10 * halfvres)
                frame[i][j:2 * halfvres - j] = (ee * np.ones(3) + frame[i][j:2 * halfvres - j]) / (1 + ee)

            # checking for walls

    return frame


def get_sprites(hres, num_sprite_sheets):
    sprites = []

    for sheet_num in range(1, num_sprite_sheets + 1):
        sheet_name = f"{sheet_num}ZombieSpriteSheet.png"
        sheet = pygame.image.load(sheet_name).convert_alpha()
        sprites.append([[], []])

        xx = 0
        for i in range(3):
            sprites[sheet_num - 1][0].append([])
            sprites[sheet_num - 1][1].append([])
            for j in range(4):
                yy = j * 100
                sprites[sheet_num - 1][0][i].append(pygame.Surface.subsurface(sheet, (xx, yy, 32, 100)))
                sprites[sheet_num - 1][1][i].append(pygame.Surface.subsurface(sheet, (xx + 96, yy, 32, 100)))

            xx += 32

    spsize = np.asarray(sprites[0][0][1][0].get_size()) * hres / 800

    return sprites, spsize

def options():
    running = True
    while running:
        menu_screen.fill((0, 0, 0))

        draw_text('Options', font, (255, 255, 255), menu_screen, menu_screen.get_width() // 2, 50)
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
            if event.type == KEYDOWN:
                if event.key == K_ESCAPE:
                    running = False

        pygame.display.update()
        mainClock.tick(60)


if __name__ == '__main__':
    main_menu()
