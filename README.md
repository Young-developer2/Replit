import pygame

# 1. Setup
pygame.init()
SCREEN_WIDTH, SCREEN_HEIGHT = 800, 400
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
clock = pygame.time.Clock()

# Colors (Character Themes)
P1_COLOR = (50, 100, 255) # Blue Fighter (Like Ryu)
P2_COLOR = (255, 50, 50)  # Red Fighter (Like Ken)
GROUND_Y = 300

class Fighter:
    def __init__(self, x, color, flip):
        self.rect = pygame.Rect(x, GROUND_Y, 50, 100)
        self.color = color
        self.vel_y = 0
        self.is_jumping = False
        self.attacking = False
        self.attack_type = 0 # 0: None, 1: Punch
        self.health = 100
        self.flip = flip

    def move(self, left, right, jump, attack, target):
        SPEED = 5
        GRAVITY = 1
        dx = 0

        # Movement
        if not self.attacking:
            if left: dx = -SPEED
            if right: dx = SPEED
            if jump and not self.is_jumping:
                self.vel_y = -15
                self.is_jumping = True

        # Gravity
        self.vel_y += GRAVITY
        self.rect.y += self.vel_y
        if self.rect.bottom > GROUND_Y + 100:
            self.rect.bottom = GROUND_Y + 100
            self.is_jumping = False

        self.rect.x += dx

        # Attack Logic
        if attack and not self.attacking:
            self.attacking = True
            self.attack_type = 1
            # Check for hit
            attack_rect = pygame.Rect(self.rect.x + (50 if not self.flip else -50), self.rect.y, 50, 50)
            if attack_rect.colliderect(target.rect):
                target.health -= 10
        
    def draw(self, surface):
        pygame.draw.rect(surface, self.color, self.rect)
        if self.attacking: # Visual indicator of a punch
            atk_x = self.rect.right if not self.flip else self.rect.left - 50
            pygame.draw.rect(surface, (255, 255, 255), (atk_x, self.rect.y + 20, 50, 20))

# 2. Instances
p1 = Fighter(200, P1_COLOR, False)
p2 = Fighter(600, P2_COLOR, True)

# 3. Game Loop
run = True
while run:
    screen.fill((30, 30, 30)) # Dark Arena
    pygame.draw.rect(screen, (100, 100, 100), (0, GROUND_Y + 100, SCREEN_WIDTH, 50)) # Floor

    keys = pygame.key.get_pressed()
    
    # Reset attack state when key released
    if not keys[pygame.K_f]: p1.attacking = False
    if not keys[pygame.K_l]: p2.attacking = False

    # Player 1 (WASD + F for punch)
    p1.move(keys[pygame.K_a], keys[pygame.K_d], keys[pygame.K_w], keys[pygame.K_f], p2)
    # Player 2 (Arrows + L for punch)
    p2.move(keys[pygame.K_LEFT], keys[pygame.K_RIGHT], keys[pygame.K_UP], keys[pygame.K_l], p1)

    p1.draw(screen)
    p2.draw(screen)

    # Health Bars
    pygame.draw.rect(screen, (255, 0, 0), (50, 20, p1.health * 2, 20))
    pygame.draw.rect(screen, (255, 0, 0), (SCREEN_WIDTH - 250, 20, p2.health * 2, 20))

    for event in pygame.event.get():
        if event.type == pygame.QUIT: run = False

    pygame.display.update()
    clock.tick(60)

pygame.quit()
