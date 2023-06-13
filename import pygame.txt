import pygame
import random
import sys
import pygame_gui

# Initialize Pygame
pygame.init()

#Create Class Cookie 
class Cookie(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = pygame.Surface((10, 10))
        self.image.fill((255, 255, 0))
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
# Create the Ghost class
class Ghost(pygame.sprite.Sprite):
    def __init__(self, x, y, color):
        super().__init__()
        self.image = pygame.Surface((30, 30))
        self.image.fill(color)
        self.rect = self.image.get_rect(center = (x,y))
        self.rect.x = x
        self.rect.y = y
        self.direction = random.choice([(0, -1), (0, 1), (-1, 0), (1, 0)])
# Update function takes in the ghost, walls, and pacman as parameters
    def update(self, walls, player_rect):
        new_rect = self.rect.move(self.direction)
        if new_rect.collidelist(walls) == -1:
            self.rect = new_rect
        else:
            self.direction = random.choice([(0, -1), (0, 1), (-1, 0), (1, 0)])
        # Check for collision with pacman
        if self.rect.colliderect(player_rect):
            # End the game
            print("Game over!")
            show_game_over_screen()
            menu()

# Set up the game loop
def run_game():
    # Set up the Pac-Man character
    pacman_radius = 16
    pacman_position = [303,403]
    pacman_direction = [0, 0]

    #Initialize score to 0 and Font for the score
    score_font = pygame.font.Font(None, 36)
    score = 0

    # Set up the maze walls 
    maze_walls = [  
        pygame.Rect(0,0,6,600),
        pygame.Rect(0,0,600,6),
        pygame.Rect(0,600,606,6),
        pygame.Rect(600,0,6,606),
        pygame.Rect(300,0,6,66),
        pygame.Rect(60,60,186,6),
        pygame.Rect(360,60,186,6),
        pygame.Rect(60,120,66,6),
        pygame.Rect(60,120,6,126),
        pygame.Rect(180,120,246,6),
        pygame.Rect(300,120,6,66),
        pygame.Rect(480,120,66,6),
        pygame.Rect(540,120,6,126),
        pygame.Rect(120,180,126,6),
        pygame.Rect(120,180,6,126),
        pygame.Rect(360,180,126,6),
        pygame.Rect(480,180,6,126),
        pygame.Rect(180,240,6,126),
        pygame.Rect(180,360,246,6),
        pygame.Rect(420,240,6,126),
        pygame.Rect(240,240,42,6),
        pygame.Rect(324,240,42,6),
        pygame.Rect(240,240,6,66),
        pygame.Rect(240,300,126,6),
        pygame.Rect(360,240,6,66),
        pygame.Rect(0,300,66,6),
        pygame.Rect(540,300,66,6),
        pygame.Rect(60,360,66,6),
        pygame.Rect(60,360,6,186),
        pygame.Rect(480,360,66,6),
        pygame.Rect(480,180,6,126),
        pygame.Rect(540,360,6,186),
        pygame.Rect(120,420,366,6),
        pygame.Rect(120,420,6,66),
        pygame.Rect(480,420,6,66),
        pygame.Rect(180,480,246,6),
        pygame.Rect(300,480,6,66),
        pygame.Rect(120,540,126,6),
        pygame.Rect(360,540,126,6),      
    ]
    
    #Creating three ghosts of different colors and adding them to ghost group
    ghost_group = pygame.sprite.Group()
    red_ghost = Ghost(100, 150, (255, 0, 0))
    ghost_group.add(red_ghost)

    blue_ghost = Ghost(150, 100, (0, 0, 255))
    ghost_group.add(blue_ghost)

    green_ghost = Ghost(250, 150, (0, 255, 0))
    ghost_group.add(green_ghost)

        # Initialize cookie size and the gap between them
    co_size = 18
    co_gap = 30

    # Calculate the number of cookies that can fit on the screen horizontally and vertically
    num_cook_x = (screen_width - co_gap) // co_gap
    num_cook_y = (screen_height - co_gap) // co_gap

    # Calculate the starting positions for the cookies
    start_x = co_gap // 2
    start_y = co_gap // 2

    # Create the cookies
    cookie_group = pygame.sprite.Group()
    for i in range(num_cook_x):
        for j in range(num_cook_y):
            cook_x = start_x + i * co_gap + co_gap//2
            cook_y = start_y + j * co_gap + co_gap//2
            
            # Check if there is a wall at the cookies postition
            cook_rect = pygame.Rect(cook_x, cook_y, co_size, co_size)
            if not any(wall_rect.colliderect(cook_rect) for wall_rect in maze_walls):
                # Creates new cookie and adds it to the cookie group
                cookies = Cookie(cook_x, cook_y)
                cookie_group.add(cookies)

    running = True
    while running:
        if running == False:
            break
        # Using a for loop for the events in our game
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
                sys.exit()
            # Keyboard Input
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP:
                    pacman_direction = [0, -1]
                elif event.key == pygame.K_DOWN:
                    pacman_direction = [0, 1]
                elif event.key == pygame.K_LEFT:
                    pacman_direction = [-1, 0]
                elif event.key == pygame.K_RIGHT:
                    pacman_direction = [1, 0]

        # Move Pac-Man
        pacman_position[0] += pacman_direction[0] * 5
        pacman_position[1] += pacman_direction[1] * 5

        # Check for pacman colliding with the maze walls
        pacman_rect = pygame.Rect(pacman_position[0] - pacman_radius, pacman_position[1] - pacman_radius, pacman_radius * 2, pacman_radius * 2)
        for wall in maze_walls:
            if pacman_rect.colliderect(wall):
                pacman_position[0] -= pacman_direction[0] * 5
                pacman_position[1] -= pacman_direction[1] * 5
                break
        # Checking if pacman collides with cookies then removing them and adding to score
        for cookies in cookie_group:
            if pacman_rect.colliderect(cookies.rect):
                cookie_group.remove(cookies)
                score += 1
            # Once all cookies are "eaten" game over
            if len(cookie_group) == 0:
                print("You won!")
                next_lvl()
                

        # Checking for collisions with walls and pacman
        for ghost in ghost_group:
            ghost_group.update(maze_walls, pacman_rect)

    
        # Black background
        screen.fill(black)

        # Draw the maze walls
        for wall in maze_walls:
            pygame.draw.rect(screen, blue, wall)

        #Draw ghosts and cookies on screen
        ghost_group.draw(screen)
        cookie_group.draw(screen) 

        # Draw Pac-Man
        pygame.draw.circle(screen, yellow, pacman_position, pacman_radius)

        #Add the score 
        score_text = score_font.render("Score: {}".format(score), True, (255, 255, 255))
        screen.blit(score_text, (10, 10))


        # Update the screen
        pygame.display.update()

        # Tick the clock
        clock.tick(60)

    # Quit Pygame
    pygame.quit()

def next_lvl():

     # Set up the Pac-Man character
    pacman_radius = 16
    pacman_position = [303,403]
    pacman_direction = [0, 0]

    #Initialize score to 0 and Font for the score
    score_font = pygame.font.Font(None, 36)
    score = 0

    # Set up the maze walls 
    maze_walls = [  
        pygame.Rect(0,0,6,600),
        pygame.Rect(0,0,600,6),
        pygame.Rect(0,600,606,6),
        pygame.Rect(600,0,6,606),
        pygame.Rect(300,0,6,66),
        pygame.Rect(60,60,186,6),
        pygame.Rect(360,60,186,6),
        pygame.Rect(60,120,66,6),
        pygame.Rect(60,120,6,126),
        pygame.Rect(180,120,246,6),
        pygame.Rect(300,120,6,66),
        pygame.Rect(480,120,66,6),
        pygame.Rect(540,120,6,126),
        pygame.Rect(120,180,126,6),
        pygame.Rect(120,180,6,126),
        pygame.Rect(360,180,126,6),
        pygame.Rect(480,180,6,126),
        pygame.Rect(180,240,6,126),
        pygame.Rect(180,360,246,6),
        pygame.Rect(420,240,6,126),
        pygame.Rect(240,240,42,6),
        pygame.Rect(324,240,42,6),
        pygame.Rect(240,240,6,66),
        pygame.Rect(240,300,126,6),
        pygame.Rect(360,240,6,66),
        pygame.Rect(0,300,66,6),
        pygame.Rect(540,300,66,6),
        pygame.Rect(60,360,66,6),
        pygame.Rect(60,360,6,186),
        pygame.Rect(480,360,66,6),
        pygame.Rect(480,180,6,126),
        pygame.Rect(540,360,6,186),
        pygame.Rect(120,420,366,6),
        pygame.Rect(120,420,6,66),
        pygame.Rect(480,420,6,66),
        pygame.Rect(180,480,246,6),
        pygame.Rect(300,480,6,66),
        pygame.Rect(120,540,126,6),
        pygame.Rect(360,540,126,6),      
    ]
    
    #Creating three ghosts of different colors and adding them to ghost group
    ghost_group = pygame.sprite.Group()
    red_ghost = Ghost(100, 150, (255, 0, 0))
    ghost_group.add(red_ghost)

    blue_ghost = Ghost(150, 100, (0, 0, 255))
    ghost_group.add(blue_ghost)

    green_ghost = Ghost(250, 150, (0, 255, 0))
    ghost_group.add(green_ghost)

    pink_ghost = Ghost(450, 250, (255, 0, 255))
    ghost_group.add(pink_ghost)

    # Initialize cookie size and the gap between them
    co_size = 18
    co_gap = 30

    # Calculate the number of cookies that can fit on the screen horizontally and vertically
    num_cook_x = (screen_width - co_gap) // co_gap
    num_cook_y = (screen_height - co_gap) // co_gap

    # Calculate the starting positions for the cookies
    start_x = co_gap // 2
    start_y = co_gap // 2

    # Create the cookies
    cookie_group = pygame.sprite.Group()
    for i in range(num_cook_x):
        for j in range(num_cook_y):
            cook_x = start_x + i * co_gap + co_gap//2
            cook_y = start_y + j * co_gap + co_gap//2
            
            # Check if there is a wall at the cookies postition
            cook_rect = pygame.Rect(cook_x, cook_y, co_size, co_size)
            if not any(wall_rect.colliderect(cook_rect) for wall_rect in maze_walls):
                # Creates new cookie and adds it to the cookie group
                cookies = Cookie(cook_x, cook_y)
                cookie_group.add(cookies)

    running = True
    while running:
        if running == False:
            break
        # Using a for loop for the events in our game
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
                sys.exit()
            # Keyboard Input
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP:
                    pacman_direction = [0, -1]
                elif event.key == pygame.K_DOWN:
                    pacman_direction = [0, 1]
                elif event.key == pygame.K_LEFT:
                    pacman_direction = [-1, 0]
                elif event.key == pygame.K_RIGHT:
                    pacman_direction = [1, 0]

        # Move Pac-Man
        pacman_position[0] += pacman_direction[0] * 5
        pacman_position[1] += pacman_direction[1] * 5

        # Check for pacman colliding with the maze walls
        pacman_rect = pygame.Rect(pacman_position[0] - pacman_radius, pacman_position[1] - pacman_radius, pacman_radius * 2, pacman_radius * 2)
        for wall in maze_walls:
            if pacman_rect.colliderect(wall):
                pacman_position[0] -= pacman_direction[0] * 5
                pacman_position[1] -= pacman_direction[1] * 5
                break
        # Checking if pacman collides with cookies then removing them and adding to score
        for cookies in cookie_group:
            if pacman_rect.colliderect(cookies.rect):
                cookie_group.remove(cookies)
                score += 1
            # Once all cookies are "eaten" game over
            if len(cookie_group) == 0:
                print("You won!")
                you_won()
                

        # Checking for collisions with walls and pacman
        for ghost in ghost_group:
            ghost_group.update(maze_walls, pacman_rect)

    
        # Black background
        screen.fill(black)

        # Draw the maze walls
        for wall in maze_walls:
            pygame.draw.rect(screen, blue, wall)

        #Draw ghosts and cookies on screen
        ghost_group.draw(screen)
        cookie_group.draw(screen) 

        # Draw Pac-Man
        pygame.draw.circle(screen, yellow, pacman_position, pacman_radius)

        #Add the score 
        score_text = score_font.render("Score: {}".format(score), True, (255, 255, 255))
        screen.blit(score_text, (10, 10))


        # Update the screen
        pygame.display.update()

        # Tick the clock
        clock.tick(60)

    # Quit Pygame
    pygame.quit()

#Shows game over on screen and press key to play again
def show_game_over_screen():
    font = pygame.font.SysFont(None, 100)
    text = font.render("GAME OVER", True, (255, 0, 0))
    screen.blit(text, (screen_width/2 - text.get_width()/2, screen_height/2 - text.get_height()/2))
    font2 = pygame.font.SysFont(None, 48)
    text = font2.render("Press any key to play again", True, (255, 255, 255))
    screen.blit(text, (screen_width/2 - text.get_width()/2, screen_height/1.5 - text.get_height()/2))
    pygame.display.flip()

    waiting = True
    while waiting:
         for event in pygame.event.get():
            if event.type == pygame.QUIT:
                waiting = False
                running = False
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                menu()

# Shows You won! on screen and Press key to play again
def you_won():
    font = pygame.font.SysFont(None, 100)
    text = font.render("YOU WON!", True, (255, 255, 0))
    screen.blit(text, (screen_width/2 - text.get_width()/2, screen_height/2 - text.get_height()/2))
    font2 = pygame.font.SysFont(None, 48)
    text = font2.render("Press any key to play again", True, (255, 255, 255))
    screen.blit(text, (screen_width/2 - text.get_width()/2, screen_height/1.5 - text.get_height()/4))
    pygame.display.flip()

    waiting = True
    while waiting:
         for event in pygame.event.get():
            if event.type == pygame.QUIT:
                waiting = False
                running = False
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                menu()  
# Starts the game by calling run_game
def start_game():
    run_game()

# Quit game
def quit_game():
    pygame.quit()
    sys.exit()


# Set up the screen dimensions
screen_width = 606
screen_height = 606
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption("Pygame Pacman")

# Set up the game clock
clock = pygame.time.Clock()

# Set up the colors
white = (255, 255, 255)
black = (0, 0, 0)
red = (255, 0, 0)
blue = (0, 0, 255)
yellow = (255,255,0)


# Variable to hold the the buttons
gui_manager = pygame_gui.UIManager((screen_width, screen_height))


# Create the "Start Game" button
start_button = pygame_gui.elements.UIButton(
    relative_rect=pygame.Rect((screen_width // 2 - 75, screen_height // 2 - 80), (150, 50)),
    text="Start Game",
    manager=gui_manager
)


# Create the "Start Game" button
lvl_button = pygame_gui.elements.UIButton(
    relative_rect=pygame.Rect((screen_width // 2 - 75, screen_height // 2 - 15 ), (150, 50)),
    text="Start 2nd Level",
    manager=gui_manager
)

# Create the "Quit" button
quit_button = pygame_gui.elements.UIButton(
    relative_rect=pygame.Rect((screen_width // 2 - 75, screen_height // 2 + 50), (150, 50)),
    text="Quit",
    manager=gui_manager
)

# Menu loop
def menu():
    is_running = True
    while is_running:
        time_delta = clock.tick(60) / 1000.0

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                is_running = False
            elif event.type == pygame.USEREVENT:
                if event.user_type == pygame_gui.UI_BUTTON_PRESSED:
                    if event.ui_element == start_button:
                        start_game()
                    elif event.ui_element == lvl_button:
                        next_lvl()
                    elif event.ui_element == quit_button:
                        is_running = False

            gui_manager.process_events(event)

        gui_manager.update(time_delta)

        screen.fill((0, 0, 0))
        gui_manager.draw_ui(screen)

        pygame.display.update()

    pygame.quit()

menu()
