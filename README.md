README файл к игре:
        // Подключение библиотек для игры: 1.Директива препроцессора, которая подключает заголовочный файл из библиотеки SFML для работы с графикой. 2. Базовая библиотека для потока данных - ввод/вывод
        3.Препроцессор для работы с векторами. 4. Библиотека для работы с временем.

        
    #include <SFML/Graphics.hpp>          
    #include <iostream>                
    #include <vector>
    #include <ctime>
 // Игра написана на принципе ООП - Объектно-ориентированного программирования, то есть игра сделана не из набора различных методов, а из заранее созданных объектов, к которым принадлежит набор различных функций к каждому из них. Благодаря этому код выглядит более нагляднее, а его написание и работа с ним ещё проще.


// Создаём Pacman'а
создали класс к которому принадлежит набор методов, далее:
    
    class Pacman {
    private:
        sf::CircleShape shape;    // создали ему круглую форму
        sf::Vector2f velocity;   // задали вектор скорости
        float speed;     // скорость Пакмана
        int lives;   // количество жизней
        bool invulnerable;   // булевая переменная, указывающая на то, уезвим ли он или нет
        sf::Clock invulnerableClock;   // Объект класса sf::Clock, используемый для отслеживания времени неуязвимости Пакмана
    
    public:
        Pacman(float radius, sf::Vector2f position, float speed, int lives)
            : speed(speed), lives(lives), invulnerable(false) {
            shape.setRadius(radius);
            shape.setPosition(position);
            shape.setFillColor(sf::Color::Yellow);
            shape.setOrigin(radius, radius);
        }
    
        void move(sf::Vector2f dir) {
            velocity = dir * speed;
        }
    
        void update() {
            shape.move(velocity);
    
            // Проверяем, прошло ли время неуязвимости после съедания таблетки
            if (invulnerable && invulnerableClock.getElapsedTime().asSeconds() > 10) {
                invulnerable = false;
            }
        }
    
        void draw(sf::RenderWindow& window) {
            window.draw(shape);
        }
    
        void loseLife() {
            if (!invulnerable) {
                lives--;
                invulnerable = true;
                invulnerableClock.restart();
            }
        }
    
        int getLives() const {
            return lives;
        }
    
        bool isInvulnerable() const {
            return invulnerable;
        }
    }
    
    class Ghost {
    private:
        sf::CircleShape shape;
        sf::Clock regenerateClock;
        bool active;
    
    public:
        Ghost(float radius, sf::Vector2f position)
            : shape(radius), active(true) {
            shape.setPosition(position);
            shape.setFillColor(sf::Color::Red);
            shape.setOrigin(radius, radius);
        }
    
        void draw(sf::RenderWindow& window) {
            if (active)
                window.draw(shape);
        }
    
        void deactivate() {
            active = false;
            regenerateClock.restart();
        }
    
        bool isActive() const {
            return active;
        }
    
        bool canRegenerate() const {
            return !active && regenerateClock.getElapsedTime().asSeconds() > 5;
        }
    
        void regenerate(sf::Vector2f position) {
            shape.setPosition(position);
            active = true;
        }
    };
    
    class Game {
    private:
        sf::RenderWindow window;
        Pacman pacman;
        std::vector<Ghost> ghosts;
        int score;
    
    public:
        Game(int width, int height)
            : window(sf::VideoMode(width, height), "Pacman"),
              pacman(20, sf::Vector2f(width / 2, height - 20), 5.0f, 3),
              score(0) {
            // Создаем врагов (привидений)
            for (int i = 0; i < 4; ++i) {
                ghosts.emplace_back(15, sf::Vector2f(100 * i + 50, 100));
            }
        }
    
        void run() {
            while (window.isOpen()) {
                handleEvents();
                update();
                render();
            }
        }
    
    private:
        void handleEvents() {
            sf::Event event;
            while (window.pollEvent(event)) {
                if (event.type == sf::Event::Closed)
                    window.close();
                else if (event.type == sf::Event::KeyPressed) {
                    if (event.key.code == sf::Keyboard::Left)
                        pacman.move(sf::Vector2f(-1, 0));
                    else if (event.key.code == sf::Keyboard::Right)
                        pacman.move(sf::Vector2f(1, 0));
                    else if (event.key.code == sf::Keyboard::Up)
                        pacman.move(sf::Vector2f(0, -1));
                    else if (event.key.code == sf::Keyboard::Down)
                        pacman.move(sf::Vector2f(0, 1));
                }
            }
        }
    
        void update() {
            pacman.update();
            checkCollisions();
        }
    
        void render() {
            window.clear();
            pacman.draw(window);
            for (auto& ghost : ghosts) {
                ghost.draw(window);
            }
            window.display();
        }
    
        void checkCollisions() {
            // Проверяем столкновение с врагами
            for (auto& ghost : ghosts) {
                if (pacman.isInvulnerable() && ghost.isActive()) {
                    // Если Pacman неуязвим, и враг активен, деактивируем врага
                    if (pacman.shape.getGlobalBounds().intersects(ghost.shape.getGlobalBounds())) {
                        ghost.deactivate();
                        score += 100; // Добавляем очки за съедение врага
                    }
                } else if (pacman.shape.getGlobalBounds().intersects(ghost.shape.getGlobalBounds())) {
                    // Если Pacman неуязвим или враг неактивен, теряем жизнь
                    pacman.loseLife();
                    if (pacman.getLives() == 0) {
                        gameOver();
                    }
                }
            }
        }
    
        void gameOver() {
            // Код для завершения игры
            std::cout << "Game Over! Your score: " << score << std::endl;
            window.close();
        }
    };
    
    int main() {
        // Задаем seed для генератора случайных чисел
        srand(time(nullptr));

        Game game(800, 600);
        game.run();
    
        return 0;
    }
