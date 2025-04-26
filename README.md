from random import randint
import time
import pgzrun
from pgzero.builtins import Actor

# set the size of the screen
WIDTH = 800
HEIGHT = 600
CENTER_X = WIDTH / 2
CENTER_Y = HEIGHT / 2

# define the variables
game_over = False
finalized = False
garden_happy = True
fangflower_collision = False
kirby_collision = False
time_elapsed = 0
start_time = time.time()
can_reset_cow = True 
raining = True  # set if it's raining

# create the cow actor
cow = Actor("cow")
cow.pos = 100, 500

# create the lists for different objects
flower_list = []
wilted_list = []
fangflower_list = []
fangflower_vy_list = []
fangflower_vx_list = []
kirby_list = []
kirby_vy_list = []
kirby_vx_list = []

# update the screen
def draw():
    global game_over, time_elapsed, finalized
    screen.clear()
    if not raining:
        screen.blit("garden", (0, 0))
    else:
        screen.blit("garden-raining", (0, 0))

if not game_over:
        cow.draw()
        for flower in flower_list:
            flower.draw()
        for fangflower in fangflower_list:
            fangflower.draw()
        for kirby in kirby_list:
            kirby.draw()
        time_elapsed = int(time.time() - start_time)
        screen.draw.text("Garden happy for: " + str(time_elapsed) + " seconds", topleft=(10, 10), color="black")
else:
        cow.draw()
        screen.draw.text("Garden happy for: " + str(time_elapsed) + " seconds", topleft=(10, 10), color="black")
        if not finalized:
            if not garden_happy:
                screen.draw.text("GARDEN UNHAPPY—GAME OVER!", color="black", topleft=(10, 50))
            elif kirby_collision:
                screen.draw.text("KIRBY ATTACK—GAME OVER!", color="black", topleft=(10, 50))
            else:
                screen.draw.text("FANGFLOWER ATTACK—GAME OVER!", color="black", topleft=(10, 50))
            finalized = True

# generate a new flower
def new_flower():
    global flower_list, wilted_list
    flower_new = Actor("flower")
    flower_new.pos = randint(50, WIDTH - 50), randint(150, HEIGHT - 100)
    flower_list.append(flower_new)
    wilted_list.append("happy")

# add a flower every period
def add_flowers():
    global game_over
    if not game_over:
        new_flower()
        clock.schedule(add_flowers, 4)  # add a new flower every 4 seconds

# check if any flower has wilted for too long
def check_wilt_times():
    global wilted_list, game_over, garden_happy
    if raining:
        return
    if wilted_list:
        for wilted_since in wilted_list:
            if wilted_since != "happy":
                time_wilted = int(time.time() - wilted_since)
                if time_wilted > 10.0:
                    garden_happy = False
                    game_over = True
                    break

# randomly wilt a flower
def wilt_flower():
    global flower_list, wilted_list, game_over
    if not game_over:
        if flower_list:
            rand_flower = randint(0, len(flower_list) - 1)
            if flower_list[rand_flower].image == "flower":
                flower_list[rand_flower].image = "flower-wilt"
                wilted_list[rand_flower] = time.time()
        clock.schedule(wilt_flower, 3)  # wilt a flower every 3 seconds

# water a wilted flower if colliding with cow
def check_flower_collision():
    global cow, flower_list, wilted_list
    index = 0
    for flower in flower_list:
        if flower.colliderect(cow) and flower.image == "flower-wilt":
            flower.image = "flower"
            wilted_list[index] = "happy"
            break
        index += 1

# check if cow is attacked by fangflower
def check_fangflower_collision():
    global cow, fangflower_list, fangflower_collision, game_over, garden_happy
    for fangflower in fangflower_list:
        if fangflower.colliderect(cow):
            cow.image = "zap"
            fangflower_collision = True
            game_over = True
            garden_happy = True
            break

# check if cow is attacked by kirby
def check_kirby_collision():
    global cow, kirby_list, kirby_collision, game_over, garden_happy
    for kirby in kirby_list:
        if kirby.colliderect(cow):
            cow.image = "zap"
            kirby_collision = True
            game_over = True
            garden_happy = True
            break

# set a random velocity for enemies
def velocity():
    random_dir = randint(0, 1)
    random_velocity = randint(2, 3)
    return -random_velocity if random_dir == 0 else random_velocity

# mutate a flower into a fangflower
def mutate_fangflower():
    global flower_list, fangflower_list, fangflower_vy_list, fangflower_vx_list, game_over
    if not game_over and flower_list:
        rand_flower = randint(0, len(flower_list) - 1)
        fangflower_pos_x = flower_list[rand_flower].x
        fangflower_pos_y = flower_list[rand_flower].y
        del flower_list[rand_flower]
        fangflower = Actor("fangflower")
        fangflower.pos = fangflower_pos_x, fangflower_pos_y
        fangflower_vx = velocity()
        fangflower_vy = velocity()
        fangflower_list.append(fangflower)
        fangflower_vx_list.append(fangflower_vx)
        fangflower_vy_list.append(fangflower_vy)
        clock.schedule(mutate_fangflower, 2)  # mutate every 2 seconds

# mutate a flower into kirby
def mutate_kirby():
    global flower_list, kirby_list, kirby_vy_list, kirby_vx_list, game_over
    if not game_over and flower_list:
        rand_flower = randint(0, len(flower_list) - 1)
        kirby_pos_x = flower_list[rand_flower].x
        kirby_pos_y = flower_list[rand_flower].y
        del flower_list[rand_flower]
        kirby = Actor("kirby")
        kirby.pos = kirby_pos_x, kirby_pos_y
        kirby_vx = velocity()
        kirby_vy = velocity()
        kirby_list.append(kirby)
        kirby_vx_list.append(kirby_vx)
        kirby_vy_list.append(kirby_vy)
        clock.schedule(mutate_kirby, 20)  # mutate every 20 seconds

# update fangflowers' positions
def update_fangflowers():
    global fangflower_list, game_over
    if not game_over:
        index = 0
        for fangflower in fangflower_list:
            fangflower.x += fangflower_vx_list[index]
            fangflower.y += fangflower_vy_list[index]
            if fangflower.left < 0 or fangflower.right > WIDTH:
                fangflower_vx_list[index] = -fangflower_vx_list[index]
            if fangflower.top < 150 or fangflower.bottom > HEIGHT:
                fangflower_vy_list[index] = -fangflower_vy_list[index]
            index += 1

# update kirbys' positions
def update_kirbys():
    global kirby_list, game_over
    if not game_over:
        index = 0
        for kirby in kirby_list:
            kirby.x += kirby_vx_list[index]
            kirby.y += kirby_vy_list[index]
            if kirby.left < 0 or kirby.right > WIDTH:
                kirby_vx_list[index] = -kirby_vx_list[index]
            if kirby.top < 150 or kirby.bottom > HEIGHT:
                kirby_vy_list[index] = -kirby_vy_list[index]
            index += 1

# reset cow's image to normal
def reset_cow():
    if not game_over:
        cow.image = "cow"

# schedule flower adding and wilting
add_flowers()
wilt_flower()

# update game state
def update(): 
    check_wilt_times()
    check_fangflower_collision()
    check_kirby_collision()
    update_fangflowers()
    update_kirbys()
    
  if not game_over:
  if keyboard.space:
  cow.image = "cow-water"
  clock.schedule(reset_cow, 0.5)
  check_flower_collision()

if keyboard.left and cow.x > 0:
cow.x -= 5
elif keyboard.right and cow.x < WIDTH:
cow.x += 5
elif keyboard.up and cow.y > 150:
cow.y -= 5
elif keyboard.down and cow.y < HEIGHT:
cow.y += 5

if time_elapsed > 15 and not fangflower_list:
mutate_fangflower()
if time_elapsed > 10 and not kirby_list:
mutate_kirby()

# start the game
pgzrun.go()
