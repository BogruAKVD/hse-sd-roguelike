# Архитектурное описание Roguelike игры

## Общие сведения о системе

Игра представляет собой классический Roguelike с такими особенностями:
- Поклеточное перемещение персонажа
- Процедурная генерация подземелий
- Пошаговая система действий
- ФОВ (поле зрения)
- Простая система боя
- Инвентарь и сбор предметов

Игра реализована на Python с использованием библиотеки tcod (libtcod).

## Композиция (диаграмма компонентов)

```mermaid
classDiagram
    class Engine {
        + Главный цикл игры
        + Обработка ввода
        + Управление состояниями
    }
    
    class Entity {
        + Игрок
        + Монстры
        + Предметы
    }
    
    class GameMap {
        + Генерация карты
        + Поле зрения
        + Проверка проходимости
    }
    
    class InputHandlers {
        + Основное меню
        + Игровой режим
        + Режим инвентаря
    }
    
    class Renderer {
        + Отрисовка карты
        + Отрисовка сущностей
        + Отрисовка UI
    }
    
    Engine --> Entity
    Engine --> GameMap
    Engine --> InputHandlers
    Engine --> Renderer
    InputHandlers --> Engine
    Renderer --> Entity
    Renderer --> GameMap
```

Основные компоненты системы:
1. **Engine** - ядро игры, управляющее главным циклом и координацией между компонентами
2. **Entity** - представляет все объекты в игре (игрока, монстров, предметы)
3. **GameMap** - отвечает за генерацию и управление картой подземелья
4. **InputHandlers** - обработчики ввода для разных состояний игры
5. **Renderer** - система отрисовки игрового состояния

## Логическая структура (диаграмма классов)

```mermaid
classDiagram
    class Engine {
        +event_handler: EventHandler
        +game_map: GameMap
        +player: Entity
        +handle_events()
        +render()
        +update_fov()
    }
    
    class EventHandler {
        +handle_events(event: Event)
        +on_exit()
    }
    
    class GameMap {
        -width: int
        -height: int
        -tiles: ndarray
        +entities: Set[Entity]
        +in_bounds(x, y)
        +render(console)
    }
    
    class Entity {
        -x: int
        -y: int
        -char: str
        -color: Tuple[int,int,int]
        -name: str
        +move(dx, dy)
        +render(console)
    }
    
    class Actor {
        +hp: int
        +max_hp: int
        +defense: int
        +power: int
        +die()
    }
    
    class Item {
        +use()
    }
    
    class Inventory {
        -items: List[Item]
        +add(item)
        +drop(item)
        +use(item)
    }
    
    Engine --> EventHandler
    Engine --> GameMap
    Engine --> Entity: player
    GameMap --> Entity: contains
    Entity <|-- Actor
    Entity <|-- Item
    Actor --> Inventory
```

Основные классы:
- **Engine**: Центральный класс, управляющий игровым циклом
- **EventHandler**: Базовый класс для обработки ввода
- **GameMap**: Представляет игровую карту и содержит все сущности
- **Entity**: Базовый класс для всех игровых объектов
- **Actor**: Наследник Entity, представляет живые существа с характеристиками
- **Item**: Наследник Entity, представляет предметы
- **Inventory**: Управление инвентарем актора

## Взаимодействия и состояния

### Диаграмма последовательности основного игрового цикла

```mermaid
sequenceDiagram
    participant Пользователь
    participant Engine
    participant EventHandler
    participant GameMap
    participant Renderer
    
    Пользователь ->> Engine: Ввод (клавиша)
    Engine ->> EventHandler: handle_events(event)
    EventHandler ->> Engine: Действие (перемещение, атака и т.д.)
    Engine ->> GameMap: Обновление состояния
    GameMap ->> Engine: Результат обновления
    Engine ->> Renderer: render()
    Renderer ->> GameMap: Получение данных для отрисовки
    Renderer ->> Пользователь: Вывод изображения
```

### Диаграмма конечного автомата состояний игры

```mermaid
stateDiagram-v2
    [*] --> MainMenu
    MainMenu --> Gameplay: Новая игра
    MainMenu --> [*]: Выход
    Gameplay --> Inventory: Открыть инвентарь
    Gameplay --> GameOver: Смерть игрока
    Inventory --> Gameplay: Закрыть инвентарь
    GameOver --> MainMenu: Вернуться в меню
```

Состояния игры:
1. **MainMenu**: Главное меню игры (новая игра, загрузка, выход)
2. **Gameplay**: Основной игровой режим
3. **Inventory**: Режим управления инвентарем
4. **GameOver**: Состояние после смерти игрока

## Описание данных

Основные структуры данных в программе:

1. **Карта**:
   - Представлена двумерным массивом тайлов
   - Каждый тайл содержит:
     - Проходимость (можно ли ходить)
     - Прозрачность (влияет на поле зрения)
     - Видимость (видит ли игрок в данный момент)
     - Исследованность (был ли тайл виден ранее)

2. **Сущности (Entities)**:
   - Позиция (x, y) на карте
   - Символьное представление (ASCII-символ)
   - Цвет
   - Имя
   - Флаги (блокирует ли проход, подбираемость и т.д.)

3. **Акторы (Actors)**:
   - Наследуют от Entity
   - Добавляют:
     - Здоровье (HP)
     - Защита
     - Сила атаки
     - Инвентарь

4. **Предметы (Items)**:
   - Наследуют от Entity
   - Содержат эффекты при использовании

5. **Состояние игры**:
   - Текущий уровень подземелья
   - Состояние игрока
   - Список активных сущностей
   - Сообщения в логе

6. **Поле зрения (FOV)**:
   - Рассчитывается алгоритмом shadow casting
   - Хранит видимые клетки для игрока
   - Обновляется при перемещении игрока
