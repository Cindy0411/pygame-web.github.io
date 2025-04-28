pip install pygbag
pygbag your_game.py
import pygame
import random
import sys
import time

# 初始化pygame
pygame.init()
pygame.font.init()

# 屏幕設置
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("句子冒險島：拯救詞語精靈")

# 顏色定義
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
BLUE = (100, 149, 237)
GREEN = (34, 139, 34)
RED = (255, 99, 71)
YELLOW = (255, 215, 0)
PURPLE = (147, 112, 219)

# 字體設置
title_font = pygame.font.SysFont("mingliu", 50)
normal_font = pygame.font.SysFont("mingliu", 30)
small_font = pygame.font.SysFont("mingliu", 20)

# 遊戲數據
class GameData:
    def __init__(self):
        self.level = 1
        self.score = 0
        self.lives = 3
        self.current_sentence = ""
        self.words = []
        self.selected_words = []
        self.correct_sentences = [
            "小貓在草地上玩耍",
            "媽媽做了一個美味的蛋糕",
            "太陽從東方升起",
            "弟弟喜歡吃蘋果",
            "我們一起去公園玩",
            "老師教我們認字",
            "小狗汪汪叫",
            "春天花兒開了",
            "妹妹有一本漂亮的書",
            "爸爸每天去上班"
        ]
        self.incorrect_sentences = [
            "玩耍小貓在草地上",
            "蛋糕媽媽做了一個美味的",
            "升起太陽從東方",
            "蘋果弟弟喜歡吃",
            "玩我們一起去公園"
        ]
        self.backgrounds = ["forest", "castle", "cave", "beach", "mountain"]
        self.current_bg = "forest"
        self.character_pos = [100, 400]
        self.target_pos = [600, 400]
        self.character_img = pygame.Surface((50, 80), pygame.SRCALPHA)
        pygame.draw.ellipse(self.character_img, (255, 218, 185), (0, 0, 50, 50))
        pygame.draw.rect(self.character_img, (65, 105, 225), (10, 40, 30, 40))
        self.enemy_img = pygame.Surface((60, 60), pygame.SRCALPHA)
        pygame.draw.polygon(self.enemy_img, RED, [(30, 0), (0, 60), (60, 60)])
        self.treasure_img = pygame.Surface((40, 40), pygame.SRCALPHA)
        pygame.draw.rect(self.treasure_img, YELLOW, (0, 0, 40, 40))
        pygame.draw.rect(self.treasure_img, (218, 165, 32), (5, 5, 30, 30))
        
    def get_new_sentence(self):
        # 隨機選擇正確或錯誤句子
        if random.random() < 0.7 or self.level < 3:
            self.current_sentence = random.choice(self.correct_sentences)
            self.words = self.current_sentence.split()
            random.shuffle(self.words)
            self.selected_words = []
        else:
            # 更高級別時加入錯誤句子
            self.current_sentence = random.choice(self.incorrect_sentences)
            self.words = self.current_sentence.split()
            self.selected_words = []
    
    def check_answer(self):
        # 檢查是否組成正確句子
        player_sentence = "".join(self.selected_words)
        return player_sentence == self.current_sentence

# 遊戲主類
class SentenceAdventure:
    def __init__(self):
        self.data = GameData()
        self.state = "menu"  # menu, game, result
        self.buttons = []
        self.word_buttons = []
        self.selected_word_buttons = []
        self.setup_menu()
        self.data.get_new_sentence()
        self.setup_word_buttons()
        self.message = ""
        self.message_timer = 0
        self.progress = 0
        self.max_progress = 10
        
    def setup_menu(self):
        self.buttons = [
            {"rect": pygame.Rect(300, 300, 200, 60), "text": "開始冒險", "action": "start"},
            {"rect": pygame.Rect(300, 400, 200, 60), "text": "遊戲說明", "action": "help"},
            {"rect": pygame.Rect(300, 500, 200, 60), "text": "離開遊戲", "action": "quit"}
        ]
    
    def setup_word_buttons(self):
        self.word_buttons = []
        self.selected_word_buttons = []
        word_count = len(self.data.words)
        
        # 計算按鈕位置
        for i, word in enumerate(self.data.words):
            x = 100 + (i % 5) * 140
            y = 150 + (i // 5) * 60
            self.word_buttons.append({
                "rect": pygame.Rect(x, y, 120, 40),
                "text": word,
                "visible": True
            })
        
        # 已選詞語的位置
        for i in range(len(self.data.selected_words)):
            x = 100 + i * 120
            y = 350
            self.selected_word_buttons.append({
                "rect": pygame.Rect(x, y, 120, 40),
                "text": self.data.selected_words[i],
                "visible": True
            })
    
    def draw_menu(self):
        # 繪製背景
        screen.fill(BLUE)
        
        # 繪製標題
        title = title_font.render("句子冒險島", True, WHITE)
        subtitle = normal_font.render("拯救詞語精靈", True, YELLOW)
        screen.blit(title, (WIDTH//2 - title.get_width()//2, 100))
        screen.blit(subtitle, (WIDTH//2 - subtitle.get_width()//2, 180))
        
        # 繪製按鈕
        for button in self.buttons:
            pygame.draw.rect(screen, GREEN, button["rect"], border_radius=10)
            pygame.draw.rect(screen, BLACK, button["rect"], 2, border_radius=10)
            text = normal_font.render(button["text"], True, BLACK)
            screen.blit(text, (button["rect"].x + button["rect"].width//2 - text.get_width()//2, 
                               button["rect"].y + button["rect"].height//2 - text.get_height()//2))
        
        # 繪製角色
        screen.blit(self.data.character_img, (WIDTH//2 - 25, 220))
    
    def draw_game(self):
        # 繪製背景
        if self.data.current_bg == "forest":
            screen.fill((34, 139, 34))  # 森林綠
        elif self.data.current_bg == "castle":
            screen.fill((176, 196, 222))  # 城堡灰
        elif self.data.current_bg == "cave":
            screen.fill((105, 105, 105))  # 洞穴灰
        elif self.data.current_bg == "beach":
            screen.fill(173, 216, 230))  # 海灘藍
        else:
            screen.fill(255, 228, 196))  # 山脈棕
            
        # 繪製遊戲信息
        level_text = small_font.render(f"關卡: {self.data.level}", True, BLACK)
        score_text = small_font.render(f"分數: {self.data.score}", True, BLACK)
        lives_text = small_font.render(f"生命: {'❤' * self.data.lives}", True, RED)
        progress_text = small_font.render(f"進度: {self.progress}/{self.max_progress}", True, BLACK)
        
        screen.blit(level_text, (20, 20))
        screen.blit(score_text, (20, 50))
        screen.blit(lives_text, (20, 80))
        screen.blit(progress_text, (20, 110))
        
        # 繪製提示
        hint = normal_font.render("請將詞語按正確順序排列:", True, BLACK)
        screen.blit(hint, (WIDTH//2 - hint.get_width()//2, 100))
        
        # 繪製詞語按鈕
        for button in self.word_buttons:
            if button["visible"]:
                pygame.draw.rect(screen, WHITE, button["rect"], border_radius=5)
                pygame.draw.rect(screen, BLACK, button["rect"], 2, border_radius=5)
                text = small_font.render(button["text"], True, BLACK)
                screen.blit(text, (button["rect"].x + button["rect"].width//2 - text.get_width()//2, 
                                 button["rect"].y + button["rect"].height//2 - text.get_height()//2))
        
        # 繪製已選詞語
        for button in self.selected_word_buttons:
            pygame.draw.rect(screen, YELLOW, button["rect"], border_radius=5)
            pygame.draw.rect(screen, BLACK, button["rect"], 2, border_radius=5)
            text = small_font.render(button["text"], True, BLACK)
            screen.blit(text, (button["rect"].x + button["rect"].width//2 - text.get_width()//2, 
                             button["rect"].y + button["rect"].height//2 - text.get_height()//2))
        
        # 繪製確認按鈕
        confirm_rect = pygame.Rect(WIDTH//2 - 60, 450, 120, 50)
        pygame.draw.rect(screen, GREEN, confirm_rect, border_radius=10)
        pygame.draw.rect(screen, BLACK, confirm_rect, 2, border_radius=10)
        confirm_text = normal_font.render("確認", True, BLACK)
        screen.blit(confirm_text, (confirm_rect.x + confirm_rect.width//2 - confirm_text.get_width()//2, 
                                 confirm_rect.y + confirm_rect.height//2 - confirm_text.get_height()//2))
        
        # 繪製角色和目標
        screen.blit(self.data.character_img, self.data.character_pos)
        screen.blit(self.data.enemy_img, (self.data.target_pos[0] - 30, self.data.target_pos[1] - 60))
        screen.blit(self.data.treasure_img, (self.data.target_pos[0] - 20, self.data.target_pos[1] - 20))
        
        # 繪製提示信息
        if self.message and self.message_timer > 0:
            msg = normal_font.render(self.message, True, RED if "錯" in self.message else GREEN)
            screen.blit(msg, (WIDTH//2 - msg.get_width()//2, 400))
            self.message_timer -= 1
    
    def draw_help(self):
        screen.fill(BLUE)
        title = title_font.render("遊戲說明", True, WHITE)
        screen.blit(title, (WIDTH//2 - title.get_width()//2, 50))
        
        instructions = [
            "歡迎來到句子冒險島！",
            "",
            "邪惡的詞語怪獸打亂了所有的句子，",
            "你需要幫助主角重新排列詞語，",
            "組成正確的句子來擊敗怪獸！",
            "",
            "玩法:",
            "1. 點擊詞語按鈕選擇詞語",
            "2. 詞語會出現在下方區域",
            "3. 按正確順序排列所有詞語",
            "4. 點擊確認按鈕檢查答案",
            "",
            "每過5關，難度會提升！",
            "收集足夠的詞語寶石就能拯救精靈！"
        ]
        
        for i, line in enumerate(instructions):
            text = small_font.render(line, True, WHITE)
            screen.blit(text, (WIDTH//2 - text.get_width()//2, 120 + i * 25))
        
        back_rect = pygame.Rect(WIDTH//2 - 60, 500, 120, 50)
        pygame.draw.rect(screen, GREEN, back_rect, border_radius=10)
        pygame.draw.rect(screen, BLACK, back_rect, 2, border_radius=10)
        back_text = normal_font.render("返回", True, BLACK)
        screen.blit(back_text, (back_rect.x + back_rect.width//2 - back_text.get_width()//2, 
                             back_rect.y + back_rect.height//2 - back_text.get_height()//2))
        self.buttons = [{"rect": back_rect, "text": "返回", "action": "back"}]
    
    def draw_result(self, win):
        screen.fill(GREEN if win else RED)
        
        if win:
            result_text = title_font.render("恭喜你贏了！", True, YELLOW)
            score_text = normal_font.render(f"最終分數: {self.data.score}", True, WHITE)
            screen.blit(result_text, (WIDTH//2 - result_text.get_width()//2, 200))
            screen.blit(score_text, (WIDTH//2 - score_text.get_width()//2, 300))
            
            # 繪製獎杯
            trophy = pygame.Surface((100, 100), pygame.SRCALPHA)
            pygame.draw.ellipse(trophy, YELLOW, (25, 0, 50, 60))
            pygame.draw.rect(trophy, YELLOW, (40, 50, 20, 40))
            screen.blit(trophy, (WIDTH//2 - 50, 350))
        else:
            result_text = title_font.render("遊戲結束", True, BLACK)
            score_text = normal_font.render(f"你的分數: {self.data.score}", True, WHITE)
            screen.blit(result_text, (WIDTH//2 - result_text.get_width()//2, 200))
            screen.blit(score_text, (WIDTH//2 - score_text.get_width()//2, 300))
        
        restart_rect = pygame.Rect(WIDTH//2 - 100, 450, 200, 60)
        pygame.draw.rect(screen, BLUE, restart_rect, border_radius=10)
        pygame.draw.rect(screen, BLACK, restart_rect, 2, border_radius=10)
        restart_text = normal_font.render("再玩一次", True, WHITE)
        screen.blit(restart_text, (restart_rect.x + restart_rect.width//2 - restart_text.get_width()//2, 
                                 restart_rect.y + restart_rect.height//2 - restart_text.get_height()//2))
        
        self.buttons = [{"rect": restart_rect, "text": "再玩一次", "action": "restart"}]
    
    def handle_click(self, pos):
        if self.state == "menu":
            for button in self.buttons:
                if button["rect"].collidepoint(pos):
                    if button["action"] == "start":
                        self.state = "game"
                    elif button["action"] == "help":
                        self.state = "help"
                    elif button["action"] == "quit":
                        pygame.quit()
                        sys.exit()
        elif self.state == "help":
            for button in self.buttons:
                if button["rect"].collidepoint(pos) and button["action"] == "back":
                    self.state = "menu"
        elif self.state == "game":
            # 檢查詞語按鈕
            for button in self.word_buttons:
                if button["visible"] and button["rect"].collidepoint(pos):
                    self.data.selected_words.append(button["text"])
                    button["visible"] = False
                    self.setup_word_buttons()
                    break
            
            # 檢查確認按鈕
            confirm_rect = pygame.Rect(WIDTH//2 - 60, 450, 120, 50)
            if confirm_rect.collidepoint(pos) and self.data.selected_words:
                if self.data.check_answer():
                    self.data.score += 10 * self.data.level
                    self.progress += 1
                    self.message = "答對了！獲得寶石！"
                    self.message_timer = 60
                    
                    # 角色移動動畫
                    for i in range(10):
                        self.data.character_pos[0] += 20
                        self.draw_game()
                        pygame.display.flip()
                        time.sleep(0.05)
                    
                    # 檢查是否升級
                    if self.progress >= self.max_progress:
                        self.data.level += 1
                        self.progress = 0
                        self.max_progress += 5
                        self.message = f"升級到第{self.data.level}關！"
                        self.message_timer = 60
                        
                        # 更換背景
                        self.data.current_bg = random.choice(self.data.backgrounds)
                else:
                    self.data.lives -= 1
                    self.message = "答錯了！再試一次！"
                    self.message_timer = 60
                    
                    if self.data.lives <= 0:
                        self.state = "result"
                        return
                
                # 重置當前題目
                self.data.get_new_sentence()
                self.setup_word_buttons()
                
                # 角色回到原位
                self.data.character_pos = [100, 400]
        elif self.state == "result":
            for button in self.buttons:
                if button["rect"].collidepoint(pos) and button["action"] == "restart":
                    self.__init__()
    
    def run(self):
        clock = pygame.time.Clock()
        running = True
        
        while running:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    if event.button == 1:  # 左鍵點擊
                        self.handle_click(event.pos)
            
            if self.state == "menu":
                self.draw_menu()
            elif self.state == "game":
                self.draw_game()
            elif self.state == "help":
                self.draw_help()
            elif self.state == "result":
                self.draw_result(self.data.score >= 100)  # 假設100分為勝利
            
            pygame.display.flip()
            clock.tick(60)
        
        pygame.quit()
        sys.exit()

# 啟動遊戲
if __name__ == "__main__":
    game = SentenceAdventure()
    game.run()
