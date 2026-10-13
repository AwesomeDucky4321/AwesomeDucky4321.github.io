# Code Autopsy: Flappy Webby

Python + pygame. Full code: [flappy_webby.py](flappy_webby.py)

## 1. The canvas

```python
WIDTH = 400
HEIGHT = 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
```

- Size: 400 × 600 pixels
- Changing it: the window resizes, but the ground (`GROUND_TOP = 530`), sun, and hills stay at fixed positions. Below 530 tall, the ground gets cut off.

## 2. The state variables

```python
bird_y = HEIGHT // 2
bird_speed = 0
pipes = []
score = 0
best_score = 0
game_state = "ready"
```

- Position: `bird_y`
- Speed: `bird_speed`
- Score: `score`, `best_score`
- Game over: `game_state = "over"`
- Declared under `# ---------- Game state ----------`, before the game loop

## 3. The game loop

```python
while running:
    ...
    pygame.display.flip()
    clock.tick(60)
```

- Runs 60 times per second
- Each frame: input → movement → collisions/scoring → drawing

## 4. Input handling

```python
if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
    if game_state == "playing":
        bird_speed = FLAP_STRENGTH
```

- Controls: spacebar and mouse click
- Changes `bird_speed` to `-8` (flap)
- Clicking the button starts/restarts the game

## 5. Collision / win-lose logic

```python
if bird_rect.colliderect(top_rect) or bird_rect.colliderect(bottom_rect):
    game_state = "over"

if bird_y + BIRD_RADIUS > GROUND_TOP:
    game_state = "over"
```

- Hitting a pipe or the ground = game over
- Ceiling stops the bird but doesn't end the game
- No win condition

## 6. Scoring

```python
if pipe["scored"] == False and pipe["x"] + PIPE_WIDTH < BIRD_X:
    pipe["scored"] = True
    score = score + 1
```

- Score lives in `score`
- +1 when a pipe passes the bird
