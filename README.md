import pygame;
import random;

# Initialize Pygame
pygame.init()

# Screen dimensions
SCREEN_WIDTH = 2000
SCREEN_HEIGHT = 1000

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)

# Player settings
PLAYER_WIDTH = 50
PLAYER_HEIGHT = 50
PLAYER_SPEED = 5

# Bullet settings
BULLET_WIDTH = 5
BULLET_HEIGHT = 10
BULLET_SPEED = 7

# Target settings
TARGET_WIDTH = 50
TARGET_HEIGHT = 50
TARGET_SPEED = 3

# Balloon settings
BALLOON_WIDTH = 40
BALLOON_HEIGHT = 60
BALLOON_SPEED = 2

# Create the screen
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Shooting Game")

# Load images
player_image = pygame.Surface((PLAYER_WIDTH, PLAYER_HEIGHT))
player_image.fill(WHITE)
bullet_image = pygame.Surface((BULLET_WIDTH, BULLET_HEIGHT))
bullet_image.fill(RED)
target_image = pygame.Surface((TARGET_WIDTH, TARGET_HEIGHT))
target_image.fill(GREEN)
balloon_image = pygame.Surface((BALLOON_WIDTH, BALLOON_HEIGHT))
balloon_image.fill(BLUE)

# Font for score
font = pygame.font.Font(None, 36)

# Player class
class Player(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = player_image
        self.rect = self.image.get_rect()
        self.rect.centerx = SCREEN_WIDTH // 2
        self.rect.bottom = SCREEN_HEIGHT - 10
        self.speed_x = 0

    def update(self):
        self.speed_x = 0
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            self.speed_x = -PLAYER_SPEED
        if keys[pygame.K_RIGHT]:
            self.speed_x = PLAYER_SPEED
        self.rect.x += self.speed_x
        if self.rect.left < 0:
            self.rect.left = 0
        if self.rect.right > SCREEN_WIDTH:
            self.rect.right = SCREEN_WIDTH

    def shoot(self):
        bullet = Bullet(self.rect.centerx, self.rect.top)
        all_sprites.add(bullet)
        bullets.add(bullet)

# Bullet class
class Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = bullet_image
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speed_y = -BULLET_SPEED

    def update(self):
        self.rect.y += self.speed_y
        if self.rect.bottom < 0:
            self.kill()

# Target class
class Target(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = target_image
        self.rect = self.image.get_rect()
        self.rect.x = random.randint(0, SCREEN_WIDTH - TARGET_WIDTH)
        self.rect.y = random.randint(-100, -40)
        self.speed_y = TARGET_SPEED

    def update(self):
        self.rect.y += self.speed_y
        if self.rect.top > SCREEN_HEIGHT:
            self.rect.x = random.randint(0, SCREEN_WIDTH - TARGET_WIDTH)
            self.rect.y = random.randint(-100, -40)

# Balloon class
class Balloon(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = balloon_image
        self.rect = self.image.get_rect()
        self.rect.x = random.randint(0, SCREEN_WIDTH - BALLOON_WIDTH)
        self.rect.y = random.randint(-100, -40)
        self.speed_y = BALLOON_SPEED

    def update(self):
        self.rect.y += self.speed_y
        if self.rect.top > SCREEN_HEIGHT:
            self.rect.x = random.randint(0, SCREEN_WIDTH - BALLOON_WIDTH)
            self.rect.y = random.randint(-100, -40)

# Create sprite groups
all_sprites = pygame.sprite.Group()
bullets = pygame.sprite.Group()
targets = pygame.sprite.Group()
balloons = pygame.sprite.Group()

# Create player
player = Player()
all_sprites.add(player)

# Create targets
for _ in range(5):
    target = Target()
    all_sprites.add(target)
    targets.add(target)

# Create balloons
for _ in range(3):
    balloon = Balloon()
    all_sprites.add(balloon)
    balloons.add(balloon)

# Initialize score
score = 0

# Game loop
running = True
clock = pygame.time.Clock()

while running:
    # Handle events
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                player.shoot()

    # Update
    all_sprites.update()

    # Check for collisions
    hits = pygame.sprite.groupcollide(targets, bullets, True, True)
    for hit in hits:
        score += 10  # Increment score for hitting a target
        target = Target()
        all_sprites.add(target)
        targets.add(target)

    balloon_hits = pygame.sprite.groupcollide(balloons, bullets, True, True)
    for hit in balloon_hits:
        score += 20  # Increment score for hitting a balloon
        balloon = Balloon()
        all_sprites.add(balloon)
        balloons.add(balloon)

    # Draw / render
    screen.fill(BLACK)
    all_sprites.draw(screen)

    # Draw the score
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (10, 10))

    # Flip the display
    pygame.display.flip()

    # Cap the frame rate
    clock.tick(60)

pygame.quit()

