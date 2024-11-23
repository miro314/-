from kivy.app import App
from kivy.uix.widget import Widget
from kivy.properties import NumericProperty, ListProperty
from kivy.clock import Clock
from kivy.core.window import Window
from random import randint


# Главный игровой виджет
class FlappyBirdGame(Widget):
    bird_y = NumericProperty(Window.height // 2)  # Положение птицы по оси Y
    bird_velocity = NumericProperty(0)  # Скорость птицы
    gravity = NumericProperty(-0.5)  # Гравитация
    pipe_gap = NumericProperty(150)  # Разрыв между верхней и нижней трубами
    pipe_speed = NumericProperty(3)  # Скорость движения труб
    score = NumericProperty(0)  # Счёт игрока
    pipes = ListProperty()  # Список труб

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.pipe_width = 80
        self.bird_size = 30
        self.spawn_pipe()
        Clock.schedule_interval(self.update, 1 / 60)

    def on_touch_down(self, touch):
        # Подъем птицы
        self.bird_velocity = 10

    def spawn_pipe(self):
        # Создание новой трубы
        pipe_height = randint(100, Window.height - self.pipe_gap - 100)
        self.pipes.append({"x": Window.width, "top": pipe_height, "bottom": pipe_height + self.pipe_gap})

    def update(self, dt):
        # Обновление состояния игры
        self.bird_y += self.bird_velocity
        self.bird_velocity += self.gravity

        # Проверка столкновений с границами
        if self.bird_y < 0 or self.bird_y > Window.height:
            self.game_over()

        # Обновление труб
        for pipe in self.pipes:
            pipe["x"] -= self.pipe_speed

        # Удаление труб за экраном и добавление новых
        if self.pipes and self.pipes[0]["x"] < -self.pipe_width:
            self.pipes.pop(0)
            self.spawn_pipe()
            self.score += 1

        # Проверка столкновений с трубами
        for pipe in self.pipes:
            if (self.bird_y < pipe["top"] or self.bird_y > pipe["bottom"]) and \
               (pipe["x"] < Window.width // 4 + self.bird_size and
                pipe["x"] + self.pipe_width > Window.width // 4 - self.bird_size):
                self.game_over()

        # Перерисовка
        self.canvas.clear()
        self.draw()

    def draw(self):
        # Рисование птицы
        with self.canvas:
            from kivy.graphics import Color, Ellipse, Rectangle

            Color(0, 0, 1)
            Ellipse(pos=(Window.width // 4 - self.bird_size // 2, self.bird_y - self.bird_size // 2),
                    size=(self.bird_size, self.bird_size))

            # Рисование труб
            for pipe in self.pipes:
                Color(0, 1, 0)
                Rectangle(pos=(pipe["x"], 0), size=(self.pipe_width, pipe["top"]))
                Rectangle(pos=(pipe["x"], pipe["bottom"]), size=(self.pipe_width, Window.height - pipe["bottom"]))

            # Отображение счёта
            Color(0, 0, 0)
            Rectangle(size=(200, 40), pos=(10, Window.height - 50))
            self.canvas.add(Color(1, 1, 1))
            self.canvas.add(Rectangle(size=(200, 40), pos=(10, Window.height - 50)))

    def game_over(self):
        Clock.unschedule(self.update)
        print("Game Over! Your Score:", self.score)
        App.get_running_app().stop()


# Главный класс приложения
class FlappyBirdApp(App):
    def build(self):
        return FlappyBirdGame()


if name == "__main__":
    FlappyBirdApp().run()
