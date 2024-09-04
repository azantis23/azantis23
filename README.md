import pygame
import random

# Initialize Pygame
pygame.init()

# Constants
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
FPS = 60

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 0)
RED = (255, 0, 0)

# Set up the display
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Block Battle")

clock = pygame.time.Clock()

# Player class
class Player(pygame.sprite.Sprite):
    def __init__(self):
        super(Player, self).__init__()
        self.image = pygame.Surface((50, 50))
        self.image.fill(GREEN)
        self.rect = self.image.get_rect()
        self.rect.center = (SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2)
        self.speed = 5
        self.health = 100
    
    def update(self, keys):
        if keys[pygame.K_LEFT] or keys[pygame.K_a]:
            self.rect.x -= self.speed
        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            self.rect.x += self.speed
        if keys[pygame.K_UP] or keys[pygame.K_w]:
            self.rect.y -= self.speed
        if keys[pygame.K_DOWN] or keys[pygame.K_s]:
            self.rect.y += self.speed
        
        # Boundary check
        if self.rect.left < 0:
            self.rect.left = 0
        if self.rect.right > SCREEN_WIDTH:
            self.rect.right = SCREEN_WIDTH
        if self.rect.top < 0:
            self.rect.top = 0
        if self.rect.bottom > SCREEN_HEIGHT:
            self.rect.bottom = SCREEN_HEIGHT

# Enemy class
class Enemy(pygame.sprite.Sprite):
    def __init__(self):
        super(Enemy, self).__init__()
        self.image = pygame.Surface((40, 40))
        self.image.fill(RED)
        self.rect = self.image.get_rect()
        self.rect.x = random.randint(0, SCREEN_WIDTH - self.rect.width)
        self.rect.y = random.randint(0, SCREEN_HEIGHT - self.rect.height)
        self.speed = random.randint(1, 4)
    
    def update(self):
        self.rect.y += self.speed
        if self.rect.top > SCREEN_HEIGHT:
            self.rect.bottom = 0
            self.rect.x = random.randint(0, SCREEN_WIDTH - self.rect.width)

# Bullet class
class Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super(Bullet, self).__init__()
        self.image = pygame.Surface((10, 10))
        self.image.fill(BLACK)
        self.rect = self.image.get_rect()
        self.rect.center = (x, y)
        self.speed = -10
    
    def update(self):
        self.rect.y += self.speed
        if self.rect.bottom < 0:
            self.kill()

# Initialize player, enemy groups
player = Player()
enemies = pygame.sprite.Group()
bullets = pygame.sprite.Group()

all_sprites = pygame.sprite.Group()
all_sprites.add(player)

for _ in range(10):
    enemy = Enemy()
    enemies.add(enemy)
    all_sprites.add(enemy)

# Game loop
running = True
while running:
    clock.tick(FPS)
    
    # Event handling
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                bullet = Bullet(player.rect.centerx, player.rect.top)
                bullets.add(bullet)
                all_sprites.add(bullet)
    
    # Update
    keys = pygame.key.get_pressed()
    player.update(keys)
    enemies.update()
    bullets.update()
    
    # Collision detection
    hits = pygame.sprite.groupcollide(enemies, bullets, True, True)
    for hit in hits:
        enemy = Enemy()
        enemies.add(enemy)
        all_sprites.add(enemy)
    
    if pygame.sprite.spritecollideany(player, enemies):
        player.health -= 1
        if player.health <= 0:
            running = False
    
    # Drawing
    screen.fill(WHITE)
    all_sprites.draw(screen)
    
    # Display health
    font = pygame.font.SysFont(None, 36)
    health_text = font.render(f'Health: {player.health}', True, BLACK)
    screen.blit(health_text, (10, 10))
    
    pygame.display.flip()

pygame.quit()
