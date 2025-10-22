# AI_-PROJECT-

PROGRAM: 

import pygame
import copy
import random
import sys
import time

# --- Constants ---
TILE_SIZE = 100
GRID_SIZE = 3
WINDOW_SIZE = TILE_SIZE * GRID_SIZE
FPS = 30
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (200, 0, 0)
GRAY = (200, 200, 200)

# --- IDS Solver ---
goal = [[1, 2, 3], [4, 5, 6], [7, 8, 0]]
moves = [(-1,0),(1,0),(0,-1),(0,1)]

def find_zero(state):
    for i in range(3):
        for j in range(3):
            if state[i][j] == 0:
                return i,j
    return None

def is_goal(state):
    return state == goal

def move_blank(state, direction):
    x, y = find_zero(state)
    dx, dy = direction
    nx, ny = x+dx, y+dy
    if 0 <= nx < 3 and 0 <= ny < 3:
        new_state = copy.deepcopy(state)
        new_state[x][y], new_state[nx][ny] = new_state[nx][ny], new_state[x][y]
        return new_state
    return None

def DLS(state, depth, path):
    if is_goal(state):
        return path
    if depth == 0:
        return None
    for move in moves:
        new_state = move_blank(state, move)
        if new_state and new_state not in path:
            result = DLS(new_state, depth-1, path + [new_state])
            if result:
                return result
    return None

def IDS(start):
    depth = 0
    while True:
        result = DLS(start, depth, [start])
        if result:
            return result
        depth += 1

# --- Shuffle Puzzle ---
def shuffled_puzzle():
    nums = list(range(9))
    while True:
        random.shuffle(nums)
        puzzle = [nums[:3], nums[3:6], nums[6:]]
        if puzzle != goal:
            return puzzle

# --- Draw Puzzle ---
def draw_puzzle(screen, state):
    screen.fill(WHITE)
    for i in range(GRID_SIZE):
        for j in range(GRID_SIZE):
            x, y = j*TILE_SIZE, i*TILE_SIZE
            num = state[i][j]
            rect = pygame.Rect(x, y, TILE_SIZE, TILE_SIZE)
            pygame.draw.rect(screen, GRAY if num==0 else RED, rect)
            pygame.draw.rect(screen, BLACK, rect, 3)
            if num != 0:
                font = pygame.font.SysFont(None, 72)
                text = font.render(str(num), True, BLACK)
                text_rect = text.get_rect(center=rect.center)
                screen.blit(text, text_rect)

# --- Main Loop ---
def main():
    pygame.init()
    screen = pygame.display.set_mode((WINDOW_SIZE, WINDOW_SIZE))
    pygame.display.set_caption("8-Puzzle Game")
    clock = pygame.time.Clock()

    puzzle = shuffled_puzzle()
    running = True

    while running:
        draw_puzzle(screen, puzzle)
        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

            # --- Keyboard Control ---
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP:
                    puzzle = move_blank(puzzle, (1,0)) or puzzle
                elif event.key == pygame.K_DOWN:
                    puzzle = move_blank(puzzle, (-1,0)) or puzzle
                elif event.key == pygame.K_LEFT:
                    puzzle = move_blank(puzzle, (0,1)) or puzzle
                elif event.key == pygame.K_RIGHT:
                    puzzle = move_blank(puzzle, (0,-1)) or puzzle
                elif event.key == pygame.K_e:
                    # Auto-solve using IDS
                    solution_path = IDS(puzzle)
                    for step in solution_path[1:]:
                        puzzle = step
                        draw_puzzle(screen, puzzle)
                        pygame.display.flip()
                        pygame.time.wait(300)

            # --- Mouse Control ---
            elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                mx, my = event.pos
                i, j = my // TILE_SIZE, mx // TILE_SIZE
                zero_i, zero_j = find_zero(puzzle)
                if (abs(zero_i - i) + abs(zero_j - j)) == 1:
                    puzzle[zero_i][zero_j], puzzle[i][j] = puzzle[i][j], puzzle[zero_i][zero_j]

        # Check goal
        if is_goal(puzzle):
            draw_puzzle(screen, puzzle)
            pygame.display.flip()
            pygame.time.wait(1000)
            print("Puzzle Solved!")
            running = False

        clock.tick(FPS)

    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    main()
