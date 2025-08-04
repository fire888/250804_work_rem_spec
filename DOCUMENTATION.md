# Документация проекта **Remplanner3D**

## 1. Общие сведения

- Все размеры задаются в **сантиметрах**.
- **Modules** создаются напрямую в приложении.
- **Systems** создаются динамическим импортом из конфигурационных файлов в зависимости от режима (`dev` / `prod`) — это помогает не раздувать итоговый бандл.

## 2. Примеры сценариев поведения

<details> 
<summary>2.1 Сдвиг текстуры пользователем</summary>
   
1. Пользователь меняет сдвиг текстуры.
2. `systems/UserInterface/panels/PanelMaterials` слушает изменение и обновляет своё состояние.
3. Панель проверяет текущий фокус элемента через `systems/Selector` и отправляет событие **`skinChangeTexture`** медиатору.
4. `modules/Skin` перехватывает событие и передаёт его в `system/Graphics/Entities`.
5. `system/Graphics/Entities` меняет материал с помощью `system/Graphics/Materials`.
6. `modules/Skin` посылает событие **`prjChange`** в `modules/Project`.
7. `modules/Project` фиксирует, что проект изменён.
</details>

<details>
<summary>2.2. Добавление декора</summary>

1. В `systems/UserInterface` на кнопку **Добавить декор** вешается обработчик, открывающий `systems/UserInterface/panels/PanelDecor`.
2. На HTML‑элемент картинки декора через `systems/UserInterface/InterfaceActions` вешается событие **`decorCreate`**.
3. `modules/Decor` перехватывает **`decorCreate`**, подгружает модель через `modules/Models`.
4. `modules/Models` бросает событие **`tmpLoadGltf`** в `Engine/systems/Graphics`.
5. `modules/Decor/Placement` выравнивает модель с помощью `modules/Decor/Lair` и инициирует событие **`plcDecorAdd`**.
6. Далее `modules/Decor/Placement`:
   - ждёт от `systems/Selector` смены активного 3D‑меша;
   - отправляет событие **`grRaySetByPlane`**;
   - переходит в режим **PLACE**.
7. `Engine/systems/Graphics/Raycast` задаёт поверхность для размещения.
8. В режиме **PLACE** `modules/Decor/Placement` (подписка на `mouseMove`) двигает меш по квартире.
9. На `mouseClick` — если режим **PLACE**, перемещение завершается.
10. На `mouseDown` — если режим **NONE**, добавляется круг поворота.
11. В режиме **ROTATE** `mouseMove` вращает меш.
12. На `mouseUp` генерируется событие **`prjChange`** (сохранить проект).
13. `modules/Project` фиксирует изменение проекта.
</details>
---

## 3. Архитектура приложения

### 3.1. Точка входа — `Remplanner3D`

- Читает `config` приложения.
- Создаёт экземпляры:

```
modules/Application
modules/Registry
modules/Project
modules/Mode
modules/Decor
modules/Skin
modules/Entourage
modules/Apartments
modules/Spatium
Engine
```

### 3.2. **Engine**

- Создаётся точкой входа `Remplanner3D`.
- По конфигу `remplanner-cfg` создаёт все **systems** (например `systems/UserInterface`).
- Все systems изолированы и общаются через **медиатор**.
- Предоставляет API для создания мешей.

### 3.3. **Modules**, создаваемые в `Remplanner3D`

| Модуль | Назначение / особенности |
|--------|-------------------------|
| `modules/Application` | Хранит права пользователя, слушает ошибки, отправляет отчёты. Создаёт пост‑процесс эффекты в зависимости от железа. Флаг `.isSandbox = window['rplanner_sandbox']` отключает отправку алертов на прод. |
| `modules/Registry` | Реестр данных для моделей (берёт `window.global_visual_data`). Отдаёт вёрстку каталогов элементов для `systems/UserInterface`. |
| `modules/Project` | Загружает и валидирует проект (класс `ProjectSource`). Хранит данные квартиры пользователя. |
| `modules/Mode` | Текущий режим взаимодействия (обычный вид, ходьба, скрытие стен). Сохраняет ракурсы камер в LocalStorage. |
| `modules/Decor` | Создаёт и добавляет в сцену меши декора. Подмодули: `Placement`, `Lair`. |
| `modules/Skin` | Управляет материалами элементов сцены. Работает с `options.saveKey`, `options.presetId`, и `modules/Skin/Presets`. |
| `modules/Entourage` | Создаёт студию для рендера. |
| `modules/Spatium` | Рендер в картинку, экспорт квартиры в `.glb`. |
| `modules/Apartments` | Генерирует «коробку» квартиры и создаёт под‑модули стен, потолка, пола и т. д. |

#### 3.3.1. Подмодули `modules/Apartments`

- **Walls**
  - `WallNiches` — ниши
  - `WallMoldings` — молдинги
  - `WallFaces`, `WallCaps`, `Generators/WallGlass` — геометрия стен
  - `WallGeometry` — вспомогательные методы
- **Portal**
  - `PortalFrame`, `PortalSlopes` — окна, двери, проёмы
- **Ceiling** — потолки
- … и др.

---

## 4. Systems

Все **systems** создаются Engine‑ом из `remplanner-cfg`, живут изолированно и общаются через медиатор.

| System | Функции |
|--------|---------|
| `systems/UserInterface` | Инициализация UI, создание панелей (`/panels`), мини‑карты и т.п.; через `InterfaceActions` отправляет события. |
| `systems/Debug` | Инструменты отладки. |
| `systems/Walker` | Активация вида от первого лица (`Engine/systems/NavigatorFPerson`), коллизии порталов. |
| `systems/Selector` | Текущий фокус на элементе; игнорирует `userData.fixed = true`. |
| `systems/BareRoom` | Обрезает квартиру по высоте (режим 1 м). |

---

## 5. Engine / Graphics

- `Engine/systems/Graphics` подключает графику к медиатору и хранит общие данные.
- **SharedData** — статическое хранилище всех мешей.
- **Mesh API** (`Graphics/Mesh`):
  - `options.userData.fixed` — игнорировать селектором
  - `options.userData.rayObstruct` — меш блокирует выбор
  - `options.userData.entity.roomId` — скрывать меш вне комнаты
  - `options.name` — уникальное имя для восстановления материалов
  - `options.material.*` — сохранение / применение материалов
  - Метод `.create(options)` генерирует геометрию, материал, меш, добавляет его в сцену и регистрирует.

Также доступны модули `SceneBasics`, `Entities`, `Geometries`, `Materials`, `Raycast`.

---

## 6. UserInterface ― детали

- `systems/UserInterface/Elements` ― утилиты для работы с HTML.
- Панели хранятся в `systems/UserInterface/panels`.

---

## 7. Аббревиатуры событий медиатора

| Сокращение | Значение |
|------------|----------|
| `gr` | Graphics |
| `grEnt` | Graphics/Entities |
| `plcDecor` | Decor/Placement |
| `regGet` | Запрос к Registry |
| `prj` | Действие Project |
| `prjGet` | Запрос к Project |
| `dbgControl` | systems/Debug (пока не используется) |
| `sel` | systems/Selector |

---

© Remplanner3D
