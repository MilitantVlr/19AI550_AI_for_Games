# Ex.No: 11  Design of Car Game using Heuristic Distance-Based  AI for Proximity 
### DATE: 25/10/2024                                                                           
### REGISTER NUMBER : 212221040145
### AIM: 
To write a Python program to simulate the car racing game using Pygame.
### Algorithm:
1.Initialize the Game:
  Import necessary libraries such as pygame and initialize the game environment.
  Set up the screen dimensions, player car, AI car, and background.
  
2.Define Car Movement Logic:
  Create movement controls for the player’s car using key press events (LEFT and RIGHT arrows).
  Implement AI movement logic with obstacle detection to dodge bombs.
  
3.Add Obstacles (Bombs):
  Randomly generate bombs at the top of the screen that fall downwards.
  Ensure they have random speeds and positions.
  
4.Collision Detection:
  Implement collision detection between cars (both player and AI) and the bombs.
  
5.Score Calculation:
  Update scores based on proximity to obstacles, with points awarded to both the player and AI for dodging.

6.Game Over Logic:
  When the player or AI collides with an obstacle, display a game-over screen with the option to retry or quit.
  
7.End the Game:
  If the player chooses to quit, close the game; otherwise, reset the game for another round.
  
### Program:

```
import pygame
import random
import math

# Initialize Pygame
pygame.init()

# Screen settings
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))

pygame.display.set_caption("AI Car Racing Game")

# Game settings
player_car_width, player_car_height = 100, 100  # Set player car size
ai_car_width, ai_car_height = 100, 100           # Set AI car size
player_x, player_y = WIDTH // 2 - player_car_width // 2, HEIGHT - player_car_height - 20
ai_x, ai_y = WIDTH // 2 + ai_car_width // 2, HEIGHT - ai_car_height - 20
car_speed = 5
dodge_speed = 15
center_x = WIDTH // 2 - player_car_width // 2
dodging = False

# Obstacle settings
obstacle_radius = 30  # Set the radius for bombs
obstacle_min_speed = 5
obstacle_max_speed = 10
obstacle_frequency = 1500
last_obstacle_time = pygame.time.get_ticks()
obstacles = []

# Load images
player_car_img = pygame.image.load('carme.png')
player_car_img = pygame.transform.scale(player_car_img, (player_car_width, player_car_height))

ai_car_img = pygame.image.load('carAI.png')
ai_car_img = pygame.transform.scale(ai_car_img, (ai_car_width, ai_car_height))

bomb_img = pygame.image.load('bomb.png')
bomb_img = pygame.transform.scale(bomb_img, (obstacle_radius * 2, obstacle_radius * 2))

# Load and scale background image
background_img = pygame.image.load('background.jfif')
background_img = pygame.transform.scale(background_img, (WIDTH, HEIGHT))  # Scale to cover the entire background

# Background position
background_y1 = 0
background_y2 = -HEIGHT  # Start the second image just above the screen

# Scores
player_score = 0
ai_score = 0
font = pygame.font.Font(None, 36)  # Font for displaying scores

# AI movement logic
def ai_move(ai_x, ai_y, obstacles):
    global dodging, ai_score
    closest_obstacle = None
    min_distance = HEIGHT
    dodged = False  # Track if AI has dodged

    for obstacle in obstacles:
        obstacle_x, obstacle_y, _, _, _ = obstacle
        distance = ai_y - obstacle_y
        if distance > 0 and distance < min_distance:
            min_distance = distance
            closest_obstacle = obstacle

    if closest_obstacle:
        obstacle_x, obstacle_y, _, _, _ = closest_obstacle

        # If an obstacle is close, dodge left or right
        if min_distance < HEIGHT // 3:
            if obstacle_x < ai_x + ai_car_width and obstacle_x + obstacle_radius * 2 > ai_x:
                dodging = True
                dodged = True  # Set dodged to True when dodging
                if ai_x > car_speed:  # Move left
                    ai_x -= dodge_speed
                elif ai_x + ai_car_width + car_speed < WIDTH:  # Move right
                    ai_x += dodge_speed
        else:
            # Center the AI car if no immediate obstacles
            if not dodging:
                if ai_x < center_x:
                    ai_x += car_speed
                elif ai_x > center_x:
                    ai_x -= car_speed
    else:
        if dodging:
            if all(obstacle[1] > ai_y + ai_car_height for obstacle in obstacles):
                dodging = False

        if not dodging:
            if ai_x < center_x:
                ai_x += car_speed
            elif ai_x > center_x:
                ai_x -= car_speed

    if dodged:  # If AI dodged an obstacle, add 10 points
        ai_score += 10

    return ai_x

# Check collision between circles
def circle_collision(x1, y1, r1, x2, y2, r2):
    distance = math.sqrt((x1 - x2) ** 2 + (y1 - y2) ** 2)
    return distance < (r1 + r2)

# Update scores based on proximity to obstacles
def update_score(center_x, center_y):
    global player_score, ai_score
    for obstacle in obstacles:
        obstacle_center = (obstacle[0] + obstacle_radius, obstacle[1] + obstacle_radius)
        distance = math.sqrt((center_x - obstacle_center[0]) ** 2 + (center_y - obstacle_center[1]) ** 2)
        if distance < 100:  # Change this value to adjust the scoring distance
            if center_y == player_y + player_car_height // 2:  # Player's center
                player_score += 1  # Increment player score
            elif center_y == ai_y + ai_car_height // 2:  # AI's center
                ai_score += 1  # Increment AI score
            break  # Only score once per frame

# Show retry and quit options
def show_game_over_screen():
    screen.fill((0, 0, 0))  # Fill the screen with black
    font = pygame.font.Font(None, 74)
    game_over_text = font.render("Game Over", True, (255, 255, 255))
    retry_text = font.render("Press R to Retry", True, (255, 255, 255))
    quit_text = font.render("Press Q to Quit", True, (255, 255, 255))

    screen.blit(game_over_text, (WIDTH // 2 - game_over_text.get_width() // 2, HEIGHT // 3))
    screen.blit(retry_text, (WIDTH // 2 - retry_text.get_width() // 2, HEIGHT // 2))
    screen.blit(quit_text, (WIDTH // 2 - quit_text.get_width() // 2, HEIGHT // 2 + 50))

    pygame.display.update()

    waiting = True
    while waiting:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r:  # Retry the game
                    waiting = False
                    reset_game()
                elif event.key == pygame.K_q:  # Quit the game
                    pygame.quit()
                    exit()

# Reset game state
def reset_game():
    global player_x, player_y, ai_x, ai_y, obstacles, player_score, ai_score, last_obstacle_time
    player_x, player_y = WIDTH // 2 - player_car_width // 2, HEIGHT - player_car_height - 20
    ai_x, ai_y = WIDTH // 2 + ai_car_width // 2, HEIGHT - ai_car_height - 20
    obstacles = []
    player_score = 0
    ai_score = 0
    last_obstacle_time = pygame.time.get_ticks()

# Game loop settings
clock = pygame.time.Clock()
FPS = 60
running = True

# Main loop
while running:
    # Draw the scrolling background
    screen.blit(background_img, (0, background_y1))  # Draw the first background image
    screen.blit(background_img, (0, background_y2))  # Draw the second background image

    # Update background positions
    background_y1 += 5  # Scroll down
    background_y2 += 5  # Scroll down

    # Reset positions if they go off-screen
    if background_y1 >= HEIGHT:
        background_y1 = -HEIGHT
    if background_y2 >= HEIGHT:
        background_y2 = -HEIGHT

    # Event handling
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
    
    # Key presses for player's car movement
    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT] and player_x > 0:
        player_x -= car_speed
    if keys[pygame.K_RIGHT] and player_x < WIDTH - player_car_width:
        player_x += car_speed

    # Generate obstacles at intervals
    current_time = pygame.time.get_ticks()
    if current_time - last_obstacle_time > obstacle_frequency:
        obstacle_x = random.randint(0 + obstacle_radius, WIDTH - obstacle_radius * 2)  # Ensure obstacles fit within screen
        obstacle_speed = random.randint(obstacle_min_speed, obstacle_max_speed)
        obstacles.append([obstacle_x, -obstacle_radius * 2, obstacle_radius, obstacle_radius * 2, obstacle_speed])
        last_obstacle_time = current_time

    # Move and draw obstacles
    for obstacle in obstacles:
        obstacle[1] += obstacle[4]  # Each obstacle has a random speed
        screen.blit(bomb_img, (obstacle[0], obstacle[1]))  # Draw bomb image
    
    # Remove off-screen obstacles
    obstacles = [ob for ob in obstacles if ob[1] < HEIGHT]

    # AI movement logic
    ai_x = ai_move(ai_x, ai_y, obstacles)
    
    # Draw player and AI cars
    screen.blit(player_car_img, (player_x, player_y))  # Draw player car
    screen.blit(ai_car_img, (ai_x, ai_y))  # Draw AI car

    # Define circle properties for collision detection
    player_radius = player_car_width // 2 * 0.5  # Smaller radius for the player car (reduced)
    ai_radius = ai_car_width // 2 * 0.5  # Smaller radius for the AI car (reduced)
    player_center = (player_x + player_car_width // 2, player_y + player_car_height // 2)
    ai_center = (ai_x + ai_car_width // 2, ai_y + ai_car_height // 2)

    # Collision detection (player)
    for obstacle in obstacles:
        obstacle_center = (obstacle[0] + obstacle_radius, obstacle[1] + obstacle_radius)  # Center of bomb
        if circle_collision(player_center[0], player_center[1], player_radius, obstacle_center[0], obstacle_center[1], obstacle_radius):
            print("Game Over!")
            show_game_over_screen()

    # Collision detection (AI)
    for obstacle in obstacles:
        obstacle_center = (obstacle[0] + obstacle_radius, obstacle[1] + obstacle_radius)  # Center of bomb
        if circle_collision(ai_center[0], ai_center[1], ai_radius, obstacle_center[0], obstacle_center[1], obstacle_radius):
            print("AI crashed!")
            show_game_over_screen()

    # Update scores for player and AI based on proximity to obstacles
    update_score(player_center[0], player_center[1])
    update_score(ai_center[0], ai_center[1])

    # Render scores on the screen
    score_text = font.render(f"Player Score: {player_score}  AI Score: {ai_score}", True, (0, 0, 0))
    screen.blit(score_text, (20, 20))  # Draw scores at the top-left corner

    # Update the display
    pygame.display.update()
    clock.tick(FPS)

pygame.quit()
```










### Output:
![mygame](https://github.com/user-attachments/assets/22cfb5eb-8ef0-47e6-b063-c3143a4a6731)

![restart](https://github.com/user-attachments/assets/56eb1fed-5b92-4470-bb8c-16ca4d74a4cd)


### Result:
Thus, the simple car racing game was implemented using Pygame.
