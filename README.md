import pygame
import random
import sys
from typing import List, Tuple

# Инициализация Pygame
pygame.init()

# Константы
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
GRID_SIZE = 20
GRID_WIDTH = SCREEN_WIDTH // GRID_SIZE
GRID_HEIGHT = SCREEN_HEIGHT // GRID_SIZE

# Цвета
BOARD_BACKGROUND_COLOR = (0, 0, 0)
SNAKE_COLOR = (0, 255, 0)
APPLE_COLOR = (255, 0, 0)
TEXT_COLOR = (255, 255, 255)


class GameObject:
    """Базовый класс для игровых объектов."""
    
    def init(self, position: Tuple[int, int] = None):
        """
        Инициализирует игровой объект.
        
        Args:
            position: Начальная позиция объекта. Если None, генерируется случайная позиция.
        """
        if position is None:
            self.position = self.generate_random_position()
        else:
            self.position = position
    
    def generate_random_position(self) -> Tuple[int, int]:
        """Генерирует случайную позицию на игровом поле."""
        return (
            random.randint(0, GRID_WIDTH - 1) * GRID_SIZE,
            random.randint(0, GRID_HEIGHT - 1) * GRID_SIZE
        )
    
    def draw(self, surface: pygame.Surface):
        """Абстрактный метод для отрисовки объекта."""
        raise NotImplementedError("Метод draw должен быть реализован в дочернем классе")


class Apple(GameObject):
    """Класс яблока - еды для змейки."""
    
    def init(self):
        """Инициализирует яблоко со случайной позицией."""
        super().init()
    
    def draw(self, surface: pygame.Surface):
        """Отрисовывает яблоко на поверхности."""
        rect = pygame.Rect(
            (self.position[0], self.position[1]),
            (GRID_SIZE, GRID_SIZE)
        )
        pygame.draw.rect(surface, APPLE_COLOR, rect)
        pygame.draw.rect(surface, (255, 255, 255), rect, 1)


class Snake(GameObject):
    """Класс змейки."""
    
    def init(self):
        """Инициализирует змейку в начальном состоянии."""
        super().init(((SCREEN_WIDTH // 2), (SCREEN_HEIGHT // 2)))
        self.positions = [self.position]
        self.length = 1
        self.direction = random.choice([(GRID_SIZE, 0), (-GRID_SIZE, 0), (0, GRID_SIZE), (0, -GRID_SIZE)])
        self.last = None
    
    def get_head_position(self) -> Tuple[int, int]:
        """Возвращает позицию головы змейки."""
        return self.positions[0]
    
    def move(self):
        """Перемещает змейку в текущем направлении."""
        head_x, head_y = self.get_head_position()
        dx, dy = self.direction
        new_x = (head_x + dx) % SCREEN_WIDTH
        new_y = (head_y + dy) % SCREEN_HEIGHT
        new_position = (new_x, new_y)
        
        # Проверка столкновения с собой
        if new_position in self.positions[1:]:
            self.reset()
            return
        
        self.positions.insert(0, new_position)
        if len(self.positions) > self.length:
            self.last = self.positions.pop()
    
    def reset(self):
        """Сбрасывает змейку в начальное состояние."""
        self.positions = [((SCREEN_WIDTH // 2), (SCREEN_HEIGHT // 2))]
        self.length = 1
        self.direction = random.choice([(GRID_SIZE, 0), (-GRID_SIZE, 0), (0, GRID_SIZE), (0, -GRID_SIZE)])
        self.last = None
    
    def draw(self, surface: pygame.Surface):
        """Отрисовывает змейку на поверхности."""
        for position in self.positions:
            rect = pygame.Rect(
                (position[0], position[1]),
                (GRID_SIZE, GRID_SIZE)
            )
            pygame.draw.rect(surface, SNAKE_COLOR, rect)
            pygame.draw.rect(surface, (255, 255, 255), rect, 1)


def draw_score(surface: pygame.Surface, score: int):
    """Отрисовывает счет на поверхности."""
    font = pygame.font.Font(None, 36)
    text = font.render(f'Счет: {score}', True, TEXT_COLOR)
    surface.blit(text, (10, 10))

def main():
    """Основная функция игры."""
    screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
    pygame.display.set_caption('Змейка')
    clock = pygame.time.Clock()
    
    snake = Snake()
    apple = Apple()
    score = 0
    
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP and snake.direction != (0, GRID_SIZE):
                    snake.direction = (0, -GRID_SIZE)
                elif event.key == pygame.K_DOWN and snake.direction != (0, -GRID_SIZE):
                    snake.direction = (0, GRID_SIZE)
                elif event.key == pygame.K_LEFT and snake.direction != (GRID_SIZE, 0):
                    snake.direction = (-GRID_SIZE, 0)
                elif event.key == pygame.K_RIGHT and snake.direction != (-GRID_SIZE, 0):
                    snake.direction = (GRID_SIZE, 0)
        
        snake.move()
        
        # Проверка съедания яблока
        if snake.get_head_position() == apple.position:
            snake.length += 1
            score += 1
            apple = Apple()
            # Убедимся, что яблоко не появилось на змейке
            while apple.position in snake.positions:
                apple = Apple()
        
        screen.fill(BOARD_BACKGROUND_COLOR)
        snake.draw(screen)
        apple.draw(screen)
        draw_score(screen, score)
        pygame.display.update()
        clock.tick(10)  # Скорость игры


if name == "main":
    main()
