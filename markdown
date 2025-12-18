# Простое веб-приложение Kanban-доска

Простое веб-приложение для управления задачами в стиле Kanban. Вы можете добавлять задачи, перемещать их между столбцами и сохранять изменения в вашем браузере.

## Функциональность:
- Три основных столбца: To Do, In Progress, Done.
- Возможность перетаскивания задач между столбцами.### Реализация простого веб-приложения Канбан-доска

**Краткое описание:**  
Это приложение представляет собой простую канбан-доску с тремя основными колонками («To Do», «In Progress», «Done»). Задача состоит в создании функционального приложения, которое позволяет пользователям создавать карточки задач, перемещать их между колонками с помощью drag-and-drop API, сохранять текущее состояние доски и отображать подсветку целевой колонки при наведении мыши.

---

## Требования к проекту:

1. **Три основные колонки**: To Do, In Progress, Done.
2. **Перетаскиваемые карточки задач**: возможность перемещения карточек с использованием Drag & Drop API HTML5.
3. **Подсветка колонки**: целевая колонка подсвечивается при наведении карты.
4. **Закрепление карточки**: после сброса карта фиксируется в нужной колонке.
5. **Создание новых задач**: ввод текста + кнопка для добавления новой задачи.
6. **Сохранение состояния**: сохранение текущего положения всех задач в браузере (`localStorage`).
7. **Оформление**: применение стилей CSS для приятного внешнего вида интерфейса.

---

## Демонстрационный пример:

Для начала давайте рассмотрим минимально рабочий код проекта.
      display: flex;
        justify-content: space-between;
    }
    
    .column {
        width: 30%;
        min-height: 300px;
        border-radius: 5px;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        padding: 10px;
        background-color: white;
        position: relative;
    }
    
    h2 {
        text-align: center;
        color: #333;
    }
    
    ul {
        list-style-type: none;
        padding: 0;
        margin: 0;
    }
    
    li.card {
        background-color: #fff;
        border: 1px solid #ccc;
        border-radius: 5px;
        padding: 10px;
        margin-bottom: 10px;
        cursor: grab;
        transition: transform 0.1s ease-in-out;
    }
    
    /* Подсветка колонки */
    .column.highlighted {
        outline: 2px dashed green;
    }
    
    input[type=text], button {
        padding: 10px;
        font-size: 16px;
        margin-right: 10px;
    }
    
    form {
        display: flex;
        align-items: center;
        margin-top: 20px;
    }
</style>
</head>
<body>
    
<div class="board-container">
    <div id="todo-column" class="column">
        <h2>To Do</h2>
        <ul></ul>
    </div>
    <div id="in-progress-column" class="column">
        <h2>In Progress</h2>
        <ul></ul>
    </div>
    <div id="done-column" class="column">
        <h2>Done</h2>
        <ul></ul>
    </div>
</div>

<!-- Форма для ввода новых задач -->
<form onsubmit="addTask(event); return false;">
    <input type="text" id="task-input" placeholder="Enter task name..." required />
    <button type="submit">Add Task</button>
</form>

<script>
// Функция сохранения данных в localStorage
const saveState = () => {
    const state = {
        todo: Array.from(document.querySelector('#todo-column > ul').children).map(card => card.textContent),
        inProgress: Array.from(document.querySelector('#in-progress-column > ul').children).map(card => card.textContent),
        done: Array.from(document.querySelector('#done-column > ul').children).map(card => card.textContent)
    };
    localStorage.setItem('kanban-state', JSON.stringify(state));
};

// Функция загрузки сохраненного состояния
const loadState = () => {
    const savedState = localStorage.getItem('kanban-state');
    if (!savedState) return;
    const { todo, inProgress, done } = JSON.parse(savedState);
    ['#todo-column', '#in-progress-column', '#done-column'].forEach((selector, index) => {
        document.querySelector(`${selector} > ul`).innerHTML = '';
        const columnData = [todo, inProgress, done][index];
        columnData.forEach(taskName => addCard(selector, taskName));
    });
};

// Создание карточки задачи
const createCardElement = (taskName) => {
    const card = document.createElement("li");
    card.classList.add("card");
    card.draggable = true;
    card.textContent = taskName;
    card.ondragstart = handleDragStart;
    return card;
};

// Обработчик перетаскивания элемента
let draggedItem = null;
const handleDragStart = (event) => {
    event.dataTransfer.effectAllowed = 'move';
    event.dataTransfer.setData('text/plain', event.target.id || '');
    draggedItem = event.target;
    setTimeout(() => {
        event.target.style.display = 'none'; // временно скрываем элемент для плавного эффекта перетаскивания
    }, 0);
};

// Проверяем над какой колонкой находится перетаскиваемый объект
document.body.addEventListener('dragover', function(e) {
    e.preventDefault();
});

// Подсветка колонки при наведении перетаскиваемого объекта
document.querySelectorAll('.column').forEach(column => {
    column.addEventListener('dragenter', highlightColumn);
    column.addEventListener('dragleave', unhighlightColumn);
});

// Подсветка колонки
const highlightColumn = (event) => {
    event.currentTarget.classList.add('highlighted');
};

// Убираем подсветку колонки
const unhighlightColumn = (event) => {
    event.currentTarget.classList.remove('highlighted');
};

// Фиксируем карточку в колонке после завершения перетаскивания
document.querySelectorAll('.column').forEach(column => {
    column.addEventListener('drop', handleDrop);
});

// Определяем куда помещается карточка
const handleDrop = (event) => {
    event.preventDefault();
    event.stopPropagation(); // Остановка распространения события вверх по DOM
    const targetColumn = event.currentTarget;
    const newPosition = event.clientY - targetColumn.offsetTop;
    let dropLocationIndex = Math.floor(newPosition / 50); // пример расчета индекса вставки исходя из высоты элементов
    if(dropLocationIndex >= targetColumn.children.length){
        dropLocationIndex = targetColumn.children.length;
    }
    targetColumn.insertBefore(draggedItem, targetColumn.children[dropLocationIndex]);
    draggedItem.style.display = ''; // Показываем элемент обратно
    saveState(); // Сохраняем новое положение карт
};

// Создаем новую задачу и добавляем её в список ToDo
const addTask = (event) => {
    event.preventDefault();
    const taskInput = document.getElementById('task-input');
    const taskText = taskInput.value.trim();
    if (taskText === '') return;
    addCard('#todo-column', taskText);
    taskInput.value = '';
    saveState();
};

// Добавляем карту в заданную колонку
const addCard = (targetColumnSelector, taskName) => {
    const targetUl = document.querySelector(`${targetColumnSelector} > ul`);
    const card = createCardElement(taskName);
    targetUl.appendChild(card);
};

loadState(); // Загружаем сохраненное состояние при запуске страницы
</script>

</body>
</html>
```
# Простое веб-приложение Kanban-доска

Простое веб-приложение для управления задачами в стиле Kanban. Вы можете добавлять задачи, перемещать их между столбцами и сохранять изменения в вашем браузере.

## Функциональность:
- Три основных столбца: To Do, In Progress, Done.
- Возможность перетаскивания задач между столбцами.
- Подсветка цели при наведении.
- Поддержка сохранения данных в localStorage.
- Форматирование и оформление CSS.

## Установка и использование:
1. Скачайте файлы проекта.
2. Запустите `index.html` в браузере.
3. Используйте приложение для организации ваших задач!

## Пример использования:
1. Нажмите кнопку «Add Task» и введите название вашей задачи.
2. Перемещайте задачи между столбцами, используя перетаскивание.

## Развертывание:
Чтобы разместить на GitHub Pages:
1. Создайте репозиторий на GitHub.
2. Залейте туда ваши файлы.
3. Включите опцию GitHub Pages.

## Автор:
Ваше имя здесь
