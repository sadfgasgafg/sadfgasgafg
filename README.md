import pygame
import random
import sys

# Game settings
pygame.init()
WIDTH, HEIGHT = 400, 600
BLOCK_SIZE = 40
FPS = 30
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Blastify Deluxe Adventure")
clock = pygame.time.Clock()

# Colors and other settings
COLORS = [(255, 0, 0), (0, 255, 0), (0, 0, 255), (255, 255, 0)]
coins = 0
explosion_power = False
explosion_cost = 10
current_level = 1
target_clearance = 30  # Number of blocks to clear to complete a level

# Board creation
def create_board():
    return [[(random.choice(COLORS), pygame.Rect(x, y, BLOCK_SIZE, BLOCK_SIZE)) for x in range(0, WIDTH, BLOCK_SIZE)]
            for y in range(0, HEIGHT, BLOCK_SIZE)]

# Explode blocks to clear board
def explode_blocks(x, y, board):
    global coins
    target_color = board[y][x][0]
    to_explode = [(x, y)]
    exploded = set()
    cleared_blocks = 0

    while to_explode:
        cx, cy = to_explode.pop()
        if (cx, cy) in exploded:
            continue
        exploded.add((cx, cy))
        for nx, ny in [(cx - 1, cy), (cx + 1, cy), (cx, cy - 1), (cx, cy + 1)]:
            if 0 <= nx < len(board[0]) and 0 <= ny < len(board):
                if board[ny][nx][0] == target_color:
                    to_explode.append((nx, ny))

    if len(exploded) > 1:
        cleared_blocks += len(exploded)  # Track blocks cleared
        for ex, ey in exploded:
            board[ey][ex] = (None, board[ey][ex][1])
    return cleared_blocks

# Update board after explosion
def update_board(board):
    for x in range(len(board[0])):
        empty_slots = []
        for y in range(len(board) - 1, -1, -1):
            color, rect = board[y][x]
            if color is None:
                empty_slots.append(y)
            elif empty_slots:
                empty_y = empty_slots.pop(0)
                board[empty_y][x] = (color, board[empty_y][x][1])
                board[y][x] = (None, rect)
                empty_slots.append(y)

# Purchase explosion power if player has enough coins
def purchase_explosion():
    global explosion_power, coins
    if coins >= explosion_cost:
        explosion_power = True
        coins -= explosion_cost
        print("Explosion power purchased!")
    else:
        print("Not enough coins to buy explosion power!")

# Display the shop for purchasing abilities
def display_shop():
    screen.fill((50, 50, 50))
    font = pygame.font.Font(None, 36)
    shop_texts = [
        f"Coins: {coins}",
        f"Explosion Power Cost: {explosion_cost} coins",
        "Press 'E' to purchase Explosion Power",
        "Press 'R' to return to the game"
    ]
    for i, text in enumerate(shop_texts):
        txt_surface = font.render(text, True, (255, 255, 255))
        screen.blit(txt_surface, (50, 100 + i * 40))
    pygame.display.flip()

# Adventure mode gameplay
def adventure_mode():
    global coins, explosion_power, current_level, target_clearance
    board = create_board()
    cleared_blocks = 0
    running = True
    
    while running:
        screen.fill((0, 0, 0))
        
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_s:
                    display_shop()
                elif event.key == pygame.K_e:
                    purchase_explosion()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                mx, my = pygame.mouse.get_pos()
                bx, by = mx // BLOCK_SIZE, my // BLOCK_SIZE
                if 0 <= bx < len(board[0]) and 0 <= by < len(board):
                    blocks_cleared = explode_blocks(bx, by, board)
                    cleared_blocks += blocks_cleared
                    if explosion_power and blocks_cleared > 0:
                        coins += blocks_cleared * 2  # Double coins with explosion power
                    else:
                        coins += blocks_cleared
                    update_board(board)
                    if cleared_blocks >= target_clearance:
                        coins += 10  # Level clear bonus
                        current_level += 1
                        cleared_blocks = 0
                        board = create_board()
                        target_clearance += 10  # Increase difficulty

        # Draw the blocks
        for row in board:
            for color, rect in row:
                if color:
                    pygame.draw.rect(screen, color, rect)
                    pygame.draw.rect(screen, (0, 0, 0), rect, 2)

        # Display information
        font = pygame.font.Font(None, 36)
        coins_text = font.render(f"Coins: {coins}", True, (255, 255, 255))
        level_text = font.render(f"Level: {current_level}", True, (255, 255, 255))
        target_text = font.render(f"Target: {target_clearance}", True, (255, 255, 255))
        screen.blit(coins_text, (10, 10))
        screen.blit(level_text, (10, 50))
        screen.blit(target_text, (10, 90))
        
        pygame.display.flip()
        clock.tick(FPS)

# Main menu with game options
def game_menu():
    global coins
    running = True
    while running:
        screen.fill((50, 50, 50))
        font = pygame.font.Font(None, 36)
        menu_texts = [
            "1. Start Adventure Mode",
            "2. Enter Shop (Press S in game)",
            "Press Q to Quit"
        ]
        
        for i, text in enumerate(menu_texts):
            txt_surface = font.render(text, True, (255, 255, 255))
            screen.blit(txt_surface, (50, 100 + i * 40))
        
        pygame.display.flip()
        
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                return
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_q:
                    running = False
                elif event.key == pygame.K_1:
                    adventure_mode()  # Start Adventure Mode

# Run main menu
game_menu()
