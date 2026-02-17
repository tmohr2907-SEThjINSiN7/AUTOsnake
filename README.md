echo "# SnakeAI – Data‑Eating Snake Game" > README.md
echo "__pycache__/" > .gitignore
echo "*.pyc" >> .gitignore
echo "*.pyo" >> .gitignore
echo "*.pyd" >> .gitignore
echo ".env" >> .gitignore
echo ".vscode/" >> .gitignore
echo ".idea/" >> .gitignore

cat > LICENSE << 'EOF'
MIT License

Copyright (c) 2026 Tobias

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
EOF

git init
git add .
git commit -m "Initial commit: SnakeAI engine"__pycache__/
*.pyc
*.pyo
*.pyd
.env
.vscode/
.idea/MIT License

Copyright (c) 2026 Tobias

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.snake-ai/
│
├── game_logic.py
├── README.md
├── LICENSE
└── .gitignoregit remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git branch -M main
git push -u origin main# SnakeAI – Data‑Eating Snake Game

A Python-based Snake engine where the snake consumes data units  
(Bits → Kilobits → Megabytes → Gigabytes → Terabytes).  
Includes an autopilot AI, toroidal wraparound movement, and thread‑safe updates.

## Features
- Greedy autopilot pathfinding
- Data‑size scoring system
- Toroidal arena (wraparound edges)
- Thread‑safe game loop
- Human‑readable score formatting

## File Overview
### `game_logic.py`
Contains the full SnakeAI class:
- movement logic  
- food spawning  
- scoring  
- autopilot  
- collision detection  
- status reporting  

## Requirements
Python 3.8+

No external dependencies required.

## License
MIT License – free to use, modify, and distribute.# game_logic.py
import random
import time
import threading

DELAY = 0.05
DATA_SIZES = [
    ("1 Bit", 1, "white"),
    ("1 Kilobit", 1000, "yellow"),
    ("1 Megabyte", 1000000, "orange"),
    ("1 Gigabyte", 1000000000, "red"),
    ("1 Terabyte", 1000000000000, "purple")
]

class SnakeAI:
    def __init__(self):
        self.score_bits = 0
        self.segments = []
        self.head = (0, 0)
        self.direction = "right"
        self.food = (0, 0)
        self.running = False
        self.lock = threading.Lock()
        self.spawn_food()

    def format_size(self, bits):
        if bits < 1000: return f"{bits} Bit"
        elif bits < 1000000: return f"{bits/1000:.1f} KB"
        elif bits < 1000000000: return f"{bits/1000000:.1f} MB"
        elif bits < 1000000000000: return f"{bits/1000000000:.1f} GB"
        else: return f"{bits/1000000000000:.2f} TB"

    def spawn_food(self):
        x = random.randint(-14, 14) * 20
        y = random.randint(-14, 14) * 20
        self.food = (x, y)
        self.current_food_data = random.choice(DATA_SIZES)

    def is_collision(self, x, y):
        return (x, y) in self.segments

    def autopilot(self):
        x, y = self.head
        directions = {
            "up": (x, y + 20),
            "down": (x, y - 20),
            "left": (x - 20, y),
            "right": (x + 20, y)
        }
        opposites = {"up": "down", "down": "up", "left": "right", "right": "left"}

        best_dir = self.direction
        min_dist = float("inf")

        for direction, (nx, ny) in directions.items():
            if direction == opposites.get(self.direction):
                continue

            tx, ty = nx, ny
            if tx > 280: tx = -280
            elif tx < -280: tx = 280
            if ty > 280: ty = -280
            elif ty < -280: ty = 280

            if not self.is_collision(tx, ty):
                fx, fy = self.food
                dist = ((tx - fx)**2 + (ty - fy)**2)**0.5
                if dist < min_dist:
                    min_dist = dist
                    best_dir = direction

        self.direction = best_dir

    def step(self):
        with self.lock:
            # Segmente bewegen
            if self.segments:
                self.segments = [self.head] + self.segments[:-1]

            # Kopf bewegen
            x, y = self.head
            if self.direction == "up": y += 20
            elif self.direction == "down": y -= 20
            elif self.direction == "left": x -= 20
            elif self.direction == "right": x += 20

            # Wrap
            if x > 290: x = -290
            elif x < -290: x = 290
            if y > 290: y = -290
            elif y < -290: y = 290

            self.head = (x, y)

            # Futter
            if abs(x - self.food[0]) < 20 and abs(y - self.food[1]) < 20:
                label, value, color = self.current_food_data
                self.score_bits += value
                # Neues Segment
                self.segments.append(self.head)
                self.spawn_food()

    def get_status(self):
        with self.lock:
            return {
                "score_bits": self.score_bits,
                "score_human": self.format_size(self.score_bits),
                "length": len(self.segments) + 1,
                "head": self.head,
                "food": self.food
            }pip install python-telegram-botimport logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, ContextTypes

# Konfiguration
BOT_TOKEN = "DEIN_TELEGRAM_BOT_TOKEN"
# Die URL deiner GitHub Pages Seite
GAME_URL = "https://DEIN_USERNAME.github.io/snake-game/"
# Der 'Short Name', den du beim @BotFather mit /newgame vergeben hast
GAME_SHORT_NAME = "snake" 

# Logging einrichten
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Sendet eine Nachricht mit dem Spiel-Button."""
    # Der Bot sendet das Spiel-Objekt
    await update.message.reply_game(GAME_SHORT_NAME)

async def play_game_callback(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Reagiert auf den 'Play'-Button des Spiels."""
    query = update.callback_query
    
    # Überprüfen, ob die Anfrage vom richtigen Spiel kommt
    if query.game_short_name == GAME_SHORT_NAME:
        # Hier wird die URL des GitHub-Spiels an den Telegram-Browser übergeben
        await query.answer(url=GAME_URL)
    else:
        await query.answer("Spiel nicht gefunden.")

def main():
    # Bot erstellen
    application = Application.builder().token(BOT_TOKEN).build()

    # Handler hinzufügen
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CallbackQueryHandler(play_game_callback))

    # Bot starten
    print("Bot läuft...")
    application.run_polling()

if __name__ == "__main__":
    main()
__pycache__/
*.pyc
*.pyo
*.pyd
.env
.vscode/
.idea/# AUTOsnake
Autopilot snake. Endless pixel PIX
