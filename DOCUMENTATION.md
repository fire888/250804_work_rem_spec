# Документация проекта **Remplanner3D**

## 1. Общие сведения

- Все размеры задаются в **сантиметрах**.
- **Modules** создаются напрямую в приложении.
- **Systems** создаются динамическим импортом из конфигурационных файлов в зависимости от режима (`dev` / `prod`) — это помогает не раздувать итоговый бандл.


## 2. Примеры сценариев поведения

**2.1 Сдвиг текстуры пользователем**

1. [systems/**UserInterface**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/UserInterface.js?ref_type=heads) вешает событие на инпут и при изменении пользователем свойства дергает метод в **PanelMaterials**  
1. [systems/UserInterface/sections/**PanelMaterials**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/sections/PanelMaterials.js?ref_type=heads) запрашивает у [systems/**Selector**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/Selector/Selector.js?ref_type=heads) текущий фокус и отправляет событие *skinChangeTexture* медиатору.
1. [modules/**Skin**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Skin.js?ref_type=heads) перехватывает событие и передаёт его в **Entities**.
1. [system/Graphics/**Entities**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Entities.js?ref_type=heads) меняет материал с помощью [system/Graphics/**Materials**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Materials.js?ref_type=heads).
1. [modules/**Skin**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Skin.js?ref_type=heads) посылает событие *prjChange* в **Project**.
1. [modules/**Project**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/Project.js?ref_type=heads) фиксирует, что проект изменён.


**2.2. Добавление декора**

1. В [systems/**UserInterface**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/UserInterface.js?ref_type=heads) на кнопку **Добавить декор** вешается обработчик, открывающий [systems/UserInterface/sections/**PanelDecor**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/sections/PanelDecor.js?ref_type=heads).
1. На HTML‑элемент картинки декора через [systems/UserInterface/InterfaceActions](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/InterfaceActions.js?ref_type=heads) вешается событие *decorCreate*.
1. [modules/**Decor**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Decor.js?ref_type=heads) перехватывает *decorCreate*, подгружает модель через [modules/**Models**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Model/Model.js?ref_type=heads).
1. [modules/**Models**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Model/Model.js?ref_type=heads) бросает событие *tmpLoadGltf* в [Engine/systems/Graphics](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Graphics.js?ref_type=heads).
1. [modules/Decor/**Placement**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Placement/Placement.js?ref_type=heads) выравнивает модель с помощью [modules/Decor/**Lair**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Lair.js?ref_type=heads) и инициирует событие *plcDecorAdd*.
1. Далее [modules/Decor/**Placement**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Placement/Placement.js?ref_type=heads):
   - ждёт от [systems/**Selector**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/Selector/Selector.js?ref_type=heads) смены активного 3D‑меша;
   - отправляет событие *grRaySetByPlane*;
   - переходит в режим *PLACE*.
1. [Engine/systems/Graphics/Raycast](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Raycast.js?ref_type=heads) задаёт поверхность для размещения.
1. [modules/Decor/**Placement**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Placement/Placement.js?ref_type=heads) в режиме **PLACE**  (подписка на `mouseMove`) двигает меш по квартире.
   - На `mouseClick` — если режим **PLACE**, перемещение завершается.
   - На `mouseDown` — если режим **NONE**, добавляется круг поворота.
   - В режиме **ROTATE** `mouseMove` вращает меш.
   - На `mouseUp` генерируется событие **`prjChange`** (сохранить проект).
1. [modules/**Project**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/Project.js?ref_type=heads) фиксирует изменение проекта.


## 3. Архитектура приложения

### 3.1 Основные узлы

[**Remplanner3D**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/RPlanner3D.js?ref_type=heads)
- Точка точка входа в приложение
- Читает сонфиг [**RPlanner3D-cfg**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/RPlanner3D-cfg.js?ref_type=heads) приложения.
- Создаёт экземпляры **modules**:
   - [modules/**Application**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Application.js?ref_type=heads)
   - [modules/**Registry**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Registry.js?ref_type=heads)
   - [modules/**Project**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/Project.js?ref_type=heads)
   - [modules/**Mode**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Mode.js?ref_type=heads)
   - [modules/**Decor**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Decor.js?ref_type=heads)
   - [modules/**Skin**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Skin.js?ref_type=heads)
   - [modules/**Entourage**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Entourage/Entourage.js?ref_type=heads)
   - [modules/**Apartments**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Apartments.js?ref_type=heads)
   - [modules/**Spatium**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Spatium/Spatium.js?ref_type=heads)
   - [**Engine**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/Engine.js?ref_type=heads)


[**Engine**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/Engine.js?ref_type=heads)
- Создаётся точкой входа [Remplanner3D](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/RPlanner3D.js?ref_type=heads).
- По конфигу [**RPlanner3D-cfg**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/RPlanner3D-cfg.js?ref_type=heads) создаёт все **systems** (например `systems/**UserInterface**`).
- Все `systems` изолированы и общаются через **медиатор**.


### 3.2 Modules создаваемые в [Remplanner3D](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/RPlanner3D.js?ref_type=heads)

[modules/**Application**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Application.js?ref_type=heads)	
- создается точкой входа Remplanner3D
- хранит права пользователя
- слушает ошибки приложения, отсылает на сервер репорты об ошибках  
- создает эффекты постпроцессинга в зависимости от аппаратной поддержки 
- `.isSandbox` = window['rplanner_sandbox'] флаг не слать алерты на реальный сервер


[modules/**Registry**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Registry.js?ref_type=heads)
- создается точкой входа Remplanner3D
- Реестр данных для моделей для добавления в квартиру пользователя. `**window.global_visual_data**` - серверные данные зашитые в страницу с бэка 
- Отдает верстку для [systems/**UserInterface**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/UserInterface.js?ref_type=heads) каталогов элементов 


[modules/**Project**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/Project.js?ref_type=heads)
- создается точкой входа Remplanner3D
- создает класс [modules/Project/**ProjectSource**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/ProjectSource.js?ref_type=heads) в котором происходит загрузка и валидация проекта пользователя
- хранит данные проекта квартиры пользователя


   [modules/Project/**ProjectSource**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/ProjectSource.js?ref_type=heads) 
   - подгрузка и валидация проекта квартиры пользователя


   [modules/Project/**OldVersion**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/OldVersion.js?ref_type=heads)
	- конвертация серверных сохраненных данных в формат удобный приложению


   [modules/Project/**SaveProject**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/ProjectSave.js?ref_type=heads)
   - создание объекта текущих изменений проекта пользователя и отправка на сервер  


[modules/**Mode**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Mode.js?ref_type=heads) 
- создается точкой входа Remplanner3D
- текущий режим взаимодействия с пользователем (Обычный вид, Ходьба, Скрытие стен).
- сохраняет пользовательские ракурсы камер в LocalStorage и подтягивает оттуда


[modules/**Decor**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Decor.js?ref_type=heads) 
- создается точкой входа Remplanner3D 
- создает и при помощи [Engine/syatems/**Graphics**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Graphics.js?ref_type=heads) и добавляет в сцену меши декора


   [modules/Decor/**Placement**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Placement/Placement.js?ref_type=heads)
   - отвечает за перемещение декора


    [modules/Decor/**Lair**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Lair.js?ref_type=heads)
    - отвечает за поверхности для размещения декора

  
[modules/**Skin**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Skin.js?ref_type=heads) 
- создается точкой входа Remplanner3D
- из проекта пользователя сигналит в сцену создать материалы
- `#create(options)` ищет материал в сохраненном проекте пользователя, если находит то берет дефолтный материал из [modules/**Registry**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Registry.js?ref_type=heads) и применяет к нему cохраненные настройки, если не находит то берет материал из [modules/Skin/**Presets**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Presets.js?ref_type=heads) и сохраняет в проекте кастомные настройки.
	- `options.saveKey` - ключ по которому материал ищется в сохраненном проекте пользователя и сохраняется если не находится и возвращается
	- `options.presetId` - если options.saveKey не находит материал то материал берется из [modules/Skin/**Presets**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Presets.js?ref_type=heads)


[modules/**Entourage**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Entourage/Entourage.js?ref_type=heads) 
- создается точкой входа Remplanner3D
- создает студию для рендера


[modules/**Spatium**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Spatium/Spatium.js?ref_type=heads) 
- создается точкой входа Remplanner3D
- рендер в картинку 
- выгрузка модели квартиры .glb

  
[modules/**Apartments**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Apartments.js?ref_type=heads) 
- создается точкой входа Remplanner3D
- создает коробку квартиры
- создает экземпляры modules/Walls, modules/Ceiling, modules/Portal, modules/Interior, modules/Floor, modules/PlinthsFloor, modules/PlinthsCeiling


### 3.3 Модули создаваемые в `modules/Apartments`

[modules/Wall](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/Wall.js?ref_type=heads) 
- создается в modules/Apartments
- хранит данные о стенах 
- создает экземпляры следующих классов


	[modules/Wall/**WallNiches**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/WallNiches.js?ref_type=heads)
	- создает Ниши 
	- хранит данные о нишах


	[modules/Wall/**WallMoldings**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/WallMoldings.js?ref_type=heads)
	- создает Молдинги 
	- хранит данные о молдингах


 - пользуется следующими методами


   	[modules/Wall/**WallFaces**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/WallFaces.js?ref_type=heads)
   	- создает, добавляет стены

  
   	[modules/Wall/**WallCaps**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/WallCaps.js?ref_type=heads)
   	- создает, добавляет верх стен


	[modules/Wall/Generators/**WallGlass**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/Generators/WallGlass.js?ref_type=heads)
	- создает, добавляет стеклянные стены  


	[modules/Wall/**WallGeometry**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/WallGeometry.js?ref_type=heads)
	- Набор вспомогательных методов создания геометрии для всех классов стен 


[modules/Portal](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Portal/Portal.js?ref_type=heads) 
- создается в modules/Apartments
- Создает внутренние откосы, окна, двери   
- пользуется методами modules/Wall/WallFaces, modules/Wall/WallGeometries
- добавляет в сцену пустые проемы, двери и окна


	[modules/Portal/**PortalFrame**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Portal/PortalFrame.js?ref_type=heads)
	- хранит данные о порталах
	- набор статических методов создания данных для Окон и Дверей


	[modules/Portal/**PortalSlopes**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Portal/PortalSlopes.js?ref_type=heads)
	- набор статических методов создания откосов
   




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
