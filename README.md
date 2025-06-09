import pygame
import sys
import random
import math

# 初始化pygame
pygame.init()

# 游戏常量
WIDTH, HEIGHT = 800, 600
BACKGROUND_COLOR = (25, 25, 50)
TEXT_COLOR = (220, 220, 255)
ACCENT_COLOR = (70, 130, 180)
HIGHLIGHT_COLOR = (255, 215, 0)
BUTTON_COLOR = (50, 100, 150)
BUTTON_HOVER_COLOR = (80, 140, 200)
INPUT_BOX_COLOR = (40, 70, 100)
INPUT_BOX_ACTIVE_COLOR = (60, 100, 140)

# 创建游戏窗口
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("猜数字游戏")
clock = pygame.time.Clock()

# 创建字体
title_font = pygame.font.SysFont(None, 64)
main_font = pygame.font.SysFont(None, 48)
small_font = pygame.font.SysFont(None, 36)

class Button:
    def __init__(self, x, y, width, height, text, action=None):
        self.rect = pygame.Rect(x, y, width, height)
        self.text = text
        self.action = action
        self.hovered = False
        
    def draw(self, surface):
        color = BUTTON_HOVER_COLOR if self.hovered else BUTTON_COLOR
        pygame.draw.rect(surface, color, self.rect, border_radius=12)
        pygame.draw.rect(surface, HIGHLIGHT_COLOR, self.rect, 3, border_radius=12)
        
        text_surf = main_font.render(self.text, True, TEXT_COLOR)
        text_rect = text_surf.get_rect(center=self.rect.center)
        surface.blit(text_surf, text_rect)
        
    def check_hover(self, pos):
        self.hovered = self.rect.collidepoint(pos)
        
    def handle_event(self, event):
        if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            if self.hovered and self.action:
                self.action()
                return True
        return False

class InputBox:
    def __init__(self, x, y, width, height):
        self.rect = pygame.Rect(x, y, width, height)
        self.text = ""
        self.active = False
        
    def handle_event(self, event):
        if event.type == pygame.MOUSEBUTTONDOWN:
            self.active = self.rect.collidepoint(event.pos)
            
        if event.type == pygame.KEYDOWN and self.active:
            if event.key == pygame.K_RETURN:
                return self.text
            elif event.key == pygame.K_BACKSPACE:
                self.text = self.text[:-1]
            elif event.key in (pygame.K_0, pygame.K_1, pygame.K_2, pygame.K_3, pygame.K_4, 
                             pygame.K_5, pygame.K_6, pygame.K_7, pygame.K_8, pygame.K_9):
                if len(self.text) < 3:
                    self.text += event.unicode
        return None
        
    def draw(self, surface):
        color = INPUT_BOX_ACTIVE_COLOR if self.active else INPUT_BOX_COLOR
        pygame.draw.rect(surface, color, self.rect, border_radius=8)
        pygame.draw.rect(surface, HIGHLIGHT_COLOR, self.rect, 2, border_radius=8)
        
        text_surf = main_font.render(self.text, True, TEXT_COLOR)
        surface.blit(text_surf, (self.rect.x + 15, self.rect.y + 10))
        
        # 绘制输入提示
        if not self.text and not self.active:
            prompt_surf = small_font.render("输入数字(1-100)", True, (150, 150, 180))
            surface.blit(prompt_surf, (self.rect.x + 15, self.rect.y + 15))

class Game:
    def __init__(self):
        self.reset_game()
        self.input_box = InputBox(WIDTH//2 - 100, HEIGHT//2 + 40, 200, 60)
        self.guess_button = Button(WIDTH//2 - 100, HEIGHT//2 + 120, 200, 60, "猜一下", self.make_guess)
        self.restart_button = Button(WIDTH//2 - 100, HEIGHT//2 + 200, 200, 60, "重新开始", self.reset_game)
        
    def reset_game(self):
        self.secret_number = random.randint(1, 100)
        self.guesses = []
        self.message = "猜一个1到100之间的数字"
        self.game_state = "playing"  # playing, won, lost
        
    def make_guess(self):
        if self.input_box.text and self.game_state == "playing":
            try:
                guess = int(self.input_box.text)
                if 1 <= guess <= 100:
                    self.guesses.append(guess)
                    
                    if guess == self.secret_number:
                        self.message = f"恭喜！你猜对了！数字是{self.secret_number}"
                        self.game_state = "won"
                    elif guess < self.secret_number:
                        self.message = f"{guess} 太小了！再试一次"
                    else:
                        self.message = f"{guess} 太大了！再试一次"
                        
                    if len(self.guesses) >= 8 and self.game_state != "won":
                        self.message = f"游戏结束！正确数字是 {self.secret_number}"
                        self.game_state = "lost"
                    
                    self.input_box.text = ""
                else:
                    self.message = "请输入1到100之间的数字"
            except ValueError:
                self.message = "请输入有效的数字"
    
    def draw(self, surface):
        # 绘制背景
        surface.fill(BACKGROUND_COLOR)
        
        # 绘制标题
        title_surf = title_font.render("猜数字游戏", True, HIGHLIGHT_COLOR)
        surface.blit(title_surf, (WIDTH//2 - title_surf.get_width()//2, 50))
        
        # 绘制当前状态信息
        message_surf = main_f
